# zabbix-ha-psg 
## on master Side
***
1. docker-compose exec -it postgres /bin/bash
2. vi /var/lib/postgresql/data/pg_hba.conf<br/>
   ```host    replication     replication_user  185.212.194.117/32         md5```
4. vi /var/lib/postgresql/data/postgresql.conf<br/>
```
   wal_level = replica
   max_wal_senders = 5
   wal_keep_size = 64MB
   shared_buffers = 512MB
   max_connections = 100            
   listen_addresses = '*'
   OR
   shared_buffers = 4GB
   work_mem = 128MB
   maintenance_work_mem = 2GB
```
5. psql -U zabbix -d zabbix_db
6. ```CREATE ROLE replica_user WITH REPLICATION LOGIN PASSWORD 'replica_password';```


### Zabbix Optimization

1. **Cache Settings in `zabbix_server.conf`:**
   ```plaintext
   CacheSize=512M
   HistoryCacheSize=256M
   HistoryTextCacheSize=128M
   ValueCacheSize=256M
   ```

2. **Enable More Preprocessors:**
   ```plaintext
   StartPreprocessors=16
   ```

3. **Database Housekeeping:**
   - Regularly archive historical data to optimize query performance.

---



## on slave Side
***
1. docker-compose up -d
2. docker-compose exec -it postgres /bin/bash -c "rm -rf /var/lib/postgresql/data/*"
3. rm -rf ./pgdata/*
4. docker compose exec -t postgres pg_basebackup -h 185.82.65.1 -U replica_user -C -S replica_1 -D /var/lib/postgresql/data -Fp -Xs -P -R --wal-method=stream

## test
***
on master <br/>
1. psql -U zabbix -d zabbix_db
2. SELECT client_addr, state FROM pg_stat_replication;
3. CREATE DATABASE students_db;
4. \c students_db;
5. CREATE TABLE student_details (first_name VARCHAR(15), last_name VARCHAR(15) , email VARCHAR(40) );
6. INSERT INTO  student_details (first_name, last_name, email) VALUES  ('Arthur', 'Spencer', 'arthurspencer@gmail.com');
7. SELECT * FROM student_details;

***
on slave <br/>
1.  psql -U zabbix -d zabbix_db
2.  \l
3.  \c students_db;
4.  SELECT * FROM student_details;
5.  
