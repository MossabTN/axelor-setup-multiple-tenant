
#### Start application
```shell script
docker-compose up -d
```

#### Create a prototype tenant (wait application to fully start)
```shell script
docker exec -it axelor bash
psql postgresql://axelor:axelor@localhost:5432/axelor -c "SELECT pg_terminate_backend(pg_stat_activity.pid) FROM pg_stat_activity WHERE pg_stat_activity.datname = 'axelor' AND pid <> pg_backend_pid();"
psql postgresql://axelor:axelor@localhost:5432/axelor -c "CREATE DATABASE backup WITH TEMPLATE axelor;" -c "CREATE USER backup4user WITH PASSWORD 'backup4password';" -c "GRANT ALL PRIVILEGES ON DATABASE backup TO backup4user;" -c "\c backup" -c "GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public to backup4user;"
```

### Config multi tenancy

##### Enable multi-tenancy
* update the file 'config/app.properties'
```properties
application.multi_tenancy = true
db.default.visible = false 
```

* Restart application
```shell script
docker-compose restart
```

* Add the hostnames to the /etc/hosts file (C:\Windows\System32\drivers\etc\hosts for **windows**)
```shell script
#paste this hostnames#
127.0.0.1       tenant1.maxilog.io
127.0.0.1       tenant2.maxilog.io
```


##### Create the first tenant ( **database**: client1, **user**: client1user, **password**: client1password) from prototype database

```shell script
docker exec -it axelor bash
psql postgresql://axelor:axelor@localhost:5432/axelor -c "CREATE DATABASE client1 WITH TEMPLATE backup;" -c "CREATE USER client1user WITH PASSWORD 'client1password';" -c "GRANT ALL PRIVILEGES ON DATABASE client1 TO client1user;" -c "\c client1" -c "GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public to client1user;"
```


##### Create the second tenant ( **database**: client2, **user**: client2user, **password**: client2password) from prototype database

```shell script
docker exec -it axelor bash
psql postgresql://axelor:axelor@localhost:5432/axelor -c "CREATE DATABASE client2 WITH TEMPLATE backup;" -c "CREATE USER client2user WITH PASSWORD 'client2password';" -c "GRANT ALL PRIVILEGES ON DATABASE client2 TO client2user;" -c "\c client2" -c "GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public to client2user;"
```