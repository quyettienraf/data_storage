# data_storage
1. 
- paper Presto: https://trino.io/paper.html
- link Teams: https://trungtv.github.io/articles/2020-03/it4931-luu-tru-va-xu-ly-du-lieu-lon

## 2. demo
### Start a very simple Trino server 
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

### Connect with the Trino CLI
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


