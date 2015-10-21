# Introduction

I have compiled these notes whilst revising for the Oracle [1Z0-061](https://education.oracle.com/pls/web_prod-plq-dad/db_pages.getpage?page_id=5001&get_params=p_exam_id:1Z0-061) Exam - Oracle Database 12c: SQL Fundamentals. They should also be relevant to the [1Z0-051](https://education.oracle.com/pls/web_prod-plq-dad/db_pages.getpage?page_id=5001&get_params=p_exam_id:1Z0-051) - Oracle Database 11g: SQL Fundamentals exam. Revision was most conducted using the excellent and highly recommended "[OCA Oracle Database 12c SQL Fundamentals I Exam Guide](http://www.amazon.com/Oracle-Database-Fundamentals-Guide-1Z0-061/dp/0071820280)" by Roopesh Ramklass.

I have aimed to include include in these notes common "gotchas" and easy to forget functionality rather than documenting everything required for the exam. This can then be used as a quick refresher before walking into the exam.

The content is broken up into sections with each heading mapping to the relevant [Oracle 1Z0-061 exam topics](https://education.oracle.com/pls/web_prod-plq-dad/db_pages.getpage?page_id=5001&get_params=p_exam_id:1Z0-061).

# Describe the features of Oracle Database 12c

* The DML (data manipulation language) commands are: `SELECT`, `INSERT`, `UPDATE`, `DELETE` and `MERGE`.
* The DDL (data definition language) commands are: `CREATE`, `ALTER`, `DROP`, `RENAME`, `TRUNCATE` and `COMMENT`.
* The DCL (data control language) command are: `GRANT` and `REVOKE`.
* The TXL (transaction control language) commands are: `COMMIT`, `ROLLBACK` and `SAVEPOINT`.

# Retrieving Data using the SQL SELECT Statement

* Concatenation with NULL is OK.
    ```sql
  'Mike||NULL||'Leonard' = 'MikeLeonard'
    ```
* Expressions with NULL always result in NULL.
  ```sql
  1 + 2 * NULL + 3 = NULL
  ```

# Restricting and Sorting Data.

* `BETWEEN` is inclusive.

# Using Single-Row Functions to Customize Output

* `TRIM` by default trims whitespace.
* To `TRIM` other characters use the following syntax.
  ```sql
  TRIM('#' from '#MYSTRING#')
  ```
* `LEADING`, `TRAILING` or `BOTH` can be specified in `TRIM` to control where the characters are trimmed from.
  ```sql
  TRIM(TRAILING '#' from '#MYSTRING#')
  TRIM(LEADING '#' from '#MYSTRING#')
  TRIM(BOTH '#' from '#MYSTRING#') -- This is the default.
  ```
* `MONTHS_BETWEEN` works backwards, that is a positive number is returned when the first argument is greater than the second.
  ```sql
  MONTHS_BETWEEN('01-JAN-15', '01-FEB-15') = -1
  MONTHS_BETWEEN('01-FEB-15', '01-JAN-15') = 1
  ```
* If `INSTR` does not find the target string 0 is returned.
  ```sql
  INSTR('a', 'b') = 0
  ```
* You can `ROUND` to nearest whole numbers (least significant digit is 0).
  ```sql
  ROUND(1584.73, -3) = 2000
  ROUND(11, -1) = 10
  ```
* `LPAD`/`RPAD` take an argument specifying the resultant length **not** how much to append:.
  ```sql
  LPAD('A', 4, '.') = '...A'
  RPAD('A', 4, '.') = 'A...'
  ```
* `NULLIF` returns the first argument if the two arguments dont match else its returns `NULL`.
  ```sql
  NULLIF('a', 'a') = NULL
  NULLIF('a', 'b') = 'a'
  ```

# Using Conversion Functions and Conditional Expressions

* Format masks behave differently when operating on numbers or characters.
  ```sql
  TO_NUMBER(1234.49, 999999.9) -- Raises an exception ORA_01722: invalid number
  TO_CHAR(1234.49, '999999.9') = 1234.5 -- Note the rounding
  ```
* The default Oracle data format mask is `DD-MON-RR`.

## TODO: FORMAT MASKS!!!!


# Reporting Aggregated Data Using the Group Functions

* `COUNT(ALL *)` is default and the same as `COUNT(*)`.
* Group functions ignore NULLs.
* Group functions can only be nested two levels deep.
* `HAVING` can come before or after the `GROUP BY`.

# Displaying Data From Multiple Tables Using Joins

* `NATURAL JOIN` joins tables using columns with identical names.
* A `NATURAL JOIN` becomes a cartesian (cross) join when no matching column names are found.
* `NATURAL JOIN` syntax:
  ```sql
  SELECT *
  FROM emp
  NATURAL JOIN dept
  ```
* Oracle join syntax:
    * INNER JOIN:
      ```sql
      SELECT *
      FROM emp, dept
      WHERE emp.deptno = dept.deptno
      ```
    * LEFT OUTER JOIN:
      ```sql
      SELECT *
      FROM emp, dept
      WHERE emp.deptno = dept.deptno (+)
      ```
    * RIGHT OUTER JOIN:
      ```sql
      SELECT *
      FROM emp, dept
      WHERE emp.deptno (+) = dept.deptno
      ```
* `USING` syntax
  ```sql
  SELECT *
  FROM emp
  JOIN dept USING(deptno)
  ```
* Queries with the `USING` syntax cannot alias the column(s) used in the `USING(...)` clause.
  ```sql
  SELECT d.deptno
  FROM emp e
  JOIN dept d USING(deptno)

  -- Results in ORA-25154: column part of USING clause cannot have qualifier.
  ```
* `USING`, `NATURAL JOIN` and `ON` are mutually exclusive and these JOIN types cannot be mixed.

# Using Subqueries to Solve Queries

## Subqueries
* Subqueries can be nested an unlimited depth in a `FROM`.
* Subqueries can "only" be nested 255 levels deep in a `WHERE`.
* Subqueries cannot be used in a `GROUP BY` or `ORDER BY`.

## Use a set operator to combine multiple queries into a single query
* `UNION`, `MINUS` and `INTERSECT` all remove duplicates and order the results. `UNION ALL` does neither of these.
* `ORDER BY` can only be used at the end of a compound (UNION, MINUS, INTERSECT) query and not in each individual part.

# Managing Tables using DML statements

* Oracle is **ACID** compliant:
  * **A** tomicity: All or nothing.
  * **C** onsistency: Within a given statement the data manipulated is from the same starting point and not modified part way through.
  * **I** solated: Until committed, changed data cannot be seen by others.
  * **D** urable: Once committed the changes are never lost.
* If an error occurs during a statement the work of the statement is undone but the work of all other statements in the same transactions remain but uncommitted.
* Whilst sometimes categorized as DML, `TRUNCATE` is DDL and cannot be rolled back.

## Insert rows into a table
* The `VALUES` keyword is not used in an `INSERT ... AS SELECT ...` statement.

## Control transactions
* Transactions are started implicitly with DML.
* Transactions ended with `COMMIT` or `ROLLBACK`.
* `COMMIT` is fast, `ROLLBACK` is slow (can possibly take longer to ROLLBACK that it originally did to do the work).
* Create a `SAVEPOINT` with `SAVEPOINT name;`.
* `ROLLBACK` a `SAVEPOINT` with `ROLLBACK TO SAVEPOINT name;`.

# Introduction to Data Definition Language

* Object names...
  * must be no longer than 30 characters;
  * must start with a letter (A-Z);
  * must include only A-Z, 0-9, \_, $ or #.
  * must be upper case (even if entered lower case they will be converted to upper)''
  * may include additional characters and be lower case if enclosed with quotes ("). However, once this is done the object must always be referred to using quotes.
* Objects names are case sensitive.
  ```sql
  CREATE TABLE test1 (
    ...
  );

  CREATE TABLE "test1" (
    ...
  );

  -- Results in two tables, one called TEST1 and another called test1.
  ```
* Object names (schema.name) must be unique with their namespace.
* Indexes and constraints have their own namespace so they can share a name with tables, views, sequences and private synonyms even within the same schema.
* DDL will fail if there is another active transaction against the object being altered.
* It is impossible to `DROP` a table if it is the subject of a `FOREIGN KEY` from another table.
* Oracle 12c includes a recycle bin that is enabled by default. Dropped objects can be recovered from here as long as they haven't been dropped with the `PURGE` option.

## Describe the data types that are available for columns
* The following datatypes are important to know for the exam:
  * `VARCHAR2` Variable-length character data from 1 byte to 4000 bytes if `MAX_STRING_SIZE=STANDARD` or 32767 bytes in `MAX_STRING_SIZE=EXTENDED`. The database is stored in the database character set.
  * `CHAR` Fixed-length character data, from 1 byte to 2000 bytes, in the database character set. If the data is not the length of the column then it will be padded with spaces.
  * `NUMBER` Numeric data, for which you can specify precision and scale. The precision can range from 1 to 38, the scale can range from -84 to 127.
  * `DATE` The is either the length zero, if the column is empty or 7 bytes. All `DATE` data includes century, year, month, day, hour, minute and second.
  * `TIMESTAMP` This is length zero if the column is empty, or up to 11 bytes, depending on the precision specified. Similar to `DATE` but with precision of up to 9 decimal places for the seconds, 6 places by default.
  * `TIMESTAMP WITH TIMEZONE` Like `TIMESTAMP` but the data is stored with a record kept of the time zone to which it refers. The length may be up to 13 bytes, depending on precision. This data type lets Oracle determine the difference between two time by normalizing them to UTA, even if the times are for different time zones.
  * `TIMESTAMP WITH LOCAL TIMEZONE` Like `TIMESTAMP`, but the data is normalize to the database time zone on saving. When retrieved, it is normalized to the time zone of the user processing it.
  * `INTERVAL YEAR TO MONTH` Used for recording a period in years and months between two `DATE`s or `TIMESTAMP`s.
  * `INTERVAL DAY TO SECONDE` Used for recording a period in days and seconds between two `DATE`s or `TIMESTAMP`s.
  * `RAW` Variable-length binary data, from 1 byte to 4000 bytes if `MAX_STRING_SIZE=STANDARD` or 32767 bytes if `MAX_STRING_SIZE=EXTENDED`. Unlike the `CHAR` and `VARCHAR2` data types, `RAW` data is not converted by Oracle Net from the databases character set to the user process's character set on `SELECT` or the other way on `INSERT`.
  * `LONG` Character data in the database character set, up to 2gb. All the functionality of `LONG` is provided by `CLOB`; `LONG`s should not be used in a modern database, and if your database has any columns of this type they should be converted to `CLOB`. There can only be one `LONG` column in a table.
  * `LONG RAW` Like `LONG`, but binary data that will not be converted by Oracle Net. Any `LONG RAW` columns should be converted to `BLOB`s.
  * `CLOB` Character data stored in the database character set, size effectively unlimited: (4gb - 1) multiplied by the database block size.
  * `BLOB` Like `CLOB` but binary data that will not undergo character set conversion by Oracle Net.
  * `BFILE` A locator pointing to a file stored on the operating system of the database server. The size of the files is limited to 4gb.
  * `ROWID` A value coded in base64 that is the pointer to the location of a row in a table. Encrypted within it is the exact physical address. `ROWID` is an Oracle proprietary data type, not visible unless specifically selected.
* `VARCHAR2`, `NUMBER` and `DATE` required a detailed understanding.
* `NUMBER` with a negative scale will round:
  ```sql
  CREATE TABLE numtest (
    id NUMBER(12, -4)
  );

  INSERT INTO numtest VALUES (12);
  INSERT INTO numtest VALUES (12345);
  INSERT INTO numtest VALUES (56789);
  INSERT INTO numtest VALUES (99999);

  SELECT *
  FROM numtest;

  --          ID
  -- -----------
  --           0
  --       10000
  --       60000
  --      100000
  ```

## Create a simple table
* `CREATE TABLE ... AS SELECT ...` copies a tables structure including `NOT NULL` and `CHECK` constraints. `PRIMARY KEY`, `UNIQUE` and `FOREIGN KEY`s are not copied.
* Various `ALTER TABLE` options
  * Adding columns
    ```sql
    ALTER TABLE emp
    ADD (job_id NUMBER);
    ```
  * Adding columns
    ```sql
    ALTER TABLE emp
    MODIFY (comm NUMBER(4,2) DEFAULT 0.05);
    ```
  * Dropping columns
    ```sql
    ALTER TABLE emp
    DROP COLUMN comm;
    ```
  * Marking columns as unused
    ```sql
    ALTER TABLE emp
    SET UNUSED COLUMN job_id;
    ```
  * Renaming columns
    ```sql
    ALTER TABLE emp
    RENAME COLUMN hiredate TO recruited
    ```
  * Marking the table as read-only
    ```sql
    ALTER TABLE emp
    READ ONLY;
    ```

## Explain how constraints are created at the time of table creation
* `UNIQUE` constraints ignore `NULL` values.
* `PRIMARY KEY` is a combination of `UNIQUE` and `NOT NULL`.
* `FOREING KEY` constraints must reference columns of a `UNIQUE` or `PRIMARY KEY` constraint in the referenced table.
* `DELETE`ing rows in a `FOREIGN KEY` referenced table is not allowed unless the constraint is specified with one of the following:
  * `ON DELETE CASCADE` Also delete the rows referencing the row to be deleted.
  * `ON DELETE SET NULL` Find any rows referencing the row to be deleted and make `NULL` the columns in the `FOREIGN KEY`.
