# SQL style guide

## Overview

This is a general style guide for writing SQL queries for Yelp's SQL Vault. This
SQL repository is maintained by the Data Science team. This guide is an attempt
help standardized any shared SQL code that produced and reused across Yelp's data
teams.

While this style guide may deviate from SQL styles that you are used to or that you
may prefer, we ask that you please adhere to this guide whenever possible (within 
reason). We believe it is more important that have a standard, shared style than
to find one that everyone likes (which is probably impossible anyway).

Original SQL style guide by Simon Holywell is licensed under a Creative Commons
Attribution-ShareAlike 4.0 International License. Based on a work at 
http://www.sqlstyle.guide.
## General

### Do

* Use consistent and descriptive identifiers and names.
* Make judicious use of white space and indentation to make code easier to read.
* Store [ISO-8601][iso-8601] compliant time and date information
  (`YYYY-MM-DD HH:MM:SS.SSSSS`).
* Try to use only standard SQL functions instead of vendor specific functions for
  reasons of portability.
* Keep code succinct and devoid of redundant SQL—such as unnecessary quoting or
  parentheses or `WHERE` clauses that can otherwise be derived.
* Include comments in SQL code where necessary. Use the C style opening `/*` and
  closing `*/` where possible otherwise precede comments with `--` and finish
  them with a new line.

```sql
SELECT file_hash  -- stored ssdeep hash
  FROM file_system
 WHERE file_name = '.vimrc';
```
```sql
/* Updating the file record after writing to the file */
UPDATE file_system
   SET file_modified_date = '1980-02-22 13:19:01.00000',
       file_size = 209732
 WHERE file_name = '.vimrc';
```

### Avoid

* CamelCase—it is difficult to scan quickly.
* Descriptive prefixes or Hungarian notation such as `sp_` or `tbl`.
* Plurals—use the more natural collective term where possible instead. For example
  `staff` instead of `employees` or `people` instead of `individuals`.
* Quoted identifiers—if you must use them then stick to SQL92 double quotes for
  portability (you may need to configure your SQL server to support this depending
  on vendor).
* Object oriented design principles should not be applied to SQL or database
  structures.

## Naming conventions

### General

* Ensure the name is unique and does not exist as a
  [reserved keyword][reserved-keywords].
* Keep the length to a maximum of 30 bytes—in practice this is 30 characters
  unless you are using multi-byte character set.
* Names must begin with a letter and may not end with an underscore.
* Only use letters, numbers and underscores in names.
* Avoid the use of multiple consecutive underscores—these can be hard to read.
* Use underscores where you would naturally include a space in the name (first
  name becomes `first_name`).
* Avoid abbreviations and if you have to use them make sure they are commonly
  understood.

```sql
SELECT first_name
  FROM staff;
```

### Tables

* Use a collective name or, less ideally, a plural form. For example (in order of
  preference) `staff` and `employees`.
* Do not prefix with `tbl` or any other such descriptive prefix or Hungarian
  notation.
* Never give a table the same name as one of its columns and vice versa.
* Avoid, where possible, concatenating two table names together to create the name
  of a relationship table. Rather than `cars_mechanics` prefer `services`.

### Columns

* Always use the singular name.
* Where possible avoid simply using `id` as the primary identifier for the table.
* Do not add a column with the same name as its table and vice versa.
* Always use lowercase except where it may make sense not to such as proper nouns.

### Aliasing or correlations

* Should relate in some way to the object or expression they are aliasing.
* If possible, avoid using an alias for shorter names and words. It is usually
  not necessary and often reduces readability.
* It is recommended to avoid using using correlation names as they reduce readability
  particularly when used in longer queries.
* Always include the `AS` keyword—makes it easier to read as it is explicit.
* Explicit use of table and column names should be used when possible to 
  disambiguate the source of columns. This also avoids "Ambiguous column name" errors.
* For computed data (`SUM()` or `AVG()`) use the name you would give it were it
  a column defined in the schema.

```sql
/* Good */
SELECT students.first_name
  FROM staff_registery AS staff
 INNER JOIN student_records AS students
    ON students.mentor_id = staff.staff_num;

/* Okay */
SELECT sr2.first_name AS fname
  FROM staff_registery AS sr1
 INNER JOIN student_records AS sr2
    ON sr2.mentor_id = sr1.staff_num;

/* Bad */
SELECT first_name fn
  FROM staff_registery s1
 INNER JOIN student_records s2
    ON s2.mentor_id = s1.staff_num;
```

```sql
/* Good */
SELECT SUM(staff.monitor_tally) AS monitor_total
  FROM staff_registery AS staff;

/* Bad */
SELECT SUM(s.monitor_tally) AS total
  FROM staff_registery AS s;
```

### Stored procedures

* The name must contain a verb.
* Do not prefix with `sp_` or any other such descriptive prefix or Hungarian
  notation.

### Uniform suffixes

The following suffixes have a universal meaning ensuring the columns can be read
and understood easily from SQL code. Use the correct suffix where appropriate.

* `_id`—a unique identifier such as a column that is a primary key.
* `_status`—flag value or some other status of any type such as
  `publication_status`.
* `_total`—the total or sum of a collection of values.
* `_num`—denotes the field contains any kind of number.
* `_name`—signifies a name such as `first_name`.
* `_seq`—contains a contiguous sequence of values.
* `_date`—denotes a column that contains the date of something.
* `_tally`—a count.
* `_size`—the size of something such as a file size or clothing.
* `_addr`—an address for the record could be physical or intangible such as
  `ip_addr`.

## Query syntax

### Reserved words

Always use uppercase for the [reserved keywords][reserved-keywords]
like `SELECT` and `WHERE`.

It is best to avoid the abbreviated keywords and use the full length ones where
available (prefer `ABSOLUTE` to `ABS`).

Do not use database server specific keywords where an ANSI SQL keyword already
exists performing the same function. This helps to make code more portable.

### White space

To make the code easier to read it is important that the correct complement of
spacing is used. Do not crowd code or remove natural language spaces.

#### Spaces

Spaces should be used to line up the code so that the root keywords all end on
the same character boundary. So for clauses that have multiple key words (e.g. LEFT JOIN,
GROUP BY) align the first word of the clause. This forms a river down the middle making 
it easy for the readers eye to scan over the code and separate the keywords from the
implementation detail. Rivers are [bad in typography][rivers], but helpful here.

```sql
/* Good */
(SELECT flora.species_name,
        AVG(flora.height) AS average_height,
        AVG(flora.diameter) AS average_diameter
   FROM flora
  WHERE flora.species_name = 'Banksia'
     OR flora.species_name = 'Sheoak'
     OR flora.species_name = 'Wattle'
  GROUP BY flora.species_name, flora.observation_date)

  UNION ALL

(SELECT garden_flora.species_name,
        AVG(garden_flora.height) AS average_height,
        AVG(garden_flora.diameter) AS average_diameter
   FROM botanic_garden_flora AS garden_flora
  WHERE garden_flora.species_name = 'Banksia'
     OR garden_flora.species_name = 'Sheoak'
     OR garden_flora.species_name = 'Wattle'
  GROUP BY garden_flora.species_name, garden_flora.observation_date);

/* Bad */
(SELECT 
    f.species_name,
    AVG(f.height) AS average_height, AVG(f.diameter) AS average_diameter
   FROM flora AS f
   WHERE f.species_name = 'Banksia'
     OR f.species_name = 'Sheoak'
     OR f.species_name = 'Wattle'
   GROUP BY f.species_name, f.observation_date)

UNION ALL

(SELECT 
    b.species_name,
    AVG(b.height) AS average_height, AVG(b.diameter) AS average_diameter
   FROM botanic_garden_flora AS b
   WHERE b.species_name = 'Banksia'
     OR b.species_name = 'Sheoak'
     OR b.species_name = 'Wattle'
   GROUP BY b.species_name, b.observation_date);
```

Notice that `SELECT`, `FROM`, etc. are all right aligned while the actual column
names and implementation specific details are left aligned.

Although not exhaustive always include spaces:

* before and after equals (`=`)
* after commas (`,`)
* surrounding apostrophes (`'`) where not within parentheses or with a trailing
  comma or semicolon.

```sql
SELECT albums.title, albums.release_date, albums.recording_date
  FROM albums
 WHERE albums.title = 'Charcoal Lane'
    OR albums.title = 'The New Danger';
```

#### Line spacing

Always include newlines/vertical space:

* before `AND` or `OR`
* after semicolons to separate queries for easier reading
* after each keyword definition
* after a comma when separating multiple columns into logical groups
* to separate code into related sections, which helps to ease the readability of
  large chunks of code.

Keeping all the keywords aligned to the righthand side and the values left aligned
creates a uniform gap down the middle of query. It makes it much easier to scan
the query definition over quickly too.

```sql
INSERT INTO albums (title, release_date, recording_date)
VALUES ('Charcoal Lane', '1990-01-01 01:01:01.00000', '1990-01-01 01:01:01.00000'),
       ('The New Danger', '2008-01-01 01:01:01.00000', '1990-01-01 01:01:01.00000');
```

```sql
UPDATE albums
   SET release_date = '1990-01-01 01:01:01.00000'
 WHERE title = 'The New Danger';
```

```sql
SELECT albums.title,
       albums.release_date, albums.recording_date, albums.production_date -- grouped dates together
  FROM albums
 WHERE albums.title = 'Charcoal Lane'
    OR albums.title = 'The New Danger';
```

### Indentation

To ensure that SQL is readable it is important that standards of indentation
are followed.

#### Joins

In most cases, joins should follow the perscribed spacing and key word alignment.
However, in some cases it may make sense to indent multiple joins to the righthand side
of the river. The join key words and any key words related to the join clause should
then follow the same 'river' spacing with root key words aligned to the join root keyword.

```sql
/* Good */
SELECT riders.last_name
  FROM riders

 INNER JOIN bikes
    ON riders.bike_vin_num = bikes.vin_num
   AND bikes.engine_tally > 2

  INNER JOIN crew
     ON riders.crew_chief_last_name = crew.last_name
    AND crew.chief = 'Y';

/* Good */
SELECT riders.last_name
  FROM riders
       INNER JOIN bikes
          ON riders.bike_vin_num = bikes.vin_num
         AND bikes.engine_tally > 2

       INNER JOIN crew
          ON riders.crew_chief_last_name = crew.last_name
         AND crew.chief = 'Y';

/* Bad */
SELECT riders.last_name
  FROM riders
       INNER JOIN bikes
       ON riders.bike_vin_num = bikes.vin_num
       AND bikes.engine_tally > 2

       INNER JOIN crew
       ON riders.crew_chief_last_name = crew.last_name
       AND crew.chief = 'Y';
```

#### Subqueries

Subqueries should also be aligned to the right side of the river and then laid
out using the same style as any other query. Sometimes it will make sense to have
the closing parenthesis on a new line at the same character position as its
opening partner—this is especially true where you have nested subqueries.

```sql
/* Good */
SELECT riders.last_name,
       (SELECT MAX(YEAR(championship_date))
          FROM champions
         WHERE champions.last_name = riders.last_name
           AND champions.confirmed = 'Y') AS last_championship_year
  FROM riders
 WHERE riders.last_name IN
       (SELECT champions.last_name
          FROM champions
         WHERE YEAR(championship_date) > '2008'
           AND champions.confirmed = 'Y');

/* bad */
SELECT riders.last_name,
(SELECT MAX(YEAR(championship_date))
   FROM champions
  WHERE champions.last_name = riders.last_name
    AND champions.confirmed = 'Y') AS last_championship_year
  FROM riders
 WHERE riders.last_name IN
(SELECT champions.last_name
   FROM champions
  WHERE YEAR(championship_date) > '2008'
    AND champions.confirmed = 'Y');
```

### Preferred formalisms

* Make use of `BETWEEN` where possible instead of combining multiple statements
  with `AND`.
* Similarly use `IN()` instead of multiple `OR` clauses.
* Where a value needs to be interpreted before leaving the database use the `CASE`
  expression. `CASE` statements can be nested to form more complex logical structures.
* Avoid the use of `UNION` clauses and temporary tables where possible. If the
  schema can be optimised to remove the reliance on these features then it most
  likely should be.

```sql
SELECT CASE postcode
       WHEN 'BN1' THEN 'Brighton'
       WHEN 'EH1' THEN 'Edinburgh'
        END AS city
  FROM office_locations
 WHERE country = 'United Kingdom'
   AND opening_time BETWEEN 8 AND 9
   AND postcode IN ('EH1', 'BN1', 'NN1', 'KW1');
```

## Create syntax

When declaring schema information it is also important to maintain human
readable code. To facilitate this ensure the column definitions are ordered and
grouped where it makes sense to do so.

Indent column definitions by four (4) spaces within the `CREATE` definition.

### Choosing data types

* Where possible do not use vendor specific data types—these are not portable and
  may not be available in older versions of the same vendor's software.
* Only use `REAL` or `FLOAT` types where it is strictly necessary for floating
  point mathematics otherwise prefer `NUMERIC` and `DECIMAL` at all times. Floating
  point rounding errors are a nuisance!

### Specifying default values

* The default value must be the same type as the column—if a column is declared
  a `DECIMAL` do not provide an `INTEGER` default value.
* Default values must follow the data type declaration and come before any
  `NOT NULL` statement.

### Constraints and keys

Constraints and their subset, keys, are a very important component of any
database definition. They can quickly become very difficult to read and reason
about though so it is important that a standard set of guidelines are followed.

#### Choosing keys

Deciding the column(s) that will form the keys in the definition should be a
carefully considered activity as it will effect performance and data integrity.

1. The key should be unique to some degree.
2. Consistency in terms of data type for the value across the schema and a lower
   likelihood of this changing in the future.
3. Can the value be validated against a standard format (such as one published by
   ISO)? Encouraging conformity to point 2.
4. Keeping the key as simple as possible whilst not being scared to use compound
   keys where necessary.

It is a reasoned and considered balancing act to be performed at the definition
of a database. Should requirements evolve in the future it is possible to make
changes to the definitions to keep them up to date.

#### Defining constraints

Once the keys are decided it is possible to define them in the system using
constraints along with field value validation.

##### General

* Tables must have at least one key to be complete and useful.
* Constraints should be given a custom name excepting `UNIQUE`, `PRIMARY KEY`
  and `FOREIGN KEY` where the database vendor will generally supply sufficiently
  intelligible names automatically.

##### Layout and order

* Specify the primary key first right after the `CREATE TABLE` statement.
* Constraints should be defined directly beneath the column they correspond to.
  Indent the constraint so that it aligns to the right of the column name.
* If it is a multi-column constraint then consider putting it as close to both
  column definitions as possible and where this is difficult as a last resort
  include them at the end of the `CREATE TABLE` definition.
* If it is a table level constraint that applies to the entire table then it
  should also appear at the end.
* Use alphabetical order where `ON DELETE` comes before `ON UPDATE`.
* If it make senses to do so align each aspect of the query on the same character
  position. For example all `NOT NULL` definitions could start at the same
  character position. This is not hard and fast, but it certainly makes the code
  much easier to scan and read.

##### Validation

* Use `LIKE` and `SIMILAR TO` constraints to ensure the integrity of strings
  where the format is known.
* Where the ultimate range of a numerical value is known it must be written as a
  range `CHECK()` to prevent incorrect values entering the database or the silent
  truncation of data too large to fit the column definition. In the least it
  should check that the value is greater than zero in most cases.
* `CHECK()` constraints should be kept in separate clauses to ease debugging.

##### Example

```sql
CREATE TABLE staff (
    PRIMARY KEY (staff_num),
    staff_num      INT(5)       NOT NULL,
    first_name     VARCHAR(100) NOT NULL,
    pens_in_drawer INT(2)       NOT NULL,
                   CONSTRAINT pens_in_drawer_range
                   CHECK(pens_in_drawer >= 1 AND pens_in_drawer < 100)
);
```

### Designs to avoid

* Object oriented design principles do not effectively translate to relational
  database designs—avoid this pitfall.
* Placing the value in one column and the units in another column. The column
  should make the units self evident to prevent the requirement to combine
  columns again later in the application. Use `CHECK()` to ensure valid data is
  inserted into the column.
* [EAV (Entity Attribute Value)][eav] tables—use a specialist product intended for
  handling such schema-less data instead.
* Splitting up data that should be in one table across many because of arbitrary
  concerns such as time-based archiving or location in a multi-national
  organisation. Later queries must then work across multiple tables with `UNION`
  rather than just simply querying one table.


## Appendix

### Reserved keyword reference

A list of ANSI SQL (92, 99 and 2003), MySQL 3 to 5.x, PostgreSQL 8.1, MS SQL Server 2000, MS ODBC and Oracle 10.2 reserved keywords.

```sql
A
ABORT
ABS
ABSOLUTE
ACCESS
ACTION
ADA
ADD
ADMIN
AFTER
AGGREGATE
ALIAS
ALL
ALLOCATE
ALSO
ALTER
ALWAYS
ANALYSE
ANALYZE
AND
ANY
ARE
ARRAY
AS
ASC
ASENSITIVE
ASSERTION
ASSIGNMENT
ASYMMETRIC
AT
ATOMIC
ATTRIBUTE
ATTRIBUTES
AUDIT
AUTHORIZATION
AUTO_INCREMENT
AVG
AVG_ROW_LENGTH
BACKUP
BACKWARD
BEFORE
BEGIN
BERNOULLI
BETWEEN
BIGINT
BINARY
BIT
BIT_LENGTH
BITVAR
BLOB
BOOL
BOOLEAN
BOTH
BREADTH
BREAK
BROWSE
BULK
BY
C
CACHE
CALL
CALLED
CARDINALITY
CASCADE
CASCADED
CASE
CAST
CATALOG
CATALOG_NAME
CEIL
CEILING
CHAIN
CHANGE
CHAR
CHAR_LENGTH
CHARACTER
CHARACTER_LENGTH
CHARACTER_SET_CATALOG
CHARACTER_SET_NAME
CHARACTER_SET_SCHEMA
CHARACTERISTICS
CHARACTERS
CHECK
CHECKED
CHECKPOINT
CHECKSUM
CLASS
CLASS_ORIGIN
CLOB
CLOSE
CLUSTER
CLUSTERED
COALESCE
COBOL
COLLATE
COLLATION
COLLATION_CATALOG
COLLATION_NAME
COLLATION_SCHEMA
COLLECT
COLUMN
COLUMN_NAME
COLUMNS
COMMAND_FUNCTION
COMMAND_FUNCTION_CODE
COMMENT
COMMIT
COMMITTED
COMPLETION
COMPRESS
COMPUTE
CONDITION
CONDITION_NUMBER
CONNECT
CONNECTION
CONNECTION_NAME
CONSTRAINT
CONSTRAINT_CATALOG
CONSTRAINT_NAME
CONSTRAINT_SCHEMA
CONSTRAINTS
CONSTRUCTOR
CONTAINS
CONTAINSTABLE
CONTINUE
CONVERSION
CONVERT
COPY
CORR
CORRESPONDING
COUNT
COVAR_POP
COVAR_SAMP
CREATE
CREATEDB
CREATEROLE
CREATEUSER
CROSS
CSV
CUBE
CUME_DIST
CURRENT
CURRENT_DATE
CURRENT_DEFAULT_TRANSFORM_GROUP
CURRENT_PATH
CURRENT_ROLE
CURRENT_TIME
CURRENT_TIMESTAMP
CURRENT_TRANSFORM_GROUP_FOR_TYPE
CURRENT_USER
CURSOR
CURSOR_NAME
CYCLE
DATA
DATABASE
DATABASES
DATE
DATETIME
DATETIME_INTERVAL_CODE
DATETIME_INTERVAL_PRECISION
DAY
DAY_HOUR
DAY_MICROSECOND
DAY_MINUTE
DAY_SECOND
DAYOFMONTH
DAYOFWEEK
DAYOFYEAR
DBCC
DEALLOCATE
DEC
DECIMAL
DECLARE
DEFAULT
DEFAULTS
DEFERRABLE
DEFERRED
DEFINED
DEFINER
DEGREE
DELAY_KEY_WRITE
DELAYED
DELETE
DELIMITER
DELIMITERS
DENSE_RANK
DENY
DEPTH
DEREF
DERIVED
DESC
DESCRIBE
DESCRIPTOR
DESTROY
DESTRUCTOR
DETERMINISTIC
DIAGNOSTICS
DICTIONARY
DISABLE
DISCONNECT
DISK
DISPATCH
DISTINCT
DISTINCTROW
DISTRIBUTED
DIV
DO
DOMAIN
DOUBLE
DROP
DUAL
DUMMY
DUMP
DYNAMIC
DYNAMIC_FUNCTION
DYNAMIC_FUNCTION_CODE
EACH
ELEMENT
ELSE
ELSEIF
ENABLE
ENCLOSED
ENCODING
ENCRYPTED
END
END-EXEC
ENUM
EQUALS
ERRLVL
ESCAPE
ESCAPED
EVERY
EXCEPT
EXCEPTION
EXCLUDE
EXCLUDING
EXCLUSIVE
EXEC
EXECUTE
EXISTING
EXISTS
EXIT
EXP
EXPLAIN
EXTERNAL
EXTRACT
FALSE
FETCH
FIELDS
FILE
FILLFACTOR
FILTER
FINAL
FIRST
FLOAT
FLOAT4
FLOAT8
FLOOR
FLUSH
FOLLOWING
FOR
FORCE
FOREIGN
FORTRAN
FORWARD
FOUND
FREE
FREETEXT
FREETEXTTABLE
FREEZE
FROM
FULL
FULLTEXT
FUNCTION
FUSION
G
GENERAL
GENERATED
GET
GLOBAL
GO
GOTO
GRANT
GRANTED
GRANTS
GREATEST
GROUP
GROUPING
HANDLER
HAVING
HEADER
HEAP
HIERARCHY
HIGH_PRIORITY
HOLD
HOLDLOCK
HOST
HOSTS
HOUR
HOUR_MICROSECOND
HOUR_MINUTE
HOUR_SECOND
IDENTIFIED
IDENTITY
IDENTITY_INSERT
IDENTITYCOL
IF
IGNORE
ILIKE
IMMEDIATE
IMMUTABLE
IMPLEMENTATION
IMPLICIT
IN
INCLUDE
INCLUDING
INCREMENT
INDEX
INDICATOR
INFILE
INFIX
INHERIT
INHERITS
INITIAL
INITIALIZE
INITIALLY
INNER
INOUT
INPUT
INSENSITIVE
INSERT
INSERT_ID
INSTANCE
INSTANTIABLE
INSTEAD
INT
INT1
INT2
INT3
INT4
INT8
INTEGER
INTERSECT
INTERSECTION
INTERVAL
INTO
INVOKER
IS
ISAM
ISNULL
ISOLATION
ITERATE
JOIN
K
KEY
KEY_MEMBER
KEY_TYPE
KEYS
KILL
LANCOMPILER
LANGUAGE
LARGE
LAST
LAST_INSERT_ID
LATERAL
LEADING
LEAST
LEAVE
LEFT
LENGTH
LESS
LEVEL
LIKE
LIMIT
LINENO
LINES
LISTEN
LN
LOAD
LOCAL
LOCALTIME
LOCALTIMESTAMP
LOCATION
LOCATOR
LOCK
LOGIN
LOGS
LONG
LONGBLOB
LONGTEXT
LOOP
LOW_PRIORITY
LOWER
M
MAP
MATCH
MATCHED
MAX
MAX_ROWS
MAXEXTENTS
MAXVALUE
MEDIUMBLOB
MEDIUMINT
MEDIUMTEXT
MEMBER
MERGE
MESSAGE_LENGTH
MESSAGE_OCTET_LENGTH
MESSAGE_TEXT
METHOD
MIDDLEINT
MIN
MIN_ROWS
MINUS
MINUTE
MINUTE_MICROSECOND
MINUTE_SECOND
MINVALUE
MLSLABEL
MOD
MODE
MODIFIES
MODIFY
MODULE
MONTH
MONTHNAME
MORE
MOVE
MULTISET
MUMPS
MYISAM
NAME
NAMES
NATIONAL
NATURAL
NCHAR
NCLOB
NESTING
NEW
NEXT
NO
NO_WRITE_TO_BINLOG
NOAUDIT
NOCHECK
NOCOMPRESS
NOCREATEDB
NOCREATEROLE
NOCREATEUSER
NOINHERIT
NOLOGIN
NONCLUSTERED
NONE
NORMALIZE
NORMALIZED
NOSUPERUSER
NOT
NOTHING
NOTIFY
NOTNULL
NOWAIT
NULL
NULLABLE
NULLIF
NULLS
NUMBER
NUMERIC
OBJECT
OCTET_LENGTH
OCTETS
OF
OFF
OFFLINE
OFFSET
OFFSETS
OIDS
OLD
ON
ONLINE
ONLY
OPEN
OPENDATASOURCE
OPENQUERY
OPENROWSET
OPENXML
OPERATION
OPERATOR
OPTIMIZE
OPTION
OPTIONALLY
OPTIONS
OR
ORDER
ORDERING
ORDINALITY
OTHERS
OUT
OUTER
OUTFILE
OUTPUT
OVER
OVERLAPS
OVERLAY
OVERRIDING
OWNER
PACK_KEYS
PAD
PARAMETER
PARAMETER_MODE
PARAMETER_NAME
PARAMETER_ORDINAL_POSITION
PARAMETER_SPECIFIC_CATALOG
PARAMETER_SPECIFIC_NAME
PARAMETER_SPECIFIC_SCHEMA
PARAMETERS
PARTIAL
PARTITION
PASCAL
PASSWORD
PATH
PCTFREE
PERCENT
PERCENT_RANK
PERCENTILE_CONT
PERCENTILE_DISC
PLACING
PLAN
PLI
POSITION
POSTFIX
POWER
PRECEDING
PRECISION
PREFIX
PREORDER
PREPARE
PREPARED
PRESERVE
PRIMARY
PRINT
PRIOR
PRIVILEGES
PROC
PROCEDURAL
PROCEDURE
PROCESS
PROCESSLIST
PUBLIC
PURGE
QUOTE
RAID0
RAISERROR
RANGE
RANK
RAW
READ
READS
READTEXT
REAL
RECHECK
RECONFIGURE
RECURSIVE
REF
REFERENCES
REFERENCING
REGEXP
REGR_AVGX
REGR_AVGY
REGR_COUNT
REGR_INTERCEPT
REGR_R2
REGR_SLOPE
REGR_SXX
REGR_SXY
REGR_SYY
REINDEX
RELATIVE
RELEASE
RELOAD
RENAME
REPEAT
REPEATABLE
REPLACE
REPLICATION
REQUIRE
RESET
RESIGNAL
RESOURCE
RESTART
RESTORE
RESTRICT
RESULT
RETURN
RETURNED_CARDINALITY
RETURNED_LENGTH
RETURNED_OCTET_LENGTH
RETURNED_SQLSTATE
RETURNS
REVOKE
RIGHT
RLIKE
ROLE
ROLLBACK
ROLLUP
ROUTINE
ROUTINE_CATALOG
ROUTINE_NAME
ROUTINE_SCHEMA
ROW
ROW_COUNT
ROW_NUMBER
ROWCOUNT
ROWGUIDCOL
ROWID
ROWNUM
ROWS
RULE
SAVE
SAVEPOINT
SCALE
SCHEMA
SCHEMA_NAME
SCHEMAS
SCOPE
SCOPE_CATALOG
SCOPE_NAME
SCOPE_SCHEMA
SCROLL
SEARCH
SECOND
SECOND_MICROSECOND
SECTION
SECURITY
SELECT
SELF
SENSITIVE
SEPARATOR
SEQUENCE
SERIALIZABLE
SERVER_NAME
SESSION
SESSION_USER
SET
SETOF
SETS
SETUSER
SHARE
SHOW
SHUTDOWN
SIGNAL
SIMILAR
SIMPLE
SIZE
SMALLINT
SOME
SONAME
SOURCE
SPACE
SPATIAL
SPECIFIC
SPECIFIC_NAME
SPECIFICTYPE
SQL
SQL_BIG_RESULT
SQL_BIG_SELECTS
SQL_BIG_TABLES
SQL_CALC_FOUND_ROWS
SQL_LOG_OFF
SQL_LOG_UPDATE
SQL_LOW_PRIORITY_UPDATES
SQL_SELECT_LIMIT
SQL_SMALL_RESULT
SQL_WARNINGS
SQLCA
SQLCODE
SQLERROR
SQLEXCEPTION
SQLSTATE
SQLWARNING
SQRT
SSL
STABLE
START
STARTING
STATE
STATEMENT
STATIC
STATISTICS
STATUS
STDDEV_POP
STDDEV_SAMP
STDIN
STDOUT
STORAGE
STRAIGHT_JOIN
STRICT
STRING
STRUCTURE
STYLE
SUBCLASS_ORIGIN
SUBLIST
SUBMULTISET
SUBSTRING
SUCCESSFUL
SUM
SUPERUSER
SYMMETRIC
SYNONYM
SYSDATE
SYSID
SYSTEM
SYSTEM_USER
TABLE
TABLE_NAME
TABLES
TABLESAMPLE
TABLESPACE
TEMP
TEMPLATE
TEMPORARY
TERMINATE
TERMINATED
TEXT
TEXTSIZE
THAN
THEN
TIES
TIME
TIMESTAMP
TIMEZONE_HOUR
TIMEZONE_MINUTE
TINYBLOB
TINYINT
TINYTEXT
TO
TOAST
TOP
TOP_LEVEL_COUNT
TRAILING
TRAN
TRANSACTION
TRANSACTION_ACTIVE
TRANSACTIONS_COMMITTED
TRANSACTIONS_ROLLED_BACK
TRANSFORM
TRANSFORMS
TRANSLATE
TRANSLATION
TREAT
TRIGGER
TRIGGER_CATALOG
TRIGGER_NAME
TRIGGER_SCHEMA
TRIM
TRUE
TRUNCATE
TRUSTED
TSEQUAL
TYPE
UESCAPE
UID
UNBOUNDED
UNCOMMITTED
UNDER
UNDO
UNENCRYPTED
UNION
UNIQUE
UNKNOWN
UNLISTEN
UNLOCK
UNNAMED
UNNEST
UNSIGNED
UNTIL
UPDATE
UPDATETEXT
UPPER
USAGE
USE
USER
USER_DEFINED_TYPE_CATALOG
USER_DEFINED_TYPE_CODE
USER_DEFINED_TYPE_NAME
USER_DEFINED_TYPE_SCHEMA
USING
UTC_DATE
UTC_TIME
UTC_TIMESTAMP
VACUUM
VALID
VALIDATE
VALIDATOR
VALUE
VALUES
VAR_POP
VAR_SAMP
VARBINARY
VARCHAR
VARCHAR2
VARCHARACTER
VARIABLE
VARIABLES
VARYING
VERBOSE
VIEW
VOLATILE
WAITFOR
WHEN
WHENEVER
WHERE
WHILE
WIDTH_BUCKET
WINDOW
WITH
WITHIN
WITHOUT
WORK
WRITE
WRITETEXT
X509
XOR
YEAR
YEAR_MONTH
ZEROFILL
ZONE
```

### Column data types

These are some suggested column data types to use for maximum compatibility between database engines.

#### Character types:

* CHAR
* CLOB
* VARCHAR

#### Numeric types

* Exact numeric types
    * BIGINT
    * DECIMAL
    * DECFLOAT
    * INTEGER
    * NUMERIC
    * SMALLINT
* Approximate numeric types
    * DOUBLE PRECISION
    * FLOAT
    * REAL

#### Datetime types

* DATE
* TIME
* TIMESTAMP

#### Binary types:

* BINARY
* BLOB
* VARBINARY

#### Additional types

* Boolean
* INTERVAL
* XML


[simon]: https://www.simonholywell.com/?utm_source=sqlstyle.guide&utm_medium=link&utm_campaign=md-document
    "SimonHolywell.com"
[issue]: https://github.com/treffynnon/sqlstyle.guide/issues
    "SQL style guide issues on GitHub"
[fork]: https://github.com/treffynnon/sqlstyle.guide/fork
    "Fork SQL style guide on GitHub"
[pull]: https://github.com/treffynnon/sqlstyle.guide/pulls/
    "SQL style guide pull requests on GitHub"
[celko]: https://www.amazon.com/gp/product/0120887975/ref=as_li_ss_tl?ie=UTF8&linkCode=ll1&tag=treffynnon-20&linkId=9c88eac8cd420e979675c815771313d5
    "Joe Celko's SQL Programming Style (The Morgan Kaufmann Series in Data Management Systems)"
[dl-md]: https://raw.githubusercontent.com/treffynnon/sqlstyle.guide/gh-pages/_includes/sqlstyle.guide.md
    "Download the guide in Markdown format"
[iso-8601]: https://en.wikipedia.org/wiki/ISO_8601
    "Wikipedia: ISO 8601"
[rivers]: http://practicaltypography.com/one-space-between-sentences.html
    "Practical Typography: one space between sentences"
[reserved-keywords]: #reserved-keyword-reference
    "Reserved keyword reference"
[eav]: https://en.wikipedia.org/wiki/Entity%E2%80%93attribute%E2%80%93value_model
    "Wikipedia: Entity–attribute–value model"
[sqlstyleguide]: http://www.sqlstyle.guide
    "SQL style guide by Simon Holywell"
[licence]: http://creativecommons.org/licenses/by-sa/4.0/
    "Creative Commons Attribution-ShareAlike 4.0 International License"
