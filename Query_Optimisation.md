# Query Performance and optimisation

There are several analysis tools which can be used to analyze query performance:

## Analysis Tools

- **The History Tab**: Displays query history for the last 14 days and can view other users' queries but cannot view their query results.

## Database Order of Execution

- **Order**: Rows > Groups > Result

## Query Profile

The query profile can show you what is making up the query and show it as a query tree. Join on columns with unique values and understand table relationships.

## Spilling to Disk

- **Bytes Spilled to Local Disk**: Volume of data which is spilled to the virtual warehouse local disk.
- **Bytes Spilled to Remote Storage**: Volume of data spilled to remote disk.

## Caching

### Services Layer

- **Metadata Cache**: Has a high availability metadata store which can maintain metadata object information and stats.
- **Results Cache**: Is used to reuse a result, the query matches previous queries, underlying table data has not changed, same role is used as the previous query.

### Warehouse Layer

- **Local Disk Cache**: They have local SSD storage which maintains raw table data, used for processing a query. The larger the warehouse, the greater the local cache. The local cache is resized and purged when it is suspended or dropped. Can be used partially to retrieve the rest of the data from a query from remote storage.

### Storage Layer

- **Remote Disk**

## Materialized Views

- A pre-computed and persisted data set which derives from a `SELECT` query.
- They are updated via a background process which ensures data is current and consistent with the base table.
- They improve query performance which makes complex queries readily available.
- Enterprise edition and above serverless feature.
- They use compute resources to perform automatic background maintenance.
- Use storage to share query results, adding to monthly storage usage for an account.
- Can be created on top of external tables to improve their query performance.
- They are limited to: single tables, joins, UDFs, HAVING/ORDER BY/LIMIT/WINDOW FUNCTIONS.

## Clustering Metadata

Snowflake maintains clustering metadata for micro partitions in a table. Number of overlapping micro partitions, depth of overlapping micro partitions.

### Clustering Depth

Measures the average depth of overlapping micro partitions for specified columns in a table. The smaller the average depth, the better clustered the table is with regards to specified columns.

### Automatic Clustering

Snowflake supports and specifies one or more tables as a clustering key for a table. Clustering aims to co-locate data of the same clustering key in the same micro partitions and also improves the performance of queries being filtered. Clustering can be reversed for larger tables in the multi-TB range.

### Choosing a Clustering Key

Snowflake recommends a max of 3-4 columns per key. Columns which are used in common queries can perform filtering and can sort operations.

### Reclustering and Clustering Cost

- As DML operations are performed on a clustered table, the data in the table might become less clustered.
- Reclustering is a background process which reorganizes data into micro partitions by clustering key.
- Initial clustering and reclustering operations consume compute and storage credits.
- Clustering is recommended for larger tables which do not change often and are queried often.

## Search Optimization Service

A table-level property which is aimed at improving the performance of selective point lookup queries.

### Search Optimization

A background process creates and maintains a search access path to enable search optimization.

- `ALTER TABLE MY_TABLE ADD SEARCH OPTIMIZATION`
- `ALTER TABLE MY_TABLE DROP SEARCH OPTIMIZATION`

The access path data structure requires space for each table on which optimization is enabled. The larger the table, the larger the access path storage costs.

## Credits

- Costs 10 Snowflake credits per Snowflake-managed compute hour.
- 5 Snowflake credits per cloud services compute hour.
