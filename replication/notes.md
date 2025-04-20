# Replication

### Mater/Backup Replication

1. One master accepts writes and replicates to one or more backups

### Multi Master Replication

1. Multiple masters accept writes and replicate to each other


### Synchronous vs Asynchronous Replication

1. Synchronous Replication
   1. All replicas must acknowledge the write before it is considered successful
   2. Guarantees data consistency but can be slower due to network latency
   3. Example: PostgreSQL with synchronous replication
2. Asynchronous Replication
   1. The master does not wait for replicas to acknowledge the write
   2. Faster but can lead to data inconsistency if the master fails before the replicas are updated
   3. Example: MySQL with asynchronous replication
3. Semi-Synchronous Replication
   1. A compromise between synchronous and asynchronous replication
   2. The master waits for at least one replica to acknowledge the write before considering it successful
   3. Example: MySQL with semi-synchronous replication
4. Multi-Master Replication
   1. Multiple masters can accept writes and replicate to each other
   2. Can lead to conflicts if two masters try to write to the same data at the same time
   3. Example: MySQL with multi-master replication
5. Conflict Resolution
   1. The process of resolving conflicts that arise when multiple masters try to write to the same data at the same time
   2. Can be done using various strategies such as last write wins, versioning, or application-level conflict resolution
   3. Example: PostgreSQL with logical replication and conflict resolution


## Demo

1. Spin up 2 docker container for postgres server
```bash
docker run -d --name master_postgres -e POSTGRES_PASSWORD=postgres -p 5454:5432 -v ./master_data:/var/lib/postgresql/data postgres && docker run -d --name replica_postgres -e POSTGRES_PASSWORD=postgres -p 5456:5432 -v ./replica_data:/var/lib/postgresql/data postgres
```
2. Stop the containers
3. Move the data from the master to replica
```bash
mv replica_data replica_data.bak
cp -R master_data replica_data
```
4. Add `host replication postgres all scram-sha-256` to the `pg_hba.conf` file in the master container
5. Add `primary_conninfo = 'application_name=replica1 host=192.168.0.110 port=5454 user=postgres password=postgres'	` to the `postgresql.conf` file in the replica container
6. Add `synchronous_standby_names = 'first 1 (replica1)'` to the `postgresql.conf` file in the master container