# mongodb-debezium-cdc

    docker-compose up -d

Initialize MongoDB replica set and insert test data:

    docker-compose exec mongodb bash -c '/usr/local/bin/init-inventory.sh'

    
Connect to KSQL console and create the connector:

```
docker exec -it ksqldb bash -c 'ksql http://ksqldb:8088'

CREATE SOURCE CONNECTOR source_inventory WITH (
    'connector.class' = 'io.debezium.connector.mongodb.MongoDbConnector',
    'tasks.max' = '1',
    'mongodb.hosts' = 'rs0/mongodb:27017',
    'mongodb.name' = 'dbserver1',
    'mongodb.user' = 'debezium',
    'mongodb.password' = 'dbz',
    'database.whitelist' = 'inventory',
    'database.history.kafka.bootstrap.servers' = 'kafka:9092',
    'transforms' = 'route,unwrap,insertKey',
    'transforms.route.type' = 'org.apache.kafka.connect.transforms.RegexRouter',
    'transforms.route.regex' = '([^.]+)\\.([^.]+)\\.([^.]+)',
    'transforms.route.replacement' = '$3',
    'transforms.unwrap.type' = 'io.debezium.connector.mongodb.transforms.ExtractNewDocumentState',
    'transforms.unwrap.drop.tombstones' = 'false',
    'transforms.unwrap.delete.handling.mode' = 'drop',
    'transforms.unwrap.operation.header' = 'true',
    'transforms.insertKey.type' = 'org.apache.kafka.connect.transforms.ExtractField$Key',
    'transforms.insertKey.field' = 'id',
    'key.converter' = 'org.apache.kafka.connect.storage.StringConverter'
);
```

From KSQL console display the change stream for orders topic:

```
SHOW TOPICS;
PRINT orders FROM BEGINNING;
```

Execute modification statements in mongo and see how the changes go trough kafka:

```
docker exec -it mongodb bash -c "mongo -u debezium -p dbz"

use inventory
db.orders.save(Object.assign(db.orders.findOne(), { quantity: 10 }))
```

## References

https://debezium.io/documentation/reference/1.3/connectors/mongodb.html

https://debezium.io/documentation/reference/configuration/mongodb-event-flattening.html

https://docs.confluent.io/current/connect/transforms/valuetokey.html