# installation Mongo 4.4

```
apt-get update
curl -fsSL https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
apt update
apt-get install -y mongodb-org
```


# Data DIr
```
mkdir -pv /data/mongodb/configdb/
mkdir -pv /data/mongodb/logs
touch /data/mongodb/logs/configsvr.log
ls -alRv /data/
chown -R mongodb:mongodb /data/
```

# Operation
```
vi /etc/mongod.conf
systemctl restart mongod
systemctl status mongod
tail -f /data/mongodb/logs/configsvr.log
journalctl -xe mongod.service
systemctl status mongod
chown mongodb:mongodb -R /data
chown mongodb:mongodb /tmp/mongodb-27018.sock
```
## Configuration MongoDB Config
```
storage:
  dbPath: /data/mongodb/configdb
  journal:
    enabled: true

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /data/mongodb/logs/configsvr.log

# network interfaces
net:
  port: 27019
  bindIp: 10.148.0.16

sharding:
  clusterRole: configsvr

replication:
  replSetName: config
```

```
mongo 10.148.0.16:27019
rs.initiate()
rs.add('10.148.0.14:27019')
rs.status()
```

# Mongo SHARDING

```
mkdir -pv /data/mongodb/sharddb/ 
mkdir -pv /data/mongodb/logs 
touch /data/mongodb/logs/shard.log 
ls -alRv /data/
chown -R mongodb:mongodb /data/
```

```
storage:
  dbPath: /data/mongodb/sharddb
  journal:
    enabled: true

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /data/mongodb/logs/shard.log

# network interfaces
net:
  port: 27018
  bindIp: 10.148.0.12

sharding:
  clusterRole: shardsvr

replication:
  replSetName: shard
```

```
rs.initiate()
rs.add('10.148.0.12:27018')
rs.status()
```

# ROUTER

```
mkdir -pv /data/mongodb/router/ 
mkdir -pv /data/mongodb/logs 
touch /data/mongodb/logs/router.log 
ls -alRv /data/
chown -R mongodb:mongodb /data/
```

```
systemLog:
  destination: file
  logAppend: true
  path: /data/mongodb/logs/router.log 
net:
  port: 27017
  bindIp: 10.148.0.15
sharding:
  configDB: config/10.148.0.16:27019,10.148.0.14:27019
```

```
$ vi /lib/systemd/system/mongod.service
ExecStart=/usr/bin/mongos --config /etc/mongod.conf
```

```
sh.addShard("shard/10.148.0.11:27018,10.148.0.12:27018")
```