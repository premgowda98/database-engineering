# Partitioning  

1. Break the tables into small units.
2. Horizontal Partitioning
   1. Split rows into partitions
3. Vertical Partitioning
   1. Split columns into partitions
   2. Large column like blob can be stored in slow access drive in its own tablespace.
4. Partition Types
   1. By Rage -> Date, Ids
   2. By List -> State ZipCode
   3. By Hash
5. Horizontal Partitioning vs Sharding
   1. Partitioning of a table in same database server.
   2. Sharding is partitioning of a table in different databases server.
   3. In HP, the table name changes
   4. But in Sharding, the table name remains same but server changes.

## Demo

### Without partition

```sql
create table grades_org (id serial not null, g int not null);
insert into grades_org (g) select random(1,100) from generate_series(0,10000000);
create index grades_org_index on grades_org(g);

explain analyze select count(g) from grades_org where g=30;
```

### Create Partition

1. The tables for the partition will not be created by the database, it should be manually created.
2. Creating index on main table will create index on all the partitions.


```sql
create table grades_parts (id serial not null, g int not null) partition by range(g);

# create partitions
create table g0035 (like grades_parts including indexes);
create table g3560 (like grades_parts including indexes);
create table g6090 (like grades_parts including indexes);
create table g90100 (like grades_parts including indexes);

# Attach partitions to main table
alter table grades_parts attach partition g0035 for values from (0) to (35);
alter table grades_parts attach partition g3560 for values from (35) to (60);
alter table grades_parts attach partition g6090 for values from (60) to (90);
alter table grades_parts attach partition g90100 for values from (90) to (120);

insert into grades_parts (g) select g from grades_org;

select max(g), min(g) from g90100; => includes only values between 90 and 100.

explain analyze select g from grades_parts where g=49; => will use the corresponding index to search for the data
```