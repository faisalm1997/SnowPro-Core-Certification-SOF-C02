# Virtual Warehouses

A virtual warehouse is a named abstraction for a massively parallel processing (MPP) compute cluster. Virtual warehouses execute DML, DQL, and data loading operations such as `SELECT`, `UPDATE`, and `COPY INTO`. A user can only interact with the named warehouse object and not the underlying compute resources.

A user can spin up and shut down an unlimited number of warehouses without resource contention. Configuration can be changed on the fly, and they contain local SSD storage which is used to store raw data retrieved from the storage layer.

Virtual warehouses can be created using the Snowflake UI or through SQL commands:

- `CREATE WAREHOUSE ... WAREHOUSE_SIZE = 'MEDIUM'`
- `DROP WAREHOUSE MY_WAREHOUSE`
- `ALTER WAREHOUSE MY_WH SUSPEND`

---

## Warehouse State and Properties

Warehouses can be in the started, suspended, or resizing state.

- **Started**: Currently active, consuming credits.
- **Suspended**: Currently shut down, not consuming credits.
- **Resizing**: Sizing up or down.

By default, a virtual warehouse is created in the started state. Suspending a warehouse can put it within the suspended state which removes the compute nodes. Resuming a virtual warehouse puts it back in the started state and can execute queries.

### State Properties

- **Auto Suspend**: Specifies the number of seconds of inactivity after which a warehouse is automatically suspended.
- **Auto Resume**: Specifies whether to auto resume when a SQL statement is submitted to it.
- **Initially Suspended**: Whether the warehouse is created initially in the suspended state.

Virtual warehouses can be created in 10 t-shirt sizes ranging from x-small to 6x-large. Underlying compute power doubles with each size. The larger the warehouse, the better the query performance. Choosing the right size of the warehouse is done by experimenting with the query of a workload.

Data loading does not require a large virtual warehouse; sizing up does not guarantee increased data loading performance.

The credits used on each warehouse size double as the size increases. The first 60 seconds after the warehouse is provisioned, per hour, the credits start to accumulate. Credit price is determined by region and Snowflake edition.

### Resource Monitors

Objects which allow users to set credit limits on user-managed warehouses. The monitors can be at either account or warehouse level. Limits can be set for a specified time or date range. When limits are reached, actions can be triggered. The monitors can only be created by ACCOUNTADMIN.

```sql
CREATE RESOURCE MONITOR 
WITH CREDIT QUOTA = 100                         -- Number of credits allocated to the resource monitor per interval 
FREQUENCY = MONTHLY                             -- Time interval
START_TIMESTAMP = '2023-01-03 00:00 GMT'        -- Timestamp for when the resource monitor starts
TRIGGERS ON 50 PERCENT DO NOTIFY                -- Triggers and their condition
         ON 75 PERCENT DO NOTIFY
         ON 95 PERCENT DO SUSPEND 
         ON 100 PERCENT DO SUSPEND_IMMEDIATE;