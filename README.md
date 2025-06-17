# SQL Server to Fabric Lakehouse DDL Converter

This repository provides a Python-based solution for converting SQL Server `CREATE TABLE` statements into Spark SQL `CREATE TABLE` syntax for use in Microsoft Fabric Lakehouse (via notebooks).

## Features

- Converts `CREATE TABLE` DDLs from SQL Server to Spark SQL syntax
- Supports:
  - Column types with parameters (e.g., `VARCHAR(100)`, `DECIMAL(10,2)`)
  - Identity columns
  - `PRIMARY KEY`, `NOT NULL`, and `DEFAULT` clauses
  - Inline and table-level constraints
  - Inline comments
- Skips unsupported features such as `FOREIGN KEY` or `CHECK` constraints
- Outputs Spark-compatible SQL without executing it
- Detects and warns about unmapped data types
- Supports multi-table DDLs
- Schema name (`dbo.`) stripping is configurable

## Getting Started

### 1. Prepare your SQL Server DDL

Example:

```sql
CREATE TABLE [dbo].[Sales] (
    [EnrollmentId] INT IDENTITY(1,1) PRIMARY KEY,
    [CustomerName] VARCHAR(100) NOT NULL,
    [Amount] DECIMAL(10,2),
    [CreatedDate] DATETIME,
    [IsActive] BIT,
    CONSTRAINT PK_Sales PRIMARY KEY (EnrollmentId)
);

CREATE TABLE [dbo2].[Products] (
    [ProductId] INT IDENTITY(1,1),
    [ProductName] VARCHAR(255) NOT NULL,
    [Price] DECIMAL(8,2),
    [CreatedOn] DATETIME DEFAULT GETDATE()
);
```

### 2. Paste the code into your Fabric Lakehouse Notebook
Use the provided converter script inside a PySpark notebook cell. It includes:

- A full SQL Server → Spark data type mapping
- A function to detect unmapped types
- A function to convert SQL Server DDLs to Spark SQL

### 3. Example usage

```sql
sql_server_ddl = """
-- Paste your full CREATE TABLE statements here
"""

# Optional: warn about unknown SQL Server types
detect_unmapped_sqlserver_types(sql_server_ddl, SQLSERVER_TO_SPARK_TYPE_MAP)

# Convert and print the Spark SQL
converted_ddl_list = convert_sqlserver_to_spark(sql_server_ddl, strip_schema=True)
for ddl in converted_ddl_list:
    print(ddl)
```

### 4. Output Example

```sql
CREATE TABLE Sales (
  `EnrollmentId` INT NOT NULL,
  `CustomerName` STRING NOT NULL,
  `Amount` DECIMAL(10,2),
  `CreatedDate` TIMESTAMP,
  `IsActive` BOOLEAN
);
```

### 5. Copy and use the generated SQL
You can run the generated Spark SQL manually in the Fabric Lakehouse Notebook to create the table.

**Configuration**

|Option|Description|
|---|---|
|strip_schema|If True, removes schema prefixes like [dbo]|
|SQLSERVER_TO_SPARK_TYPE_MAP|Easily customizable dictionary to map SQL Server types to Spark types|

**Known Limitations**

- Does not preserve foreign keys, check constraints, or indexes
- Assumes standard formatting of SQL Server DDL (one table per CREATE TABLE)
- Inline COMMENT syntax is simulated using -- at the end of the column line
- ⚠️ Depending on the structure and complexity of the original DDL, the conversion may not always work perfectly. Please review and adjust the output as needed for your specific use case.