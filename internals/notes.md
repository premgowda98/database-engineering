# Internals

### Storage Concepts

1. Table
2. Row_id
   1. Internal and system maintained
   2. In postgres also called as tuple_id
3. Page
   1. Rows are stored in a pages
   2. Page size can vary between databases
      1. 8kb - Postgres
      2. 16kb - MySQL
4. IO
5. Heap data structure
   1. The table with all it's content are stored in Heap
   2. Traversing the heap is very expensive
   3. And we indexes to help us exactly what part of heap to be searched for.
6. Index data structure - B-Tree
   1. Is the pointer for the data in heap.
   2. Also stored as page.
   3. If can fit in memory, search is fast.
7. Note
   1. Clustered Index (Index organized table)
      1. Is Sequential
      2. Heap table is stored around single index
      3. Primary Key is usually a clustered index
      4. Any other indexes point to the primary key
   2. Postgres only have secondary indexes and all indexes point directly to the row_id which lives in heap.


