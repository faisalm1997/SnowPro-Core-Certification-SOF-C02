# Snowflake Features and Architecture

## What is Snowflake?

Snowflake is a cloud-native data platform which is offered as a service. It supports six key workloads:

1. **Data Warehouse**: Structured and relational data, ANSI standard SQL, ACID compliant transactions, data stored in databases, schemas, and tables.
2. **Data Lake**: Scalable storage and compute, schema does not need to be defined upfront, and can natively process semi-structured data formats.
3. **Data Engineering**: COPY INTO statements and Snowpipe, separate compute clusters, and also tasks and streams which can be used for data pipelines. All data is encrypted at rest and in transit.
4. **Data Science**: Can remove data management roadblocks from centralized storage, SageMaker/Dataiku and other DS tooling can attach to Snowflake.
5. **Data Sharing**: One Snowflake account can share tabular data with another account in the same region privately and securely. BI tools can connect to Snowflake and they have other data sharing products on Data Marketplace, Data Exchange.
6. **Data Applications**: Has several connectors and drivers like ODBC, JDBC. UDFs and stored procedures can be written in SQL, Python, or Java. External UDFs can be stored outside of Snowflake in the respective cloud platform. Some preview features in Snowpark.

Snowflake software is purpose-built to be cloud-agnostic. All the compute, networking, and other cloud infrastructure to run a single Snowflake account is held within the chosen cloud provider environment/account. Snowflake can then make use of the cloud's elasticity, scalability, high availability, cost efficiency, and durability.

### SaaS

- No management of hardware
- Transparent updates and patches with no visible downtime
- Subscription payment model PAYG
- Ease of access
- Automatic optimization such as partitioning of data

---

## Multi-Cluster Shared Data Architecture

### Shared Disk

Works by scaling out nodes in a compute cluster but keeping storage in a single location. All data is accessible from all cluster nodes, any machine can read or write any portion of the data.

**Advantages**:

- Relatively simple to manage, single source of truth

**Disadvantages**:

- Single point of failure
- Bandwidth and network latency
- Limited scalability
- Queries can overwhelm the centralized storage device

### Shared Nothing

Similar architecture to the likes of Hadoop and Spark. The nodes in this architecture do not share any hardware. Storage and compute are located on separate machines which are networked together. The nodes in the cluster contain a subset of the data locally, instead of the nodes sharing one central data repository.

**Advantages**:

- Co-locating compute and storage avoids networking latency issues
- Generally cheaper than shared disk architecture to build and maintain
- Improved scaling over shared disk architecture, can easily add a node to the architecture

**Disadvantages**:

- Scaling of the nodes is still limited
- Storage and compute can be tightly coupled and can have a tendency to overprovision which can be costly

---

## The Snowflake Architecture

Comprises of three layers:

1. **Cloud Services Layer**: This is where the authentication, access control, infrastructure management, query optimizer, transaction manager, and security controls happen. Also where the metadata is stored.
2. **Query Processing Layer**: This is where the virtual warehouses are held. The compute clusters are called virtual warehouses. All virtual warehouses which are created have access to the centralized storage.
3. **Data Storage Layer**: This is where the data which is stored in tables is part of schemas and databases.

**Advantages**:

- Decouple storage, compute, and management services with this type of multi-cluster shared data architecture.
- No limit on how quickly or how much these layers can be scaled. All layers are infinitely scalable.
- Workloads can be isolated and be monolithic in a sense, they can be isolated as compute nodes in the form of virtual warehouses.
- Cloud-agnostic layer makes sure Snowflake works the same on any cloud provider.

### Data Storage Layer

- Has persistent and infinitely scalable cloud storage residing within the cloud provider of choice e.g., S3, Blob storage.
- Snowflake users proxy to get availability and durability guarantees of the cloud provider's storage.
- Data which is loaded is organized by databases, schemas, as tables.
- Both structured and semi-structured data files can be loaded and stored in Snowflake.
- When data files are loaded or rows are inserted into a table, Snowflake reorganizes the data into a compressed columnar table file format.
- The data that is loaded or inserted is also partitioned into 'micro partitions'.
- Storage is billed by how much is stored based on a flat rate per TB calculated monthly.
- Data is not directly accessible in the underlying storage but only by SQL commands.

### Query Processing Layer

- Consists of virtual warehouses which execute processing tasks required to return results for most SQL statements.
- A virtual warehouse is a compute cluster that Snowflake manages.
- Nodes which use share-nothing compute clusters make use of local caching.
- Warehouses can be created or removed instantly, they can be paused or removed, unlimited number can be created with its own configuration. Virtual warehouses come in their own t-shirt sizes and all warehouses have consistent access to the same data in the storage layer.

### Services Layer

- The collection of highly available and scalable services which can coordinate activities such as authentication and query optimization across all accounts.
- Similar to the underlying virtual warehouses resources, also runs on the compute instances.

---

## Types of Snowflake Editions

- **Standard**: Core functionality of Snowflake. Basic ANSI Snowflake features, advanced DML statements, time travel, and data governance/protection features.
- **Enterprise**: Like standard but for larger enterprises and organizations.
- **Business Critical**: Offers high levels of data protection and database failover. Can enable private connectivity.
- **Virtual Private Snowflake (VPS)**: Offers the highest level of security for organizations which have the strictest requirements, e.g., financial institutions. Large organizations which collect, store, and share highly sensitive data.

---

## The Snowflake Object Model

- **Organization**: Used to manage one or more accounts, setup and administer Snowflake features which make use of multiple accounts. Monitors usage across accounts.
  - **Org Admin Role**: Can manage accounts, enable cross-account features, and monitor account usage.
- **Account**: The admin name for a collection of storage, compute, and cloud services which are deployed and managed. Each account is hosted on a single cloud provider. Each account has a single geo-location and a single Snowflake edition.
  - **Account Admin Role**: For account management.
  - **Account URL**: `xyz3134123.us-east-2.aws.snowflakecomputing.com`
    - `xyz3134123.us-east-2.aws`: Account identifier
    - `xyz3134123`: Account locator
    - `us-east-2`: Region ID
    - `aws`: Cloud provider
- **Databases**: Must have a unique identifier in an account, must start with a character and cannot contain special characters and spaces.
- **Schemas**: Must have a unique identifier in the database, must start with a character and cannot contain special characters and spaces.

---

## Table Types in Snowflake

- **Permanent or Standard Tables**: Default table type which exists until dropped. Has 90 days time travel and is fail-safe.
- **Temporary**: Used for transitory data, only there for a session, 1 day time travel and NOT fail-safe.
- **Transient**: Exists until dropped. No fail-safe period and has 1 day time travel.
- **External**: Querying data from outside Snowflake, read-only table, no queries. No time travel and no fail-safe.

---

## View Types in Snowflake

- **Standard**: `CREATE VIEW MY_VIEW`, does not contribute to storage costs. If the source table is dropped then the query will return an error. Used to restrict contents of a table.
- **Materialized**: `CREATE MATERIALIZED VIEW`, stores results of a query definition and periodically refreshes it. Incurs costs as it is serverless. Boosts performance of external tables.
- **Secure**: `CREATE SECURE VIEW`, both standard and materialized can be secure. Query definition only visible to authorized users. Some query optimizations can bypass improved security.

---

## User Defined Functions (UDFs)

A UDF is a schema-level object which can enable users to write their own functions in SQL, JavaScript, Python, Java.

- UDFs can accept 0 or more parameters. They can also return scalar or tabular results (UDTF), can be called as part of a SQL statement, UDFs can also be overloaded.
  - e.g., `CREATE FUNCTION EXAMPLE_FUNCTION(var1, var2): AS $$ dffadsfdsfdsffdsf $$`
  - Can be used in SQL statements, `SELECT EXAMPLE_FUNCTION(col1) FROM MY_TABLE;`

### JavaScript UDF

- Can be specified in the language parameter, can refer to themselves recursively, Snowflake data types are mapped to JavaScript data types.

### Java UDF

- Boots up JVM to execute the function. Snowflake currently supports UDFs written in Java in 8, 9, 10, and 11. They can specify their definition as inline code or a pre-compiled jar file. Cannot be designated as secure.

### External UDF

- Has 4 parts to the function: function name/parameter, return type, integration object, and URL proxy service.
  - **Call Lifecycle**: Client program makes the call to Snowflake, Snowflake executes the query as an external function and has an API integrated to it. The HTTPS proxy service gets a POST request from the external function and returns an HTTPS response.
  - **Limitations**: External UDFs are slower, scalar only, non-shareable, and less secure.

---

## Stored Procedures

Stored procedures are named collections of SQL statements which contain procedural logic. Snowflake can execute stored procedures in JavaScript, in SQL, or in Python/Java/Scala in Snowpark.

- A single stored procedure can be written with input and identifier parameters, users can also define the language they want to write the stored procedure in as well as execute the owner of the stored procedure. Stored procedures can mix JavaScript and SQL whilst using the Snowflake JavaScript API.

---

## Sequences

They are commonly found in SQL databases, unique numbers automatically. There are two optional parameters in sequences, start and increment which you can produce values from. Increments can also be a negative number, you can do `NEXTVAL` which will produce the value which is next in the sequence.

- e.g., `SELECT DEFAULT_SEQUENCE.NEXTVAL;`

- User can bring back all values for the sequence, by doing: `SELECT INCREMENT_SEQUENCE.NEXTVAL, INCREMENT_SEQUENCE.NEXTVAL, etc ....`

---

## Tasks and Streams

### Task

A task is an object which is used to schedule the execution of a SQL statement or a stored procedure using Snowflake scripting. You can have root tasks and child tasks, root tasks must define a schedule. Can have up to 1000 child tasks in a DAG.

- **Workflow**: ACCOUNTADMIN role can `CREATE TASK` on a schedule and specify the warehouse, `COPY INTO table` from a stage and start the task by `ALTER TASK EXAMPLE_TASK RESUME;`

### Stream

An object which is created to view or track DML changes to a source table, inserts/updates/deletes. Can create and query streams. The output can give information on the change, metadata update, and row_id columns.

---

## Billing Overview

Snowflake works on PAYG, it has detailed billing for each month and how much you have used in terms of resources. You can also purchase credits upfront to pay for Snowflake compute and capacity. These credits are cheaper than the on-demand PAYG option.

### Billing Components

- **Virtual Warehouse Services**: Credit is calculated based on the size of the warehouse, calculated per second basis when in started state, credit is calculated on a minimum of 60 seconds.
- **Cloud Services and Compute**: Credits calculated at a rate of 4.4 credits per hour, only resources exceeding 10% of daily usage of compute are billed (cloud services adjustment).
- **Serverless Services**: Each serverless feature has its own credit rate per hour. Serverless is composed of both compute and cloud services.
- **Storage**: Calculated on a monthly basis on the average number of on-disk bytes per day in database tables and internal stages. Costs calculated on a dollar value rate per TB based on capacity on demand, cloud provider, and region.
- **Ingress/Egress Data Transfer**: Data transfer charges apply from one region to another or from one cloud platform to another. Unloading data from Snowflake and also `COPY INTO`, replicating data to a Snowflake account in a different region or cloud platform and external functions moving data into and out of Snowflake.

### Credits

- Billed for virtual warehouse services, cloud services, and serverless services.

### Dollar Value

- Data storage and ingress/egress data transfer.

---

## Connectors and Drivers

Snowflake has connectors for all technologies such as Python, Go, PHP, .NET, Node.js, Spark, Kafka, JDBC/ODBC drivers. JDBC/ODBC drivers allow third-party tools to connect to Snowflake. Users are able to add Snowflake connectors into their code base, add username/password and then add in their SQL statement.

### Partner Tools

- **BI Tools**: Tableau, PBI, Qlik
- **Data Integration**: dbt, Fivetran
- **Security/Governance**: Collibra, Datadog, HashiCorp Vault
- **SQL Dev and Management**: Various tools
- **ML/DS**: SageMaker, Dataiku, DataRobot

---

## Snowflake Scripting

Another way to interact with data is to use Snowflake scripting, it is an extension of SnowflakeSQL. Used to write stored procedures and procedural code outside of a stored procedure. The code is written within a scripting block, with `DECLARE`, `BEGIN`, `EXCEPTION`, and `END`. Variables can be used within the scope of the block, they can also be declared and assigned in the `BEGIN` section with the `LET` keyword.

### Other Forms of Scripting

- **Branching Constructs**: If/else statements
- **Looping Constructs**: For/while loops
- **Cursor**: To loop through values in a table
- **ResultSet**: Like a cursor which defines a query

---

## Snowpark

Snowpark is an API which is accessed outside of Snowflake, allowing us to query and process our data which supports Java, Scala, and Python. Works via data frames and uses queries as add-ons e.g., `.select()`, `.join()`. It is lazily evaluated and executed, works via data frames (data frame is a 2D table of rows and columns like a spreadsheet).

After calling the data frame, users are able to transform the data and then print.
