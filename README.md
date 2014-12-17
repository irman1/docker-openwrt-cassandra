Cassandra 2.1.2 as a Docker container. For development use only.

## Quickstart

### Single Node

Without arguments, the container starts the C* server:

```
docker run -d --name cass mcreations/openwrt-cassandra
```

You can also use the container for the client tools (cqlsh etc.) by
passing it the full path to the command:

```
docker run -it --link cass:cass mcreations/openwrt-cassandra /opt/cassandra/bin/cqlsh cass
```

## Configuration Details

Cassandra server is started with the following settings which can also be passed with ```docker run -e```:

- ```MAX_HEAP_SIZE``` 1 GB
- ```HEAP_NEWSIZE``` 100 MB
- ```CLUSTER_NAME```
- ```SEEDS``` is set to the IP of the running container
- ```OPS_IP``` optional address of OpsCenter
- data directory is /data

This is a sample command line with custom parameters:

```
docker run -d --name cass1 -v /data/cass1:/data \
           -e MAX_HEAP_SIZE=600m -e HEAP_NEWSIZE=40m \
           -e CLUSTER_NAME=mycluster -e OPS_IP=192.168.1.1 \
           mcreations/openwrt-cassandra
```

For the complete details of the configuration, please see

- [start-cassandra](https://github.com/m-creations/docker-openwrt-cassandra/blob/master/image/root/start-cassandra)
- [cassandra-env.sh.in](https://github.com/m-creations/docker-openwrt-cassandra/blob/master/image/root/tmp/cassandra-env.sh.in)

## Configuring authentication

To configure Cassandra to use internal authentication

```
docker run -d --name cass -e AUTHENTICATOR=PasswordAuthenticator -e USERNAME=cassandrauser1 -e PASSWORD=casspasswd -e REPLICA_FACTOR=4 mcreations/openwrt-cassandra
```

```AUTHENTICATOR``` is the value of authenticator parameter in cassandra.yaml and its default value is AllowAllAuthenticator, but when PasswordAuthenticator is used, ```USERNAME``` and ```PASSWORD``` parameters are mandatory to authenticate a user from a client.

```REPLICA_FACTOR``` is number of replications of the keystores related to security data and its default value is 3.

#### Multiple Nodes

Follow the single node setup to get the first node running and keep
track of its IP. Run the following to launch the other nodes in the
cluster:

```
SEED_IP=$(docker inspect -f '{{ .NetworkSettings.IPAddress }}' cass1)
```

```
for name in cass{2..5}; do
  echo "Starting node $name"
  docker run -d --name $name -v /data/$name:/data -e MAX_HEAP_SIZE=600m -e HEAP_NEWSIZE=100m -e CLUSTER_NAME=testcluster -e OPS_IP=192.168.1.1 -e SEEDS=$SEED_IP mcreations/openwrt-cassandra
  sleep 30
done
```

Once all the nodes are up, check cluster status using:

```
nodetool --host $SEED_IP status
```
