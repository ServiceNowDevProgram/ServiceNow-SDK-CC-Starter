# SYS_DATA_SOURCE

<!-- Related skill: importing-data -->

## Data Source object

Create a data source [sys_data_source] using the Record API.

## Properties

| Name    | Type             | Description                                                                                                                                                                                                            |
| ------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `$id`   | String or Number | Required. A unique ID for the metadata object provided in the following format, where `<value>` is a string or number: `$id: Now.ID[<value>]`. When you build the application, this ID is hashed into a unique sys_ID. |
| `table` | String           | Required. Must be `'sys_data_source'` for data source records.                                                                                                                                                         |
| `data`  | Object           | Required. Fields and their values for the data source configuration. See data object properties below.                                                                                                                 |
| `$meta` | Object           | Metadata for the application metadata. With the `installMethod` property, you can map the application metadata to an output directory that loads only in specific circumstances.                                       |

## data object

Configure the data source properties within the data object.

### Core Properties

| Name                     | Type    | Max Length | Description                                                                                                                                         |
| ------------------------ | ------- | ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`                   | String  | 40         | Required. The unique name of the data source.                                                                                                       |
| `type`                   | String  | 40         | Required. Type of data source. Valid values: `File`, `JDBC`, `data_stream`, `LDAP`, `OIDC`, `REST`, `CUSTOM`                                        |
| `import_set_table_name`  | String  | 80         | Required. Name of the import set table to create/use for staging imported data. Prefix with scope name.                                             |
| `import_set_table_label` | String  | 40         | Display label for the import set table.                                                                                                             |
| `active`                 | Boolean | 40         | Flag that indicates whether the data source is active. Valid values: • true: Data source is active. • false: Data source is inactive. Default: true |

### File-Based Properties

Use these properties when `type` is `'File'`:

| Name                    | Type    | Max Length | Description                                                                                                                                                         |
| ----------------------- | ------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `format`                | String  | 100        | Required for File type. Format of the file. Valid values: `CSV`, `CSV (tab)`, `XML`, `JSON`, `Excel`, `Custom (Parse by Script)`                                    |
| `file_path`             | String  | 1000       | Path to the source file.                                                                                                                                            |
| `file_retrieval_method` | String  | 40         | Method for retrieving files. Valid values: `Attachment`, `FTP`, `SCP`, `SFTP`, `HTTP`, `HTTPS`                                                                      |
| `csv_delimiter`         | String  | 40         | Delimiter character for CSV files. Default: `,`                                                                                                                     |
| `header_row`            | Integer | 40         | Row number containing column headers in spreadsheets. Default: 1                                                                                                    |
| `sheet_name`            | Integer | 40         | Excel sheet name for spreadsheet imports.                                                                                                                           |
| `sheet_number`          | Integer | 40         | Excel sheet number for spreadsheet imports (1-based).                                                                                                               |
| `zipped`                | Boolean | 40         | Flag that indicates whether the source file is compressed. Valid values: - true: File is compressed. - false: File is not compressed. Default: false                |
| `xpath_root_node`       | String  | 100        | MANDATORY for XML format. XPath expression defining each row in XML data (e.g., `//record`, `/root/items/item`). Data source will fail without this field.          |
| `jpath_root_node`       | String  | 100        | MANDATORY for JSON format. JSONPath expression defining each row in JSON data (e.g., `$.employees[*]`, `$.data.records`). Data source will fail without this field. |

#### File Retrieval Method - Field Applicability Matrix

The following table shows which fields are required or applicable based on the `file_retrieval_method` value:

| Field Name                                                              | Attachment | HTTP/HTTPS | FTP      | SCP      | SFTP     | Notes                                                  |
| ----------------------------------------------------------------------- | ---------- | ---------- | -------- | -------- | -------- | ------------------------------------------------------ |
| Connection Fields                                                       |
| `file_path`                                                             | N/A        | Required   | Required | Required | Required | URL for HTTP/HTTPS, file path for FTP/SCP/SFTP         |
| `scp_server`                                                            | N/A        | N/A        | Required | Required | Required | Server hostname or IP                                  |
| `scp_port`                                                              | N/A        | N/A        | Optional | Optional | Optional | Default: 21 (FTP), 22 (SCP/SFTP)                       |
| `scp_user_name`                                                         | N/A        | N/A        | Required | Required | Required | Leave empty - user sets manually                       |
| `scp_password`                                                          | N/A        | N/A        | Required | Required | Required | Leave empty - user sets manually                       |
| `scp_authentication`                                                    | N/A        | N/A        | Optional | Optional | Optional | Authentication method                                  |
| `ssh_keyfile_path`                                                      | N/A        | N/A        | N/A      | Optional | Optional | For SSH key authentication                             |
| Format-Specific Fields (apply to all retrieval methods based on format) |
| `csv_delimiter`                                                         | Optional   | Optional   | Optional | Optional | Optional | For CSV format only                                    |
| `header_row`                                                            | Optional   | Optional   | Optional | Optional | Optional | For CSV/Excel formats                                  |
| `sheet_name`                                                            | Optional   | Optional   | Optional | Optional | Optional | For Excel - either sheet_name OR sheet_number required |
| `sheet_number`                                                          | Optional   | Optional   | Optional | Optional | Optional | For Excel - either sheet_name OR sheet_number required |
| `xpath_root_node`                                                       | Required   | Required   | Required | Required | Required | Required for XML format - defines row structure        |
| `jpath_root_node`                                                       | Required   | Required   | Required | Required | Required | Required for JSON format - defines row structure       |
| `zipped`                                                                | Optional   | Optional   | Optional | Optional | Optional | If source file is compressed                           |

Important Notes:

- Attachment method: File is uploaded directly to ServiceNow. No connection fields needed.
- HTTP/HTTPS method: For publicly accessible URLs. No authentication fields needed (unless using Connection Alias).
- FTP/SCP/SFTP methods: Require manual credential configuration after deployment.
- Format-specific fields: Apply to all retrieval methods but depend on the `format` value (CSV, Excel, XML, JSON).
- **CRITICAL Mandatory Fields**:
  - **ALL data sources**: `name`, `type`, `import_set_table_name` (always required)
  - **File type**: `format` field required (CSV, Excel, XML, JSON)
  - **XML format**: `xpath_root_node` required (defines row structure)
  - **JSON format**: `jpath_root_node` required (defines row structure)
  - **JDBC type**: `format` (driver class) + either `table_name` OR `sql_statement` required

### Database (JDBC) Properties

Use these properties when `type` is `'JDBC'`:

| Name                            | Type     | Max Length | Description                                                                                                                                                                                                                                                                                                              |
| ------------------------------- | -------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `format`                        | String   | 100        | Required for JDBC type. JDBC driver class name. Valid values: <br>• `org.mariadb.jdbc.Driver` - MySQL/MariaDB<br>• `oracle.jdbc.OracleDriver` - Oracle<br>• `com.microsoft.sqlserver.jdbc.SQLServerDriver` - SQL Server<br>• `com.ibm.db2.jcc.DB2Driver` - DB2 Universal<br>• `com.sybase.jdbc3.jdbc.SybDriver` - Sybase |
| `connection_url`                | String   | 1000       | JDBC connection URL for database connections.                                                                                                                                                                                                                                                                            |
| `connection_url_parameters`     | String   | 100        | Connection URL properties for additional JDBC parameters.                                                                                                                                                                                                                                                                |
| `database_name`                 | String   | 40         | Name of the database to connect to.                                                                                                                                                                                                                                                                                      |
| `jdbc_server`                   | String   | 40         | Database server hostname or IP address.                                                                                                                                                                                                                                                                                  |
| `database_port`                 | String   | 40         | Port number for database connection.                                                                                                                                                                                                                                                                                     |
| `jdbc_user_name`                | String   | 40         | Database username for authentication.                                                                                                                                                                                                                                                                                    |
| `jdbc_password`                 | Password | 40         | Database password (encrypted).                                                                                                                                                                                                                                                                                           |
| `table_name`                    | String   | 40         | **Required if not using sql_statement**. Name of the source table in database.                                                                                                                                                                                                                                           |
| `sql_statement`                 | String   | 4000       | **Required if not using table_name**. SQL query for database imports.                                                                                                                                                                                                                                                    |
| `database_primary_key`          | String   | 40         | Primary key field in the database table.                                                                                                                                                                                                                                                                                 |
| `oracle_port`                   | String   | 40         | Port for Oracle database connections.                                                                                                                                                                                                                                                                                    |
| `oracle_sid`                    | String   | 40         | Oracle System Identifier.                                                                                                                                                                                                                                                                                                |
| `use_integrated_authentication` | Boolean  | 40         | Flag that indicates whether to use integrated authentication. Valid values: • true: Use integrated authentication. • false: Use username/password. Default: false                                                                                                                                                        |

### LDAP Properties

Use these properties when `type` is `'LDAP'`:

| Name                        | Type      | Max Length | Description                          |
| --------------------------- | --------- | ---------- | ------------------------------------ |
| `ldap_target`               | Reference | 32         | Reference to LDAP OU Definition      |
| `ldapprobe_result_set_rows` | Integer   | -          | Maximum rows for LDAP probe results. |

### Connection & Authentication Properties

| Name                 | Type      | Max Length | Description                                                                                        |
| -------------------- | --------- | ---------- | -------------------------------------------------------------------------------------------------- |
| `connection_alias`   | Reference | 32         | Reference to Connection & Credential Alias (sys_alias table).                                      |
| `connection_timeout` | Integer   | 40         | Connection timeout in seconds. Default: 30                                                         |
| `query_timeout`      | Integer   | 40         | Query timeout in seconds.                                                                          |
| `mid_server`         | Reference | 32         | Reference to MID Server for connection (ecc_agent table). Required for internal network resources. |
| `scp_server`         | String    | 40         | SCP server hostname or IP address.                                                                 |
| `scp_port`           | String    | 40         | SCP server port number.                                                                            |
| `scp_authentication` | String    | 40         | Authentication method for SCP connections.                                                         |
| `scp_user_name`      | String    | 40         | SCP username for authentication.                                                                   |
| `scp_password`       | Password  | 255        | SCP password (encrypted).                                                                          |
| `ssh_keyfile_path`   | String    | 40         | Path to SSH key file for authentication.                                                           |

### Processing & Performance Properties

| Name                      | Type    | Max Length | Description                                                                                                                                                     |
| ------------------------- | ------- | ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `batch_size`              | Integer | 40         | Number of records to process in each batch. Recommended: 100-2000                                                                                               |
| `limit`                   | Integer | 40         | Maximum number of records to import per run.                                                                                                                    |
| `maximum_rows`            | Integer | 40         | Maximum rows to process.                                                                                                                                        |
| `offset`                  | Integer | 40         | Starting offset for data retrieval (for pagination).                                                                                                            |
| `use_batch_import`        | Boolean | 40         | Flag that indicates whether to use batch import processing. Valid values: • true: Use batch import. • false: Don't use batch import. Default: false             |
| `enable_parallel_loading` | Boolean | 40         | Flag that indicates whether to enable parallel data loading. Valid values: • true: Enable parallel loading. • false: Disable parallel loading. Default: false   |
| `support_pagination`      | Boolean | 40         | Flag that indicates whether the data source supports pagination. Valid values: • true: Supports pagination. • false: Doesn't support pagination. Default: false |

### Incremental Import Properties

| Name                                    | Type               | Max Length | Description                                                                                                                                                                     |
| --------------------------------------- | ------------------ | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `use_last_run_datetime`                 | Boolean            | 40         | Flag that indicates whether to use last run datetime for incremental imports. Valid values: • true: Use last run datetime. • false: Don't use last run datetime. Default: false |
| `last_run_database_field`               | String             | 40         | Database field for tracking last run.                                                                                                                                           |
| `last_run_datetime`                     | DateTime           | 40         | Last run date/time for incremental imports.                                                                                                                                     |
| `last_success_import_time`              | DateTime           | 40         | Timestamp of last successful import.                                                                                                                                            |
| `connection_override_last_success_time` | Simple Name Values | 4000       | Connection override last success time configuration.                                                                                                                            |

### Advanced Properties

| Name                           | Type      | Max Length | Description                                                                                                                                                      |
| ------------------------------ | --------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `data_loader`                  | Script    | 8000       | Custom script for data loading logic.                                                                                                                            |
| `parsing_script`               | Script    | 8000       | Script for custom data parsing and transformation.                                                                                                               |
| `parallel_loading_script`      | Script    | 8000       | Script for parallel loading logic.                                                                                                                               |
| `data_in_single_column`        | Boolean   | 40         | Flag that indicates whether all data is in a single column. Valid values: • true: Data is in single column. • false: Data is in multiple columns. Default: false |
| `discard_arrays`               | Boolean   | 40         | Flag that indicates whether to discard array data structures in JSON/XML. Valid values: • true: Discard arrays. • false: Keep arrays. Default: false             |
| `expand_node_children`         | Boolean   | 40         | Flag that indicates whether to expand XML/JSON node children. Valid values: • true: Expand children. • false: Don't expand. Default: false                       |
| `category`                     | String    | 40         | Category for organizing data sources.                                                                                                                            |
| `data_stream_action`           | Reference | 32         | Reference to Data Stream action (sys_hub_action_type_definition table).                                                                                          |
| `data_stream_action_inputs`    | Glide Var | 32         | Reference to Data stream action inputs (sys_hub_action_input table).                                                                                             |
| `data_stream_connection_alias` | Reference | 32         | Reference to Data stream connection alias (sys_alias table).                                                                                                     |
| `request_action`               | Reference | 32         | Reference to Action Type for web services (sys_hub_action_type_definition table).                                                                                |

## Related Table Schemas

When creating LDAP data sources, you need to create records in multiple related tables. This section documents the schema for these supporting tables.

### ldap_server_config Schema

The `ldap_server_config` table stores the base LDAP server configuration. This is the foundation that other LDAP-related records reference.

| Field             | Type                 | Required | Max Length | Description                                                                                                                         |
| ----------------- | -------------------- | -------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `name`            | String               | Yes      | 100        | Unique name for the LDAP server configuration. Used for identification.                                                             |
| `server_url`      | String               | Yes      | 255        | Primary LDAP server URL. Format: `ldap://hostname:port` or `ldaps://hostname:port`                                                  |
| `port`            | Integer              | No       | -          | LDAP server port. Default: 389 (ldap), 636 (ldaps).                                                                                 |
| `rdn`             | String               | No       | 255        | Root Distinguished Name - the starting point for LDAP searches.                                                                     |
| `dn`              | String               | Yes      | 255        | Bind Distinguished Name - the DN used for authentication.                                                                           |
| `password`        | Password (Encrypted) | Yes      | 255        | **LEAVE EMPTY in code** - Password for the bind DN. User must set manually in ServiceNow after deployment for security.             |
| `active`          | Boolean              | No       | -          | Whether this LDAP server configuration is active. Default: `true`                                                                   |
| `ssl`             | Boolean              | No       | -          | Whether to use SSL/TLS encryption. Set to `true` for ldaps:// URLs. Default: `false`                                                |
| `authenticate`    | Boolean              | No       | -          | Whether to authenticate with the LDAP server using the provided DN/password. Default: `true`                                        |
| `connect_timeout` | Integer              | No       | -          | Connection timeout in seconds. Default: 10. Range: 1-300.                                                                           |
| `read_timeout`    | Integer              | No       | -          | Read timeout in seconds. Default: 30. Range: 1-600.                                                                                 |
| `paging`          | Boolean              | No       | -          | Whether to use LDAP paging for large result sets. Recommended: `true`. Default: `false`                                             |
| `vendor`          | String (Choice)      | No       | 40         | LDAP vendor/implementation type. Valid values: `active_directory`, `openldap`, `edirectory`, `domino`, `other`. Default: `openldap` |

**Table Relationships:**

- Referenced by: `ldap_ou_config.server`, `ldap_server_url.server`
- This is the **base configuration** that all other LDAP records reference

### ldap_ou_config Schema

The `ldap_ou_config` table defines Organizational Unit configurations for LDAP imports. Each OU config specifies which part of the LDAP directory to query and how to import that data.

| Field          | Type                | Required | Max Length | Description                                                                                          |
| -------------- | ------------------- | -------- | ---------- | ---------------------------------------------------------------------------------------------------- |
| `name`         | String              | Yes      | 100        | Unique name for this OU configuration. Used for identification.                                      |
| `server`       | Reference           | Yes      | 32         | **Reference to `ldap_server_config` record object** - Defines which LDAP server this OU belongs to.  |
| `ou`           | String              | Yes      | 255        | Organizational Unit Distinguished Name. Defines the search base in the LDAP directory.               |
| `filter`       | String              | No       | 1000       | LDAP filter expression to limit which objects are imported.                                          |
| `active`       | Boolean             | No       | -          | Whether this OU configuration is active and available for imports. Default: `true`                   |
| `query_field`  | String              | No       | 40         | Additional LDAP query field for refined searches. Advanced use case.                                 |
| `map`          | Reference           | No       | 32         | Reference to `sys_impex_map` - Import/export mapping configuration. Used for complex field mappings. |
| `table`        | String (Table Name) | No       | 80         | Target ServiceNow table for imports. If not specified, uses the data source's import set table.      |
| `entry_sys_id` | String              | No       | 32         | System identifier for the LDAP entry. Internal use.                                                  |

**Table Relationships:**

- References: `ldap_server_config` (via `server` field)
- Referenced by: `sys_data_source.ldap_target`
- This is the **middle layer** that connects the LDAP server to the data source

**Field Usage Notes:**

- `server`: **CRITICAL** - Must reference the record object, NOT a sys_id string or Now.ID. Correct: `server: ldapServer`
- `ou`: Must be a valid LDAP Distinguished Name. Use the full path from root.
- `filter`: Standard LDAP filter syntax. Empty filter imports all objects in the OU.
- `table`: Only needed if importing directly to a table (bypassing import set transformations)

### ldap_server_url Schema

The `ldap_server_url` table defines connection endpoints for LDAP servers. Multiple URLs can be configured for failover and load balancing.

| Field                | Type      | Required | Max Length | Description                                                                                                                                 |
| -------------------- | --------- | -------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `server`             | Reference | Yes      | 32         | **Reference to `ldap_server_config` record object** - Defines which LDAP server configuration this URL belongs to.                          |
| `url`                | String    | Yes      | 255        | LDAP server URL with protocol, hostname/IP, and port. Format: `ldap://hostname:port` or `ldaps://hostname:port`.                            |
| `active`             | Boolean   | No       | -          | Whether this URL is active and available for connections. Default: `true`. Set to `false` to temporarily disable.                           |
| `order`              | Integer   | No       | -          | Priority order for connection attempts. **Lower numbers are tried first**. Default: 100. Use increments of 100 (100, 200, 300) for clarity. |
| `operational_status` | Boolean   | No       | -          | Indicates if the server URL is operationally available. ServiceNow may update this based on connection health. Default: `true`              |

**Table Relationships:**

- References: `ldap_server_config` (via `server` field)
- This table is **optional** - Only needed for multiple URLs (failover/load balancing)

**Field Usage Notes:**

- `server`: **CRITICAL** - Must reference the record object, NOT a sys_id string or Now.ID. Correct: `server: ldapServer`
- `url`: Must include protocol (ldap:// or ldaps://), hostname, and port. Use ldaps:// for production.
- `order`: Lower values = higher priority. Same order values = load balancing. Different order values = failover.
- **Optional Record**: Only create ldap_server_url records if you need multiple URLs for high availability

## Examples

### Example 1: File type Data Source with CSV format and Attachment file retrieval method

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const csvDataSource = Record({
  $id: Now.ID["csv-user-import"],
  table: "sys_data_source",
  data: {
    name: "User CSV Import",
    type: "File",
    format: "CSV",
    file_retrieval_method: "Attachment",
    csv_delimiter: ",",
    header_row: 1,
    import_set_table_name: "u_import_users",
    import_set_table_label: "User Import Staging",
    batch_size: 500,
    use_batch_import: true,
    active: true
  }
});
```

### Example 2: File type Data Source with Excel format and Attachment file retrieval method

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const employeeExcelDataSource = Record({
  $id: Now.ID["employee_excel_datasource"],
  table: "sys_data_source",
  data: {
    name: "employee_excel_datasource",
    type: "File",
    description:
      "Excel file upload data source for importing Health Employee Data",

    // File Configuration
    format: "Excel",
    file_retrieval_method: "Attachment",
    sheet_number: 1,
    header_row: 1,

    // Import Set Configuration
    import_set_table_name: "u_health_employee_data", // scope name + table label
    import_set_table_label: "Health Employee Data", // table label

    // Processing Configuration
    batch_size: 100,
    maximum_rows: 10000,
    use_batch_import: true,
    active: true
  }
});
```

### Example 3: LDAP Data Source with all required configurations

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

// Step 1: Create LDAP Server Configuration
export const ldapServer = Record({
  $id: Now.ID["users_ldap_server"],
  table: "ldap_server_config",
  data: {
    name: "users_ldap_server",
    // LDAP Connection - Pre-populated with user-provided values
    server_url: "ldap://ldap.company.com", // Pre-populated from user input
    port: 389, // Pre-populated from user input
    dn: "cn=admin,dc=company,dc=com", // Pre-populated from user input (bind DN)
    rdn: "", // Pre-populated from user input (root DN)
    password: "", // LEAVE EMPTY - User will set manually in ServiceNow
    active: true,
    ssl: false,
    authenticate: true,
    connect_timeout: 30,
    read_timeout: 60,
    paging: true,
    vendor: "openldap"
  }
});

// Step 2: Create LDAP OU Configuration
export const ldapOU = Record({
  $id: Now.ID["users_ou"],
  table: "ldap_ou_config",
  data: {
    name: "users_ou",
    ou: "ou=users,dc=company,dc=com", // Pre-populated from user input
    filter: "(uid=e*)", // Pre-populated from user input
    server: ldapServer, // Reference to LDAP server config
    active: true
  }
});

// Step 3: Create LDAP Server URL
export const ldapServerUrl = Record({
  $id: Now.ID["users_ldap_server_url"],
  table: "ldap_server_url",
  data: {
    active: true,
    operational_status: true,
    order: 100,
    server: ldapServer, // Reference to LDAP server config
    url: "ldap://ldap.company.com" // Pre-populated from user input
  }
});

// Step 4: Create Main LDAP Data Source
export const ldapDataSource = Record({
  $id: Now.ID["users_datasource"],
  table: "sys_data_source",
  data: {
    name: "users_datasource",
    type: "LDAP",
    description: "LDAP data source to import users from ldap.forumsys.com",

    // Import Set Configuration
    import_set_table_name: "u_users_import", // scope name + table label
    import_set_table_label: "Users Import", // table label

    // LDAP Configuration
    ldap_target: ldapOU, // Reference to LDAP OU config
    ldapprobe_result_set_rows: 1000,

    // Processing Configuration
    batch_size: 100,
    maximum_rows: 5000,
    connection_timeout: 60,
    query_timeout: 300,
    use_batch_import: true,
    active: true
  }
});
```

### Example 4: Database (JDBC) Data Source with all required configurations

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const databaseDataSource = Record({
  $id: Now.ID["test-jdbc"],
  table: "sys_data_source",
  data: {
    name: "Test JDBC",
    type: "JDBC",
    format: "org.mariadb.jdbc.Driver", // MySQL/MariaDB driver

    // Database Connection
    database_name: "glide_znowassistalchemist",
    database_port: "3306",
    jdbc_server: "localhost",
    jdbc_user_name: "",
    jdbc_password: "", // LEAVE EMPTY - User will set manually in ServiceNow

    // Query Configuration
    table_name: "task",
    query: "All Rows from Table",

    // Import Configuration
    import_set_table_name: "sn_import_set_exam_jdbc_sql_data",
    import_set_table_label: "JDBC SQL DATA",

    // Processing Configuration
    batch_size: 1000,
    connection_timeout: 0,
    query_timeout: 0,
    use_last_run_datetime: false,
    enable_parallel_loading: false,
    active: true
  }
});
```

### Example 5: Database (JDBC) Data Source with Oracle format

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const oracleDataSource = Record({
  $id: Now.ID["example-jdbc-oracle-location"],
  table: "sys_data_source",
  data: {
    name: "Example JDBC Oracle Location",
    type: "JDBC",
    format: "oracle.jdbc.OracleDriver", // Oracle driver

    // Database Connection
    jdbc_server: "xxx.service-now.com", // Pre-populated from user input
    oracle_sid: "sandb02", // Pre-populated from user input
    oracle_port: "1521", // Pre-populated from user input
    database_user_name: "", // Pre-populated from user input
    database_password: "", // LEAVE EMPTY - User will set manually

    // Query Configuration
    table_name: "AC_LOCATION", // table name
    query: "All Rows from Table",

    // Import Configuration
    import_set_table_name: "sn_import_set_exam_jdbc_oracle_data", // scope name + table label
    import_set_table_label: "JDBC ORACLE DATA", // table label

    // Processing Configuration
    use_last_run_datetime: false
  }
});
```

### Example 6: Database (JDBC) Data Source with SQL Server format

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const sqlServerDataSource = Record({
  $id: Now.ID["sql-data-source"],
  table: "sys_data_source",
  data: {
    name: "SQL Data Source",
    type: "JDBC",
    format: "com.microsoft.sqlserver.jdbc.SQLServerDriver", // SQL Server driver

    // Database Connection
    database_name: "glide_znowassistalchemist", // database name
    database_port: "3306", // database port
    instance_name: "MSSQLSERVER", // instance name
    jdbc_server: "localhost", // jdbc server
    database_user_name: "",
    database_password: "", // LEAVE EMPTY - User will set manually

    // Query Configuration
    table_name: "task", // table name
    query: "Specific SQL",
    sql_statement: "SELECT * FROM task WHERE state = 1", // sql statement

    // Import Configuration
    import_set_table_name: "sn_import_set_exam_sql_data", // scope name + table label
    import_set_table_label: "SQL Data", // table label

    // Processing Configuration
    batch_size: 1000,
    connection_timeout: 0,
    query_timeout: 0,
    use_last_run_datetime: false,
    enable_parallel_loading: false
  }
});
```

### Example 7: File type Data Source with Excel format and FTP file retrieval method

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const ftpExcelDataSource = Record({
  $id: Now.ID["ftp-excel-inventory-import"],
  table: "sys_data_source",
  data: {
    name: "Inventory Data Source",
    type: "File",
    format: "Excel",
    file_retrieval_method: "FTP",

    // FTP Configuration
    scp_server: "eu-central-1.sftpcloud.io",
    scp_user_name: "",
    scp_password: "", // LEAVE EMPTY - User will set manually in ServiceNow
    file_path: "/Downloads/inventory.xlsx",
    scp_port: "22",

    // Excel Configuration
    sheet_number: 1,
    header_row: 1,

    // Import Configuration
    import_set_table_name: "sn_import_set_exam_inventory_record_data", //scope name + table name
    import_set_table_label: "Inventory Record Data",
    batch_size: 1000
  }
});
```

### Example 8: XML File Data Source

example xml file

```xml
<products>
    <product>
        <name>Product 1</name>
        <description>Description 1</description>
    </product>
    <product>
        <name>Product 2</name>
        <description>Description 2</description>
    </product>
</products>
```

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const xmlDataSource = Record({
  $id: Now.ID["xml-product-import"],
  table: "sys_data_source",
  data: {
    name: "Product XML Import",
    type: "File",
    format: "XML",
    file_retrieval_method: "Attachment",

    // XML Configuration
    // MANDATORY: xpath_root_node is REQUIRED for XML format
    // Without this field, the data source will fail to import any data
    xpath_root_node: "/products/product", // XPath expression that defines each row
    expand_node_children: true,

    // Import Configuration
    import_set_table_name: "u_product_import",
    import_set_table_label: "Product Import",
    batch_size: 100,
    active: true
  }
});
```

### Example 9: File type Data Source with custom (parse by script) format and Attachment file retrieval method

example txt file

```txt
name: John Doe
age: 30
gender: male;
name: Jane Smith
age: 25
gender: female;
name: John lee
age: 22
gender: male;
```

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const customFormatDataSource = Record({
  $id: Now.ID["custom-format-import"],
  table: "sys_data_source",
  data: {
    name: "Employee Data Import",
    type: "File",
    format: "Custom (Parse by Script)",
    file_retrieval_method: "Attachment",
    parsing_script: `// The input value can be accessed through the variables named "line", "lineNumber" and "result".
// The function uses the 'result' variable to return parsed rows back to ServiceNow.
(function(line, lineNumber, result) {
    try {
        if (!line) return;

        // Split the line into individual record segments using ';'
        var recordSegments = line.split(';');

        for (var recordIndex = 0; recordIndex < recordSegments.length; recordIndex++) {
            var recordText = recordSegments[recordIndex].trim();
            if (!recordText) continue;

            // Split record into individual key-value strings
            var keyValuePairs = recordText.split(',');

            // Object to hold field names and their corresponding values
            var recordObject = {};

            for (var pairIndex = 0; pairIndex < keyValuePairs.length; pairIndex++) {
                var pairText = keyValuePairs[pairIndex];
                var keyValue = pairText.split('=');

                var fieldName = keyValue[0] ? keyValue[0].trim() : '';
                var fieldValue = keyValue[1] ? keyValue[1].trim() : '';

                if (fieldName)
                    recordObject[fieldName] = fieldValue;
            }

            // Add the parsed record as a new row in the import set
            result.addRow(recordObject);
        }

    } catch (error) {
        gs.log("Error in custom parsing script (line " + lineNumber + "): " + error);
    }
})(line, lineNumber, result);
`,

    // Import Configuration
    import_set_table_name: "test_custom_data",
    import_set_table_label: "Test Custom Data",
    batch_size: 100,
    active: true
  }
});
```

### Example 10: File type Data Source with JSON formated file and Attachment file retrieval method

example json file

```json
{
  "problems": {
    "problem": [
      {
        "name": "Problem 1",
        "description": "Description 1"
      },
      {
        "name": "Problem 2",
        "description": "Description 2"
      }
    ]
  }
}
```

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const jsonDataSource = Record({
  $id: Now.ID["json-problem-import"],
  table: "sys_data_source",
  data: {
    name: "Problem JSON Import",
    type: "File",
    format: "JSON",
    file_retrieval_method: "Attachment",

    // JSON Configuration
    // MANDATORY: jpath_root_node is REQUIRED for JSON format
    // Without this field, the data source will fail to import any data
    jpath_root_node: "/problems/problem", // JSONPath expression that defines each row

    // Import Configuration
    import_set_table_name: "u_problem_import",
    import_set_table_label: "Problem Import",
    batch_size: 100,
    active: true
  }
});
```

### Example 11: REST Integration Hub Data Source

**IMPORTANT**: REST data sources require `request_action` field that references an Integration Hub action. The Integration Hub action must have `action_template = DATASOURCE_REQUEST`.

**Code Pattern When Action is Found:**

```typescript
import '@servicenow/sdk/global'
import { Record } from '@servicenow/sdk/core'

// LLM searched sys_hub_action_type_definition and found action with:
// - name='REST GET' (or user-provided name)
// - action_template='DATASOURCE_REQUEST'
// - sys_id='abc123...'


export const restJsonDataSource = Record({
    $id: Now.ID['rest-json-import'],
    table: 'sys_data_source',
    data: {
        name: 'REST API JSON Import',
        type: 'REST',
        format: 'JSON',

        // REST Configuration
        request_action: 'abc123...', // sys_id of Integration Hub action

        // MANDATORY for JSON: jpath_root_node is REQUIRED
        jpath_root_node: 'results/result'
        // Import Configuration
        import_set_table_name: 'u_rest_json_import',
        import_set_table_label: 'REST JSON Import',
        batch_size: 100,
        active: true,
    },
})
```

**Code Pattern When Action NOT Found (use placeholder):**

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

// LLM searched sys_hub_action_type_definition but found NO action with action_template='DATASOURCE_REQUEST'

export const restJsonDataSource = Record({
  $id: Now.ID["rest-json-import"],
  table: "sys_data_source",
  data: {
    name: "REST API JSON Import",
    type: "REST",
    format: "JSON",

    // TODO: MANUAL CONFIGURATION REQUIRED
    // No compatible Integration Hub action found with action_template='DATASOURCE_REQUEST'
    // After deployment: Navigate to System Definition > Data Sources > [this record]
    // Set 'Request action' field to your Integration Hub action
    // request_action: <reference to sys_hub_action_type_definition>,

    // MANDATORY for JSON: jpath_root_node is REQUIRED
    jpath_root_node: "results/result",

    // Import Configuration
    import_set_table_name: "u_rest_json_import",
    import_set_table_label: "REST JSON Import",
    batch_size: 100,
    active: true
  }
});
```

**Manual Configuration Instructions (When Action Not Found):**

```
MANUAL CONFIGURATION REQUIRED AFTER DEPLOYMENT:

1. Navigate to: System Definition > Data Sources
2. Open the data source record: "REST API JSON Import"
3. In the "Request action" field:
   a. Click the search icon
   b. Search for your Integration Hub action
   c. IMPORTANT: Verify action_template='DATASOURCE_REQUEST'
   d. Select the action
4. Save the record

NOTE: Only Integration Hub actions with action_template='DATASOURCE_REQUEST'
are compatible with data sources.
```

### Example: CSV Import from HTTP Server

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const httpCsvImport = Record({
  $id: Now.ID["http-csv-employee-import"],
  table: "sys_data_source",
  data: {
    name: "HTTP CSV Employee Import",
    type: "File",
    format: "CSV",
    file_retrieval_method: "HTTP",

    // HTTP Configuration
    file_path: "http://localhost:8000/Downloads/employees.csv",
    scp_port: 8000,
    scp_server: "localhost",

    // CSV Configuration
    csv_delimiter: ",",
    header_row: 1,

    // Import Configuration
    import_set_table_label: "Employee Import",
    import_set_table_name: "sn_import_set_exam_employee_import", //scope name + table name
    batch_size: 100,
    active: true
  }
});
```
