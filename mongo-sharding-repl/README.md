# pymongo-api

## Как запустить

Запускаем mongodb и приложение

```shell
docker compose up -d
```


## Как проверить

### Если вы запускаете проект на локальной машине

Откройте в браузере http://localhost:8080




### Настройка 

### Подключитесь к серверу конфигурации и сделайте инициализацию:
```shell
docker compose exec -T configSrv mongosh <<EOF
rs.initiate(
      {
        _id : "config_server",
           configsvr: true,
        members: [
          { _id : 0, host : "configSrv:27017" }
        ]
      }
    );
EOF
```
### Инициализируйте кластера:
```shell
docker compose exec -T shard1 mongosh --port 27018 <<EOF
rs.initiate({_id: "shard1", members: [
{_id: 0, host: "shard1:27018"},
{_id: 1, host: "shard1_replica1:27021"},
{_id: 2, host: "shard1_replica2:27022"}
]}) 
EOF

docker compose exec -T shard2 mongosh --port 27019 <<EOF
rs.initiate({_id: "shard2", members: [
{_id: 0, host: "shard2:27019"},
{_id: 1, host: "shard2_replica1:27023"},
{_id: 2, host: "shard2_replica2:27024"}
]}) 
EOF
```


### Инцициализируйте роутер и наполните его тестовыми данными:

```shell
docker compose exec -T mongos_router mongosh --port 27020 <<EOF
sh.addShard( "shard1/shard1:27018");
sh.addShard( "shard2/shard2:27019");

sh.enableSharding("somedb");
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

use somedb

for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i})

db.helloDoc.countDocuments() ;
EOF
```


### Сделайте проверку на шардах:
```shell
docker compose exec -T shard1 mongosh --port 27018 <<EOF
use somedb;
db.helloDoc.countDocuments();
EOF

docker compose exec -T shard2 mongosh --port 27019 <<EOF
use somedb;
db.helloDoc.countDocuments();
EOF
```
