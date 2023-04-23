# data_storage

## 1. Presto 
- paper Presto: https://trino.io/paper.html
- link Teams: https://trungtv.github.io/articles/2020-03/it4931-luu-tru-va-xu-ly-du-lieu-lon

## 2. Demo
### 2.1. Start a very simple Trino server 
```
docker pull trinodb/trino
```
You can launch a single node Trino cluster for testing purposes. The Trino node will function both as a coordinator and a worker. To launch it, execute the following:
```
docker run -p 8080:8080 --name trino trinodb/trino
```
Wait for the following message log line:
```
INFO	main	io.trino.server.Server	======== SERVER STARTED ========
```
The Trino server is now running on localhost:8080 (the default port).

### 2.2. Connect with the Trino CLI
Run the Trino CLI, which connects to localhost:8080 by default:
```
docker exec -it trino trino
```
You can pass additional arguments to the Trino CLI:
```
docker exec -it trino trino --catalog tpch --schema sf1
```
- Inspect available catalogs, schema, table and more
```
show catalogs;
show schemas in tpch;
show tables in tpch.sf1;
use tpch.sf1;
select * from nation;
show functions;
select * from orders limit 10;
describe nation;

```
- Learn and test SQL


### 2.3. Trino multi node
https://github.com/Lewuathe/docker-trino-cluster#images

#### 2.3.1. Features
- Multiple node cluster on docker container with docker-compose
- Distribution of pre-build trino docker images
- Override the catalog properties with custom one
- Terraform module to launch ECS based cluster

#### 2.3.2. Images

|Role|Image|Pulls|Tags|
|:---|:---|:---:|:---:|
|coordinator|lewuathe/trino-coordinator|[![Docker Pulls](https://img.shields.io/docker/pulls/lewuathe/trino-coordinator.svg)](https://cloud.docker.com/u/lewuathe/repository/docker/lewuathe/trino-coordinator)|[tags](https://cloud.docker.com/repository/docker/lewuathe/trino-coordinator/tags)|
|worker|lewuathe/trino-worker|[![Docker Pulls](https://img.shields.io/docker/pulls/lewuathe/trino-worker.svg)](https://cloud.docker.com/u/lewuathe/repository/docker/lewuathe/trino-worker)|[tags](https://cloud.docker.com/repository/docker/lewuathe/trino-worker/tags)|

We are also providing ARM based images. Images for ARM have suffix `-arm64v8` in the tag. For instance, the image of 336 has two types of images supporting multi-architectures. Following architectures are supported for now.

- `linux/amd64`
- `linux/arm64/v8`

#### 2.3.3 Usage

Images are uploaded in [DockerHub](https://hub.docker.com/). These images are build with the corresponding version of trino. Image tagged with 306 uses trino 306 inside. Each docker image gets two arguments

|Index|Argument|Description|
|:---|:---|:---|
|1|discovery_uri| Required parameter to specify the URI to coordinator host|
|2|node_id|Optional parameter to specify the node identity. UUID will be generated if not given|

You can launch multi node trino cluster in the local machine as follows.

- <b> If run in chip ARM </b>
```sh
# Create a custom network
$ docker network create trino_network

# Launch coordinator
$ docker run -p 8080:8080 -itd \
    --net trino_network \
    --name coordinator \
    lewuathe/trino-coordinator:354-arm64v8 http://localhost:8080 1

# Launch two workers
$ docker run -itd \
    --net trino_network \
    --name worker1 \
    lewuathe/trino-worker:354-arm64v8 http://coordinator:8080 2

$ docker run -itd \
    --net trino_network \
    --name worker2 \
    lewuathe/trino-worker:354-arm64v8 http://coordinator:8080 3
```

- <b> If run in chip x86 </b>
```sh
# Create a custom network
$ docker network create trino_network

# Launch coordinator
$ docker run -p 8080:8080 -itd \
    --net trino_network \
    --name coordinator \
    lewuathe/trino-coordinator:354 http://localhost:8080 1

# Launch two workers
$ docker run -itd \
    --net trino_network \
    --name worker1 \
    lewuathe/trino-worker:354 http://coordinator:8080 2

$ docker run -itd \
    --net trino_network \
    --name worker2 \
    lewuathe/trino-worker:354 http://coordinator:8080 3
```

#### 2.3.4. docker-compose.yml

[`docker-compose`](https://docs.docker.com/compose/compose-file/) enables us to coordinator multiple containers more easily. You can launch a multiple node docker trino cluster with the following yaml file. `command` is required to pass discovery URI and node id information which must be unique in a cluster. If node ID is not passed, the UUID is generated automatically at launch time.

```yaml
version: '3'

services:
  coordinator:
    image: "lewuathe/trino-coordinator:${trino_VERSION}"
    ports:
      - "8080:8080"
    container_name: "coordinator"
    command: http://coordinator:8080 coordinator
  worker0:
    image: "lewuathe/trino-worker:${trino_VERSION}"
    container_name: "worker0"
    ports:
      - "8081:8081"
    command: http://coordinator:8080 worker0
  worker1:
    image: "lewuathe/trino-worker:${trino_VERSION}"
    container_name: "worker1"
    ports:
      - "8082:8081"
    command: http://coordinator:8080 worker1
```

The version can be specified as the environment variable.

```
$ trino_VERSION=354 docker-compose up
```

### connector
- https://trino.io/docs/current/connector/mongodb.html


#### 3. Run database 
- MySQL:
```
docker pull mysql:latest
docker run --name mysql_test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=1234 -d mysql:latest
```
- MongoDB:
```
docker run -itd -p 27017:27017 --name mongo_test mongo:5.0.16-focal
```
- Postgres:
```
docker run --name postgres_test -p 5432:5432 -e POSTGRES_PASSWORD=1234 -d postgres:latest
```
