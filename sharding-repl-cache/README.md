## Старт

#### Запускаем mongodb и приложение
```shell
docker compose up -d
```

#### Подключиться и инициализировать сервер конфигурации:
```shell
docker exec -it configSrv mongosh --port 27017

rs.initiate(
  {
    _id : "config_server",
    configsvr: true,
    members: [
      { _id : 0, host : "configSrv:27017" }
    ]
  }
);
exit();
```

#### Инициализировать шарды и добавить реплики:
1
```shell
docker exec -it shard1 mongosh --port 27018

rs.initiate(
  {
    _id: "rs0",
    members: [
      { _id: 0, host: "shard1:27018" },
      { _id: 1, host: "shard1_replica:27018" }
    ]
  }
);
exit();
```
2
```shell
docker exec -it shard2 mongosh --port 27019

rs.initiate(
  {
    _id: "rs1",
    members: [
      { _id: 0, host: "shard2:27019" },
      { _id: 1, host: "shard2_replica:27019" }
    ]
  }
);
exit();
```

#### Инициализировать роутер и наполнить его тестовыми данными:
```shell
docker exec -it mongos_router mongosh --port 27020

 sh.addShard( "rs0/shard1:27018");
 sh.addShard( "rs1/shard2:27019");

 sh.enableSharding("somedb");
 sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )

 use somedb

 for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i})

 db.helloDoc.countDocuments()
 exit();
```

#### Сделать проверку на первом шарде:
```shell 
docker exec -it shard1 mongosh --port 27018
 use somedb;
 db.helloDoc.countDocuments();
 exit();
```

#### Сделать проверку на втором шарде:
```shell
docker exec -it shard2 mongosh --port 27019
 use somedb;
 db.helloDoc.countDocuments();
 exit();
```
