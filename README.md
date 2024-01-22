# PMM client service
here are some commands to setup pmm monitoring server and client

### 1.Install PMM Server using docker compose :
```
docker compose up -d
```

### 2.Install PMM Client App :
```
wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
sudo apt update
sudo apt install pmm2-client
```

### 3.Add PMM Client Node :
```
sudo pmm-admin config --server-insecure-tls --server-url=https://admin:<password>@<pmm_server_ip_address>:<https-port>
```

### 4.After add node you can check node status on pmm server on this URL
```
http://<pmm-server-ip>:<http-port>/graph/inventory/nodes
```

### 5.Create a PMM user for monitoring on PostgresqlDB:
if you database on the docker container you can use this :

```sudo docker exec -it <container-name> psql -d <db-name> -U <db-user>```

and run this scripts:
```
CREATE USER pmm WITH SUPERUSER ENCRYPTED PASSWORD '<pass>';
GRANT ALL PRIVILEGES ON DATABASE <database-name> TO pmm;
ALTER USER pmm CONNECTION LIMIT 10;
```

### 6.ADD pg_stat_statements Extention
```
CREATE EXTENSION pg_stat_statements SCHEMA public;
GRANT pg_read_all_stats TO pmm;
```

### 8.config pg_hda.conf
go to `/var/lib/postgresql/data/pg_hba.conf`

and add this on pg_hba.conf:
```host    all             pmm             <postgres-server-ip>          md5```

<postgres-server-ip> 
ex : 192.168.1.50/32

### 9.config postgresql.conf
go to `/var/lib/postgresql/data/postgresql.conf`

and add this on postgresql.conf end of line:
```
shared_preload_libraries = 'pg_stat_statements'
track_activity_query_size = 2048 # Increase tracked query string size
pg_stat_statements.track = all   # Track all statements including nested
track_io_timing = on             # Capture read/write stats
```

### 10.Register the server for monitoring :
```
sudo pmm-admin add postgresql --username=pmm --password=<pass> --host=127.0.0.1 --port=,db-port> --database=<database-name> --service-name=<service-name> --query-source=pgstatements comments-parsing="off" --debug
```


### 6.Enjoy it ;)