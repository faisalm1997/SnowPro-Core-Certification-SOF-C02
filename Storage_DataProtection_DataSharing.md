# Storage, Data Protection and Data Sharing

## Storage Summary

Data is stored within the cloud provider of choice as either blob or S3. The data is then passed to Snowflake and reorganized into its columnar data structure which is compressed and optimized for analytical processing. Structured and semi-structured data can be ingested into the storage layer, and the underlying file is no longer accessible in Snowflake.

The data is also micro-partitioned and keys are automatically assigned to records. Files are assigned to stages and can be done internally/externally before loading data into Snowflake. Micro partitions are stored in the services layer.

## Micro-Partitions

- Snowflake partitions are formed along the natural ordering of the input data as it is inserted or loaded.
- Micro partitions are the physical file stored in blob storage and range in size between 50-500MB of uncompressed data.
- Micro partitions undergo a reorganization process into the columnar data format for Snowflake.
- Micro partitions are immutable, they are written once and read many times.

Each micro partition also has metadata, including min and max values. The metadata allows Snowflake to optimize a query by first checking the min-max metadata of a column. This then discards micro partitions from the query plan which are not required. Metadata is considered smaller than the actual data which means queries are faster.

---

## Time Travel and Fail-Safe

## Data Lifecycle

- Current data storage > Time travel retention > Fail-safe

### Time Travel

Time travel enables users to:

- Restore objects such as tables, schemas, and databases that have been removed. `UNDROP`
- Analyze historical data by querying at points in the past. `SELECT * FROM MY_TABLE AT (TIMESTAMP---)`
- Create clones of objects from a point in the past.

### Time Travel Retention Period

Users can configure the retention period for time travel by `DATA_RETENTION_TIME_IN_DAYS`.

- The default retention period on account, database, schema, and table level is 1 day/24 hours on the standard edition of Snowflake. For the enterprise edition, it can be between 1-90 days. Temporary and transient objects can have a max retention period of 1 day across all editions.

- **AT**: This keyword allows you to capture historical data inclusive of all changes, made by a statement or transaction up until that point.
- **BEFORE**: Select historical data from a table up to but not including any changes made by a specified statement or transaction.
- **UNDROP**: Can be used to restore the most recent version of a dropped table, schema, or database. If the same name of the object is returned, an error occurs. To view dropped objects you can use `SHOW TABLES HISTORY`.

### Fail-Safe

- Current data storage > Time travel retention > Fail-safe

Fail-safe is a non-configurable period of 7 days in which historical data can be recovered by contacting Snowflake support. It could take several hours or days for Snowflake to complete the full recovery. Fail-safe is only available for permanent objects.

---

## Cloning

The process of creating a copy of an existing object within a Snowflake account. However, only Snowpipes with external stages can be cloned.

- Cloning is a metadata-only operation and copies properties, structure, and configuration of its source. Cloning does not contribute to storage costs until data is modified or new data is added to the clone.

### Zero Copy Cloning

- `CREATE TABLE MY_CLONE CLONE MY_TABLE;` Changes made after the point of cloning then start to create additional micro partitions. Changes made to the clone or the source of the clone are independent so they don't affect each other. Clones can be cloned with no near limits, cloning is metadata-only, thus it is a very quick process (rapid integration testing).

### Cloning Rules

- A cloned object does not retain privileges of the source object with the exception of tables.
- Cloning is recursive for databases/schemas.
- External tables and internal named stages cannot be cloned.
- A cloned table does not contain the load history of the source table.
- Temporary and transient tables can only be cloned as temporary and not as permanent.

### Cloning with Time Travel

- Time travel and cloning features can be combined to create a clone of an existing database, schema, table, and stream at a point within their retention period.
- If the source object did not exist at the time specified in the AT/BEFORE parameter, an error is thrown.

---

## Replication

A feature in Snowflake which is used to replicate databases between Snowflake accounts within an organization. A database can be selected as primary from which you can select/create secondary databases in other accounts.

- `ALTER DATABASE DB_1 ENABLE REPLICATION TO ACCOUNTS ORG1.ACCOUNT2;`

When a primary DB is replicated, a snapshot of the database and its objects are created and transferred to the secondary database.

- `CREATE DATABASE DB_1_REPLICA AS REPLICA OF ORG1.ACCOUNT1.DB_1;`

The secondary database can then be refreshed periodically.

- `ALTER DATABASE DB_1_REPLICA REFRESH;`

### Replication Rules

1. External tables, event tables, temporary stages, and class instances are not replicated.
2. Refreshing a secondary database can be automated via a task.
3. The refresh operation can fail if a primary database contains an event or external table.
4. Privileges granted to database objects on primary are not replicated to the secondary database.
5. Billing for the database replication is based on data transfer and compute.

---

## Data Storage

Cost is calculated on a monthly basis based on the average number of disk bytes stored on a daily basis in a Snowflake account. Monthly costs for storing data in Snowflake are based on a flat rate per TB. Database tables are made up of current data storage, time travel retention, and fail-safe features.

The storage pricing/cost is determined by cloud provider, region, and pricing plan.

---

## Secure Data Sharing

Allows an account to provide read-only access to selected database objects to other Snowflake accounts without having to transfer any data.

Sharing is enabled by using `SHARE`. It is created by a data provider and consists of two configurations: grants on database objects, consumer account definitions. An account can share tables, external tables, secure views, secure materialized views, and secure UDFs. Sharing is not available on VPS.

- `GRANT USAGE (specify object) TO SHARE MY_SHARE;`

Objects which are added to a share are immediately available to all consumers. Only one database can be added per share. No limits on the number of shares you can create or the number of accounts you can add to a share.

### Data Provider

- To create a share, `CREATE SHARE` privilege must be granted. Access to a share or database object in a share can be revoked at any time. A share can only be granted to accounts in the same region and cloud provider as the data provider account.

### Data Consumer

- Only one database can be created per share, cannot use the time travel feature on shared database objects. Cannot create a clone of a shared database or database objects. Consumer would need to `IMPORT SHARE` and consumers cannot create objects inside the shared database. They also cannot refresh shared database objects.

### Reader Account

- Provides the ability for non-Snowflake customers to gain access to a provider's data. They can't insert or copy data into the account, can only consume data from the provider account which created it. Provider accounts assume responsibility for the reader account they have created and are billed for usage.

---

## Data Exchange

A private version of the data marketplace for accounts to provide and consume data. Data exchange can be set up for an account via Snowflake support. The Snowflake account which is hosting the data exchange can manage members as providers and consumers of the data and manage requests for listing data sources.
