
docker-compose up --force-recreate -d

## MySQL
mysql -h127.0.0.1 -uroot -p123456
```
CREATE DATABASE test;

CREATE TABLE test.users (
  id bigint PRIMARY KEY AUTO_INCREMENT,
  name varchar(20) NULL,
  birthday timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO test.users (name) VALUES ('hello');
INSERT INTO test.users (name) VALUES ('world');
INSERT INTO test.users (name) VALUES ('hudi');
```

## package
### flink
```
cd /opt
wget https://repo.huaweicloud.com/apache/flink/flink-1.13.3/flink-1.13.3-bin-scala_2.11.tgz
tar xf flink-1.13.3-bin-scala_2.11.tgz
cd flink-1.13.3

wget -P ./lib/ https://repo1.maven.org/maven2/org/apache/flink/flink-sql-connector-elasticsearch7_2.11/1.13.3/flink-sql-connector-elasticsearch7_2.11-1.13.3.jar | \
wget -P ./lib/ https://repo1.maven.org/maven2/org/apache/flink/flink-connector-jdbc_2.11/1.13.3/flink-connector-jdbc_2.11-1.13.3.jar | \
wget -P ./lib/ https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.1.1/flink-sql-connector-mysql-cdc-2.1.1.jar | \
wget -P ./lib/ https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.22/mysql-connector-java-8.0.22.jar
```

### seatunnel
```
git clone git@github.com:apache/incubator-seatunnel.git
cd incubator-seatunnel
git reset --hard d2dc8bf
mvn clean install -DskipTests

tar xf seatunnel-dist/target/seatunnel-dist-2.0.5-SNAPSHOT-2.11.8-bin.tar.gz
cd seatunnel-dist-2.0.5-SNAPSHOT-2.11.8
```

### cdc
```
tee config/application.conf <<-'EOF'
SET 'table.dml-sync' = 'true';
SET 'parallelism.default' = '1';
SET 'execution.checkpointing.interval' = '5sec';

CREATE TABLE mysql_users (
  id BIGINT PRIMARY KEY NOT ENFORCED,
  name STRING,
  birthday TIMESTAMP(3)
) WITH (
  'connector' = 'mysql-cdc',
  'hostname' = 'localhost',
  'port' = '3306',
  'username' = 'root',
  'password' = '123456',
  'database-name' = 'test',
  'table-name' = 'users'
);

CREATE TABLE es_users (
  id BIGINT PRIMARY KEY NOT ENFORCED,
  name STRING,
  birthday TIMESTAMP(3)
) WITH (
  'connector' = 'elasticsearch-7',
  'hosts' = 'http://localhost:9200',
  'index' = 'users',
  'format' = 'json'
);

INSERT INTO es_users SELECT * FROM mysql_users;
EOF
```

### start
```
bin/start-seatunnel-sql.sh --target local -Drest.port=8081 -Dtaskmanager.numberOfTaskSlots=1
```

## Kibana

localhost:5601
```
### demo cluster settings

PUT _cluster/settings
{
  "transient": {
    "cluster.routing.allocation.disk.threshold_enabled": false
  }
}


PUT _settings
{
  "index": {
    "blocks": {
      "read_only_allow_delete": "false"
    }
  }
}

## cdc user table

GET users/_search

POST /_sql?format=txt
{
  "query": "SELECT id, name, birthday FROM users",
  "fetch_size": 10
}
```










