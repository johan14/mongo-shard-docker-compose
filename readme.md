Mongo Sharded Cluster with Docker Compose
=========================================
A simple sharded Mongo Cluster with a replication factor of 3 running in `docker` using `docker-compose`.

Designed to be quick and simple to get a local or test environment up and running. Needless to say... DON'T USE THIS IN PRODUCTION!


### Mongo Components

* Config Server (3 member replica set): `config01`,`config02`,`config03`
* 3 Shards (each a 3 member replica set):
	* `shard01a`,`shard01b`,`shard01c`
	* `shard02a`,`shard02b`,`shard02c`
	* `shard03a`,`shard03b`,`shard03c`
* 1 Router (mongos): `router`
* (TODO): DB data persistence using docker data volumes

### First Run (initial setup)
**Start all of the containers** (daemonized)

```
docker-compose up -d
```

**Initialize the replica sets (config server and shards) and router**

```
sh init.sh
```

This script has a `sleep 20` to wait for the config server and shards to elect their primaries before initializing the router

**Verify the status of the sharded cluster**

```
docker-compose exec router mongo localhost:27029
mongos> sh.status()
--- Sharding Status --- 
  sharding version: {
  	"_id" : 1,
  	"minCompatibleVersion" : 5,
  	"currentVersion" : 6,
  	"clusterId" : ObjectId("608296cc7f0ac2f2c21cb2f2")
  }
  shards:
        {  "_id" : "shard01",  "host" : "shard01/shard01a:27020,shard01b:27021,shard01c:27022",  "state" : 1 }
        {  "_id" : "shard02",  "host" : "shard02/shard02a:27023,shard02b:27024,shard02c:27025",  "state" : 1 }
        {  "_id" : "shard03",  "host" : "shard03/shard03a:27026,shard03b:27027,shard03c:27028",  "state" : 1 }
  active mongoses:
        "4.4.5" : 1
  autosplit:
        Currently enabled: yes
  balancer:
        Currently enabled:  yes
        Currently running:  no
        Failed balancer rounds in last 5 attempts:  0
        Migration Results for the last 24 hours: 
                No recent migrations
  databases:
```

### Normal Startup
The cluster only has to be initialized on the first run. Subsequent startup can be achieved simply with `docker-compose up` or `docker-compose up -d`

### Accessing the Mongo Shell
Its as simple as:

```
docker-compose exec router mongo localhost:27029
```

### Resetting the Cluster
To remove all data and re-initialize the cluster, make sure the containers are stopped and then:

```
docker-compose rm
```

Execute the **First Run** instructions again.
