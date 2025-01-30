# Access Control Overview

Access control is about who can perform what operations on which objects. Snowflake uses two access control systems:

## Role-Based Access Control (RBAC)

RBAC assigns access privileges to roles, which are then assigned to users.

- **Privilege > Role > User**

## Discretionary Access Control (DAC)

DAC determines which object has which owner and grants access to that object.

- **Role owns object > Role A grants permissions to Role B > Role B selects object and can access it**

## Securable Objects

Nearly all objects in Snowflake are securable. You can grant access to a user to access a certain object but may not be able to give it access to modify its configuration.

Every securable object is owned by a single role. This can be found by executing the `SHOW object` command. The role that owns the object can grant or revoke privileges to other roles on the object, as well as have full control of the object and share it.

Unless allowed by a grant, access to a securable object will be denied.

---

## Roles

A role is an entity to which privileges on securable objects can be granted or revoked. They are assigned to users to give them the authorization to perform actions. A user can have multiple roles and switch between them.

Note: A role is a securable object which means you grant a role to another role and create a role hierarchy. Privileges of child roles are inherited by parent roles.

### System-Defined Roles

Snowflake has six system-defined roles:

- **ORGADMIN**: Manages operations at the organizational level, creates and views accounts in an organization, and views usage info across the organization.
- **ACCOUNTADMIN**: Top-level and most powerful role for an account, views and operates all objects in an account, manages billing and credit, stops any running SQL statements, encapsulates SYSADMIN and SECURITYADMIN.
- **SECURITYADMIN**: Manages grants globally via MANAGE GRANTS. Can create, monitor, and manage users and roles.
- **SYSADMIN**: Can create warehouses, databases, schemas, and other objects in an account, and give custom roles permissions to SYSADMIN.
- **USERADMIN**: Manages users and roles, can create users and roles in an account.
- **PUBLIC**: Auto-granted to any user or role in an account. Owns secure objects and is available to any user and role in an account.

### Custom Roles

Custom roles allow you to create a role with custom and fine-grained security privileges. This allows admins to work with system-defined roles. Custom roles can be created by SECURITYADMIN and USERADMIN as well as any role which has the CREATE ROLE privilege.

---

## Privileges

A security privilege defines a level of access to an object. For each object, there is a set of security privileges that can be granted to it.

There are four categories of security privileges:

- Global privileges
- Privileges for account objects
- Privileges for schemas
- Privileges for schema objects

Privileges are maintained by using the `GRANT` and `REVOKE` commands.

---

## User Authentication

The process of authenticating with Snowflake via user-provided username and password credentials. User authentication is the default method of authentication.

A password must be:

- Between 8-256 characters long
- Contain at least one digit
- Contain at least one uppercase letter and one lowercase letter

---

## Multi-Factor Authentication (MFA)

An additional layer of security which requires the user to prove their identity not only with a password but with an additional piece of information. MFA in Snowflake is controlled by Duo Security.

MFA is enabled on a per-user basis and only via the UI. It is recommended that all users who have ACCOUNTADMIN should enable MFA.

### MFA Properties

- **MINS_TO_BYPASS_MFA**: Minutes to temporarily disable MFA for the user so they can log in.
- **DISABLE_MFA**: Disables MFA for the user, canceling their enrollment. The user must re-enroll to use MFA.
- **ALLOWS_CLIENT_MFA_CACHING**: MFA token caching reduces the number of prompts required while connecting and authenticating to Snowflake.

---

## Federated Authentication (SSO)

Enables users to connect to Snowflake via SSO. Snowflake can delegate authentication responsibility to an SAML 2.0 compliant external provider. This has native support for Okta and ADFS (IdP). An IdP is an independent service provider responsible for creating and maintaining user credentials as well as authenticating users into Snowflake via SSO. In a federated environment, Snowflake is referred to as a service provider (SP).

### Federated Authentication Workflow

1. Clicks SSO button
2. Enters IdP credentials
3. Clicks on Snowflake application
4. Login successful

Users can specify an IdP as `SAML_IDENTITY_PROVIDER`. Users can create a security integration which combines two account-level properties, i.e., `SAML_IDENTITY_PROVIDER` and `SSO_LOGIN_PAGE`. The security integration holds all the configuration parameters held by the two account properties.

---

## Key-Pair Authentication

Users can generate key-pairs using OpenSSL. A key pair has a public and private key. You can assign the public key to the user and configure the appropriate Snowflake client. You can also configure key-pair rotation.

---

## OAuth

Snowflake can support the OAuth 2.0 protocol. This is an open standard protocol that allows supported clients authorized access to Snowflake without sharing or storing login credentials. Snowflake OAuth, external OAuth.

---

## SCIM

System for Cross-Domain Identity Management (SCIM) can be used to manage users and groups in cloud applications using RESTful APIs.

---

## Network Policies

Provide the user with the ability to allow or deny access to their Snowflake account based on a single IP or list of addresses.

Network policies are composed of an allowed and blocked IP range. Blocked IP ranges are applied first. Only support IPv4 addresses and use CIDR notation to express an IP subnet range. Can be applied at the account level or to individual users. If a user is associated with both an account-level and user-level network policy, the user-level policy takes precedence.

### Account-Level Network Policy

Only one network policy can be associated with one account at any one time. SECURITYADMIN and ACCOUNTADMIN can apply and attach policies using `ATTACH POLICY`.

### User-Level Network Policy

Only one network policy can be associated with any one user at any one time. SECURITYADMIN and ACCOUNTADMIN can apply and attach policies using `ATTACH POLICY`.

---

## Data Encryption

Data is encrypted at rest and in transit.

### Encryption at Rest

- AES-256 strong encryption
- Table data and internal stage data

### Encryption in Transit

- HTTPS TLS 1.2 encryption
- This sits between Snowflake and the Snowflake drivers such as ODBC, JDBC, Web UI, and SnowSQL.

### E2E Encryption Flow

1. User can `PUT` a file into an internal stage.
2. `COPY INTO` table from internal stage.
3. The same thing can happen with external stages where a cloud resource is used and then `COPY INTO` table.

### Hierarchy Key Model

- **root_key > account master keys > table master keys > file keys for each file**

Each table master key corresponds to one database table. Every table is encrypted with a separate key. Key hierarchies are used to segment data and limit the risk of key exposure and minimize key material. A key hierarchy reduces the attack surface if that key is compromised.

### Key Rotation

The practice of replacing existing account and table encryption keys every 30 days with a new key.

### Re-Keying

Once a retired key exceeds one year, Snowflake creates a new encryption key and re-encrypts all data protected by the retired key using the new key. Re-keying applies keys using the latest security standards.

### Tri-Secret Secure and Customer Managed Keys

Tri-secret secure allows you to encrypt data in Snowflake using a composite master key, one part customer-managed key and one part account-managed key by Snowflake. The customer-managed key is held in the key store of the cloud provider. Tri-secret secure allows for greater control over your data - making it impossible in Snowflake to decrypt data without the customer-managed key. If a data breach does happen, users can withdraw access to the customer-managed key which secures the data.

Snowflake support needs to be contacted to enable tri-secret secure.

---

## Column Level Security

### Dynamic Data Masking

Sensitive data is dynamically masked when loaded into Snowflake as plain text. It is masked at the time of query for unauthorized users. The query will return the sensitive data as it is for authorized users.

- **CREATE MASKING POLICY**

Masking policies can be applied to all columns and tables that you require. Data masking policies are schema-level objects like tables and views. Creating and applying these policies can be done by users. They can be nested in tables and views that reference those tables. A policy is applied no matter where the column is referenced in the SQL statement. It can be applied when the object is created or after the object is created.

### External Tokenization

Tokenization is replacing sensitive data with random characters/numbers. Tokenized data is loaded into Snowflake and detokenized at query run time. Authorized users can call external functions which will return external tokenization services with tokenized data.

---

## Row Level Security

Row access policies enable a security team to restrict which rows are returned in a query.

- **CREATE ROW ACCESS POLICY**

Row access policies are schema-level objects which means a schema/database must exist in Snowflake and have segregation of duties. They can be nested. Adding a masking policy to a column will fail if a row access policy is already applied. Row access policies are evaluated before data masking policies.

---

## Secure Views

A type of view which is designed to limit access to underlying tables or internal structured details of a view. Both standard and materialized views can be secure.

- **CREATE SECURE VIEW**

The definition of the secure view is only available to the owner and they can bypass query optimizations which can expose data in the underlying table.

---

## Account Usage

Snowflake provides a read-only database called Snowflake which has six schemas. They contain views providing data for fine-grained usage metrics at the account and object level.

Only users with ACCOUNTADMIN can access the Snowflake database. Account usage views can record dropped objects and there is a latency between an event and when the event was recorded (45 mins to 3 hours). The retention period for these views is one year.

### Information Schema

Each database created includes a schema called `information_schema` (SQL-92 ANSI). It contains views displaying the metadata for all objects contained in the database, metadata for all account-level objects, and table functions for historical and usage data across an account. It has no latency. Retention of data is from 7 days to 6 months which varies by view/table function.
