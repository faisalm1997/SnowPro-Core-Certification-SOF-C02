# Data Loading and Unloading

Data movement involves the following processes:

- User > stage > table
- User > stage > pipe > table > stage > user

You can use the `INSERT INTO <TABLE>` command or upload via the UI.

## Stages

Stages are temporary storage locations for data files used in the data loading and unloading process.

### Types of Stages

- **Internal**: User stage > Table stage > Named stage
- **External**: Named stage

### User Stage

These stages are automatically allocated when a user is created. The `PUT` command is used. They cannot be altered or dropped and are not appropriate if multiple users need access to the stage.

### Table Stage

Automatically allocated when a table is created. The `PUT` command is used. They cannot be altered or dropped. Must have ownership privileges on the table.

### Named Stage

User creates this stage. The `PUT` command is used. It is a securable object.

### External Stage and Storage Integrations

External stages refer to data files stored in a location outside Snowflake.

- User creates the object > cloud utilities are used > storage location can be private or public.
- `ON_ERROR` and `PURGE` can be set on stages.
- A storage integration is a reusable and securable Snowflake object which can be applied across stages and is recommended to avoid having to set sensitive information for each stage definition.

### Stage Helper Commands

- **LIST**: Lists the contents of a stage, including path/size of staged file/MD5 and last updated timestamp.
- **SELECT**: Query the contents of the staged file directly using SQL, useful to inspect files prior to data unloading and loading. Metadata columns such as filename and row numbers for a staged file.
- **REMOVE**: Remove files from internal or external stages, can specify a path for specific folders or files. Named and internal stages can include database and schema global pointers.

---

## PUT Command

The `PUT` command can upload data files from a local directory on a client machine to any of the three internal stages. `PUT` cannot be executed from within worksheets. Duplicate files uploaded by `PUT` are ignored. Uploaded files are encrypted with a 128-bit key with optional support for a 256-bit key.

---

## Bulk Loading

The `COPY INTO table` statement copies the contents of an internal or external stage or external location directly into a table. The `COPY INTO` statement requires a user to create a virtual warehouse to execute. Load history is stored in the metadata for 64 days to ensure the file is not duplicated.

Snowflake allows users to perform simple transformations on data as it is loaded into a table. Column reordering, column omission, casting, and truncating text strings that exceed a target length. Users can specify a set of fields to load from the staged data files using a SQL query.

Files can be loaded from external stages in the same way as internal stages. Data transfer billing may apply when loading data from files in a cloud storage service in a different region. Files can be copied directly from a cloud location such as S3. Snowflake recommends encapsulating the cloud storage service in an external stage such as S3 external tables.

### COPY INTO TABLE Validation

- **VALIDATION_MODE**: Optional parameter will allow you to perform a dry run of the load process to expose errors.
- **VALIDATE**: A table function to view all errors encountered during a previous `COPY INTO` execution. Accepts a job ID of a previous query or the last load operation executed.

### File Formats

Options can be set on a named stage or `COPY INTO` statement. File format options can be declared or they can be rolled up into independent file format Snowflake objects. File formats can be applied to both named stages and `COPY INTO` statements.

`COPY INTO` statements take precedence over file formats being declared. Users are able to set the file format for the object using the 'type' property. Each 'type' has its own properties which are related to parsing the specific file format.

If a file format is not defined, the default option is .csv in UTF-8 encoding.

---

## Snowpipe

A user is able to create a PIPE to ingest and stream data.

### Methods for Detecting Files within PIPES

1. Users can automate Snowpipe using cloud messaging services, only possible with external stages (e.g., SNS/SQS).
2. Users can call Snowpipe REST endpoints.

The pipe SQL code object can define `COPY INTO` statements that will execute when a file is uploaded to a stage.

#### Example: Cloud Messaging

- User > S3 Bucket/external stage (PUT Object) > SQS > SnowPipe (COPY INTO table) > Table

#### Example: REST Endpoint

- User > S3 Bucket/external stage > Pipe > COPY INTO > Table
                                        ^  
- User >>>>>>>>>>>>>>>>>>>>>>>>>>>> REST endpoint

Snowpipe is designed to load new raw data within a minute of a notification being sent via event-driven architecture. It uses serverless, Snowflake-managed compute resources to load data files, not a user-managed virtual warehouse. Snowpipe load history is stored in the metadata and can hold for 14 days. It is used to prevent reloading the same files. Even when the pipe is paused, event messages received by the pipe are held for 14 days.

### Bulk Loading vs Snowpipe

- **Authentication**: Bulk loading relies on the security options supported by the client for authenticating and using a session. Snowpipe needs key pair authentication with JSON Web Tokens to call REST endpoints.
- **Load History**: Bulk loading stores load history in the metadata of the target table for 64 days. Snowpipe stores load history for 14 days in the metadata of the pipe.
- **Compute Resources**: Bulk loading requires a user-specific warehouse to execute `COPY` statements. In Snowpipe, it uses Snowflake-supplied compute resources.
- **Billing**: Bulk loading is billed for the amount of time each virtual warehouse is active. For Snowpipe, Snowflake tracks the consumption of resources, per second/per core granularity. Snowpipe actively queues and processes data files. An overhead is included in the utilization costs charged for Snowpipe, 0.06 credits per 1000 files notified or listed via event notifications or REST API calls.

### Best Practices for Data Loading

1. 100-250MB compressed
2. Organize data by path
3. Load and then query
4. Pre-sort data
5. Snowpipe will load once per minute

---

## Data Unloading

Data can be unloaded from Snowflake using the `COPY INTO <location>` command instead of copying into a table for data loading.

The `GET` command is used to download a staged file to the local machine. By default, results unloaded to a stage using the `COPY INTO` command are split into multiple files. All data files which are unloaded to internal stages are encrypted by 128-bit keys.

Output files can be prefixed. You can `COPY INTO` and partition by, e.g., date to unload data into a directory structure. You can also `COPY INTO` a cloud provider's blob or S3 storage directly `COPY INTO 's3://dadsdfsdfasdf'` (S3 bucket location/URL).

### COPY INTO <location> Options

- **OVERWRITE**: Boolean value which specifies whether the `COPY` command will overwrite existing files with matching names.
- **SINGLE**: Boolean which specifies whether to generate a single file for output or multiple files.
- **MAX_FILE_SIZE**: Number > 0 specifies the upper limit in bytes of each file.
- **INCLUDE_QUERY_ID**: Boolean which specifies whether to add a UUID to the output file.

---

## GET Command

The `GET` command is the reverse of `PUT`.

- `GET` allows users to specify a source or stage and a local directory to download files to.
- `GET` cannot be used for external stages.
- `GET` cannot be executed within worksheets.
- Downloaded files are always encrypted.
- **Parallel**: Optional parameter specifies the number of threads to use for downloading files. Increasing this number can improve parallelization when downloading larger files.
- **Pattern**: Optional parameter which can specify a regex pattern for filtering files you want to download.

---

## Semi-Structured Data Overview

These are files of type JSON/XML. They have flexible schemas, so the number of fields can vary from one entity to the next. Semi-structured files normally contain a way in which they can self-describe each field. They also contain nested structures, arrays, and are composed of keys and values.

Snowflake has extended its capability to allow SQL to include semi-structured functions and notations for accessing data stored in those types.

### Semi-Structured Data Types

- **Array**: Contains 0 or more elements of data, each element is accessed by its position in the array.
- **Object**: Represents collections of key-value pairs.
- **Variant**: Universal semi-structured data type used to represent arbitrary data structures. Variant can hold up to 16MB compressed data per row.

### Semi-Structured Data Formats

- **JSON**: Plain text data format based on JavaScript (load/unload).
- **AVRO**: Binary row-based storage format based on Apache Hadoop (load).
- **ORC**: Highly efficient binary format used to store Hive data (load).
- **PARQUET**: Binary format designed for projects in Hadoop (load/unload).
- **XML**: Consists of tags <> and elements (load).

### Loading Semi-Structured Data

Semi-structured data files > stage > `COPY INTO` table. A file format is created after the stage and before copying into the table.

### Loading Approaches

- **ELT**: `CREATE TABLE` > `COPY INTO TABLE`, can only use variants.
- **ETL**: `CREATE TABLE` > `COPY INTO TABLE`, can use string, number, date, etc., data types.
- **Automatic Schema Detection**: User is able to infer the schema and then match by column name. Columns can be matched as case-sensitive or not - `INFER SCHEMA`.

### Accessing Semi-Structured Data

- **Dot Notation**: `SELECT src:employee.name FROM EMPLOYEES`, the column name is case-insensitive but key names are case-sensitive so the wrong query would result in an error.
- **Bracket Notation**: `SELECT SRC['employee']['name'] FROM EMPLOYEES`, the column name is case-insensitive but key names are case-sensitive so the wrong query would result in an error.
- **Repeating Element**: `SELECT SRC:skills[0] FROM EMPLOYEES`, you have to specify the element index `SELECT GET(SRC, 'skills')[0] FROM EMPLOYEE`.

### Casting Semi-Structured Data

You can specify a double colon `::`, `TO_datatype`, `AS_datatype` to convert values for a column to another data type.

### FLATTEN Table Function

`FLATTEN` is a table function which accepts values such as VARIANT, OBJECT, ARRAY and produces a row for each item. User is able to specify a path inside the object to flatten it. They can also flatten all sub-elements as recursive. The output of flatten: sequence, key, path, index, value and this (which is the element being flattened and is useful in recursive flattening).

A lateral flatten is a technique which is used to flatten a nested data structure such as a list or a tree. It can be flattened into a single flat list.
