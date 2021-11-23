# cluster-es
docker-compose elasticsearch

# Requirements

- [docker GNU/Linux](https://docs.docker.com/engine/install/) or
- [docker for MacOs Windows](https://www.docker.com/products/docker-desktop)
It is necessary to install docker-compose to run multiple docker containers with a single instruction
- [docker-compose](https://docs.docker.com/compose/install/)

verify the installations using the following commands

```bash
docker -v
docker-compose -v
```
# Build and execute

open a terminal and clone the git repository

```bash
git clone https://github.com/kevin-Oz/cluster-es.git 
```

once downloaded go to the repository directory and run the file using docker-compose

This will run 2 nodes elasticsearch
```bash
cd cluster-es/
docker-compose -f cluster-elasticsearch.yml up
```
you can check the amount of nodes in your browser

**http://localhost:9200/_cat/nodes?v**

# More options

you can add the -d flag to hide the logs of containers

```bash
docker-compose -f cluster-elasticsearch.yml up -d
```

if you want to delete the cluster you can execute the following command

```bash
docker-compose -f cluster-elasticsearch.yml down
```

# Potential errors

Verify that port 9200 is available on your machine.

```bash
ss -talp | grep 9200
```


You can increase the virtual memory limits, by running on your GNU/linux host

```bash
sysctl -w vm.max_map_count=262144
```

If you want to set this permanently, you need to edit /etc/sysctl.conf and set vm.max_map_count to 262144
([for more info you can see](https://www.elastic.co/guide/en/elasticsearch/reference/current/vm-max-map-count.html) )

# Explanation of the file


```bash
version: '3.0'
```
Indicates the version of the docker-compose file.

## Volumes
Are the paths where the data is stored locally, you can put a name to the volume or a path to your local host (by default the named volumes are stored in */var/lib/docker/volumes/*).

```bash
volumes:
  data01:
    driver: local
  data02:
    driver: local
```

## Network

Assign a network, which will be used in bridge mode
( [for more information you can consult docker network](https://docs.docker.com/network/))

```bash
networks:
  elastic:
    driver: bridge
```

## Services
are the different containers that will be executed
- **image**: name of the image and elasticsearch tag to use (define the version )
- **container_name**: name of the docker container where the image will be executed
```bash
image: docker.elastic.co/elasticsearch/elasticsearch:7.15.2
container_name: es02
```
In the next section we add environment variables to configure our elasticsearch instance, we also tell the container not to have a memory limit and we assign the network and volume where the data will be stored (volume data02 for example).

```bash
 environment:
      - node.name=es02  #assign a name to the node
      - cluster.name=es-docker-cluster # define the cluster name
      - discovery.seed_hosts=es01,es02 # declare the available nodes
      - cluster.initial_master_nodes=es01,es02 # negotiation of which node is eligible to be the master
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data02:/usr/share/elasticsearch/data
    networks:
      - elastic
```
