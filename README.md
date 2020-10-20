# mongodb-debezium-cdc

    docker-compose up -d

Initialize MongoDB replica set and insert test data:

    docker-compose exec mongodb bash -c '/usr/local/bin/init-inventory.sh'

    docker exec -it ksqldb bash -c 'ksql http://ksqldb:8088'


```
CREATE SOURCE CONNECTOR source_inventory WITH (
    'connector.class' = 'io.debezium.connector.mongodb.MongoDbConnector',
    'tasks.max' = '1',
    'mongodb.hosts' = 'rs0/mongodb:27017',
    'mongodb.name' = 'dbserver1',
    'mongodb.user' = 'debezium',
    'mongodb.password' = 'dbz',
    'database.whitelist' = 'inventory',
    'database.history.kafka.bootstrap.servers' = 'kafka:9092',
    'transforms' = 'route,mongoflatten,createKey',
    'transforms.route.type' = 'org.apache.kafka.connect.transforms.RegexRouter',
    'transforms.route.regex' = '([^.]+)\\.([^.]+)\\.([^.]+)',
    'transforms.route.replacement' = '$3',
    'transforms.mongoflatten.type' = 'io.debezium.connector.mongodb.transforms.ExtractNewDocumentState',
    'transforms.mongoflatten.drop.tombstones' = 'false',
    'transforms.createKey.type' ='org.apache.kafka.connect.transforms.ValueToKey', 
    'transforms.createKey.fields' = '_id'
);
```

## References

https://debezium.io/documentation/reference/1.3/connectors/mongodb.html

https://debezium.io/documentation/reference/configuration/mongodb-event-flattening.html

