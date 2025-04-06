# Indexing

1. Create a table with million rows

```sql
create table temp (t int);
insert into temp select random(1, 100) from generate_series(0, 1000000);
```

2. `\d <table-name>` list info about table including the indexes
3. Create employee table and insert the data

```sql
create table employees( id serial primary key, name text);

create or replace function random_string(length integer) returns text as 
$$
declare
  chars text[] := '{0,1,2,3,4,5,6,7,8,9,A,B,C,D,E,F,G,H,I,J,K,L,M,N,O,P,Q,R,S,T,U,V,W,X,Y,Z,a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z}';
  result text := '';
  i integer := 0;
  length2 integer := (select trunc(random() * length + 1));
begin
  if length2 < 0 then
    raise exception 'Given length cannot be less than 0';
  end if;
  for i in 1..length2 loop
    result := result || chars[1+random()*(array_length(chars, 1)-1)];
  end loop;
  return result;
end;
$$ language plpgsql;

insert into employees(name)(select random_string(10) from generate_series(0, 1000000));
```

4. `\d employees;` display all info of table.
5. `explain analyze select id from employees where id=40;` shows the execution plan
   1. Uses Index Only Scan
6. `explain analyze select name from employees where id=40;`
   1. Uses Index Scan
7. `explain analyze select id from employees where name='Po';`
   1. Parallel Sequential Scan
8. After creating index `create index employee_name on employees(name);`
9. Now the search on name is pretty fast and use Bitmap Index Scan

### Key vs Non Key Index

1. Non key index
   1. Column can be included along with the index

>Note: Just copying and pasting with excerpt from cybertec-postgresql article: `PostgreSQL does not update a table row in place. Rather, it writes a new version of the row (the PostgreSQL term for a row version is “tuple”) and leaves the old row version in place to serve concurrent read requests. VACUUM later removes these “dead tuples”. If you delete a row and insert a new one, the effect is similar: we have one dead tuple and one new live tuple. This is why many people (me, among others) explain to beginners that “an UPDATE in PostgreSQL is almost the same as a DELETE, followed by an INSERT”.` So, that's why we need to update all the indexes.

## UUID

1. [UUID as Index in MySQL vs Postgres](https://youtu.be/Y5mWz4vK10A)
2. UUID as index for Clustered Index databases are bad idea. SInce the whole table will be around the index
3. And UUID being random, on every write the index i.e, B-Tree has to be updated which is very expensive
4. Whereas in Postgres the index points to a row_id, it does not have that implication. 
5. And every update in postgres is delete and insert, so all writes will be append only and index updates its pointer.
6. So no order is required in postgres index.

# B-Tree and B+Tree

1. B-Tree
   1. Balanced data structure for fast traversal.
   2. Has Nodes
   3. Each element has a key and a value. Value is usually data pointer to the row.
   4. Data pointer can point to primary key (mysql) or tuple (postgres).
   5. A node = Disk page
   6. Limitations
      1. Storing both key and value in each element takes more space
2. B+ Tree
   1. Only stores the key in the internal nodes
   2. Values are stored only in leaf nodes
   3. Internal nodes can fir in memory and the leaf nodes on the disk (heap)