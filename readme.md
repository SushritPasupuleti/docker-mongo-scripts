# Create a MongoDB replica set in Docker

Pull mongo

```bash
docker pull mongo
```

## Architecture of the network

| image         | mongo               | mongo                 | mongo                 |
| ------------- | ------------------- | --------------------- | --------------------- |
| cluster       | mongo1<br>(Primary) | mongo2<br>(Secondary) | mongo3<br>(Secondary) |
| local network | localhost:27001     | localhost:27002       | localhost:27003       |

<br>

Check networks on system

```bash
docker network ls
```

create new docker network

```bash
docker network create my-mongo-cluster
```

### Container 1 Setup

```bash
docker run \
-p 27001:27017 \
--name mongo1 \
--net my-mongo-cluster \
mongo mongod --replSet my-mongo-set
```

` --net adds the container to the my-mongo-cluster`

### Container n Setup

```bash
docker run \
-p 27002:27017 \
--name mongo2 \
--net my-mongo-cluster \
mongo mongod --replSet my-mongo-set
```

### Connect to mongo shell of the primary container

```bash
docker exec -it mongo1 mongo
```

Create configuration

```bash
> db = (new Mongo('localhost:27017')).getDB('test')
test
> config = {
  	"_id" : "my-mongo-set",
  	"members" : [
  		{
  			"_id" : 0,
  			"host" : "mongo1:27017"
  		},
  		{
  			"_id" : 1,
  			"host" : "mongo2:27017"
  		},
  		{
  			"_id" : 2,
  			"host" : "mongo3:27017"
  		}
  	]
  }
```

Run replica set
```bash
> rs.initiate(config)
```