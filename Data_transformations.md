# Data transformation

Data transformations can happen in Snowflake via Snowflake functions, estimation functions, unstructured file functions, file support REST API, directory tables, and table sampling.

## Snowflake Supported Function Types

- **Scalar**: A function which returns one value per invocation. These are used for returning one value per row.
- **Aggregate**: Operate on values across rows which can perform mathematical calculations such as sum, average, and counting.
- **Window**: Subset of aggregate functions, allowing us to aggregate on a subset of rows as input to a function.
- **Table**: Returns a set of rows for each input row. The returned set can contain zero, one, or more rows. Each row can contain one or more columns.
- **System Functions $SYSTEM**: Provides a way to execute actions in the system and helps to provide information about the system. Can also provide information about queries. e.g., `SELECT SYSTEM$PIPE_STATUS(Pipe_name)`

## Estimation Functions

### Cardinality Estimation

Estimate the number of distinct values. Snowflake has implemented something called the HyperLogLog cardinality estimation algorithm which returns an approximation of the distinct number of values of a column.

- `HLL()`
- `HLL_ACCUMULATE()`
- `HLL_COMBINE()`
- `HLL_ESTIMATE()`
- `HLL_EXPORT()`
- `HLL_IMPORT()`

### Similarity Estimation

Estimate the similarity of two or more sets. Snowflake implemented a two-step process to estimate similarity, without the need to compute the intersection or union of two sets. A value of 0 means no overlap between two data sets.

### Frequency Estimation

Estimate frequency values in a set. Snowflake has implemented functions which use the space-saving algorithm to produce an estimation of values and their frequencies.

- `APPROX_TOP_K`
- `APPROX_TOP_K_ACCUMULATE`
- `APPROX_TOP_K_COMBINE`
- `APPROX_TOP_K_ESTIMATE`

### Percentile Estimation

Estimate the percentile of values in a set. Snowflake has implemented the t-digest algorithm as an efficient way of estimating approximate percentile values in data sets.

- `APPROX_PERCENTILE`
- `APPROX_PERCENTILE_ACCUMULATE`
- `APPROX_PERCENTILE_COMBINE`
- `APPROX_PERCENTILE_ESTIMATE`

## Table Sampling

A convenient way to read a random subset of rows from a table. Table sampling can be fraction-based.

- **Examples**: `SAMPLE BERNOULLI/ROW`, `SAMPLE SYSTEM/BLOCK`, `REPEATABLE/SEED`
- **Fixed Size Sampling**: Does not support block sampling and the use of seed. Adding these will result in an error.

## Unstructured Data

Unstructured data can come in the form of multimedia and documents.

### File Functions

Unstructured data files > stage > select file functions > URL

- **BUILD_SCOPED_FILE_URL**: When this function is called in a query, must have USAGE privileges on an external named stage and READ privileges on an internal named stage. Used for only 24 hours.
- **BUILD_STAGE_FILE_URL**: Calling this function, whether it's part of a query, UDF, stored procedures, or view, requires privileges on the underlying stage. It is USAGE for external stages or READ for internal stages.
- **GET_PRESIGNED_URL**: Calling this function, whether it's part of a query, UDF, stored procedures, or view, requires privileges on the underlying stage. It is USAGE for external stages or READ for internal stages.

## Directory Tables

Directory tables can have data loaded from both internal and external stages.

- **Example**: `SELECT * FROM DIRECTORY(@INT_STAGE)`

Directory tables must be refreshed to reflect the most up-to-date changes made to stage contents. This can include new files being uploaded, removed files, and changes to files in the path.

### External Stages

- Can enable a directory table on an external stage.
- Can describe stage and retrieve ARN for Snowflake-managed SQS in the field `directory_notification_channel`.
- Can configure event notifications for S3 bucket to notify Snowflake-managed SQS associated with the stage when new or updated table is available to read into the directory table metadata.

## File Support REST API

User cannot download pre-signed file URL.

- **GET**: `/Api/Files`

### There are Two Ways

- **SCOPED URL**: `BUILD_SCOPED_FILE_URL()`
  - **Authorization**: Only the user who generated the scoped URL can download the staged file.
- **FILE URL**: `BUILD_STAGE_FILE_URL()`
  - **Authorization**: Any role that has privileges on the underlying stage can access the file
  