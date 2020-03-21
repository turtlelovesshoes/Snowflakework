

## combine the app version and the student name
'''
USE warehouse "TEST_RACHEL_TEMP_EMPLOYEE";
USE database temp_db;

Create or replace table App_versions(data variant); 

CREATE or Replace FILE FORMAT r_json
   TYPE = "JSON"
   COMPRESSION = "GZIP"
   FILE_EXTENSION= 'json.gz'
   STRIP_OUTER_ARRAY = TRUE
   
Create or replace file format name
 file_format = (type = 'csv')
 field_delimiter = '~' 
 FIELD_OPTIONALLY_ENCLOSED_BY= '"' 
 EMPTY_FIELD_AS_NULL = TRUE 
 NULL_IF="" 
 ESCAPE_UNENCLOSED_FIELD = None

COPY INTO App_versions
   FROM @~/staged 
   file_format=(TYPE = 'csv')
   on_error = 'skip_file';

COPY INTO App_versions
  FROM (SELECT $1
          FROM @~/staged)
  FILE_FORMAT = (TYPE = 'csv')
;
   
   
 list @~;
 
 SELECT * FROM App_versions limit 10;
 
Select data[0]:available::boolean as avail, data[0]:color::string as col, data[0]:name::string as title, data[0]:version::string as version from App_versions;

Select data:available::boolean as avail, data:color::string as col, data:name::string as title, data:version::string as version from App_versions;

CREATE TEMPORARY TABLE App_versions (Avail boolean, Color varchar); 
create view App_version_view (x, x_times_2) as select data:available::boolean as avail, data:color::string as col from App_versions;
REMOVE TABLE App_Versions;


'''
Xxx
## try constraints on a address book and recipts
USE warehouse "TEST_RACHEL_TEMP_EMPLOYEE";
USE database temp_db;



create or replace table name_key (
    id integer not null,
    id_sub integer not null,
    constraint pkey_1 primary key (id, id_sub) not enforced,
    name varchar
    );
create or replace table recipts (
    col_a integer not null,
    col_b integer not null,
    constraint fkey_1 foreign key (col_a, col_b) references name_key (id, id_sub) not enforced,
    recipt_date datetime
    );

Insert into name_key values (0, 0, 'Geinie'), (1, 1, 'Greg'), (2,2, 'Alex'), (3,3, 'Willow'); 
Insert into recipts values(0,0, Current_date()), (1,1, Current_date()), (2,2, Current_date()), (3,3, Current_date());

Select * from name_key;
Select * from recipts;

Select * from name_key
join recipts on name_key.id = recipts.col_a
where id = 0 or col_b = 2;
xx
USE warehouse "TEST_RACHEL_TEMP_EMPLOYEE";
USE database temp_db;

list @~;
copy into mytable from @~/staged file_format = (format_name = 'my_csv_format');

SELECT * FROM staged/array-of-things-locations-1.csv.gz


CREATE OR REPLACE TABLE ARRAYLOC(name varchar, location_type varchar, category varchar, notes varchar, status1 varchar, latitude float, longitude varchar, location varchar);

COPY INTO ARRAYLOC
   FROM @staged/array-of-things-locations-1.csv.gz;
 
 CREATE or Replace FILE FORMAT r_csv
   TYPE = "CSV"
   COMPRESSION = "GZIP"
   FILE_EXTENSION= 'csv.gz'
   SKIP_HEADER = 1
   ERROR_ON_COLUMN_COUNT_MISMATCH=FALSE
   EMPTY_FIELD_AS_NULL = TRUE;
 
create or replace stage staged
    file_format='r_csv';
    
copy into ARRAYLOC from @~/staged 
   file_format = (format_name = 'r_csv');
   
SELECT * FROM ARRAYLOC LIMIT 10;
xx
SE warehouse "TEST_RACHEL_TEMP_EMPLOYEE";
USE database temp_db;

SELECT * FROM "TEMP_DB"."PUBLIC"."TEMP_HISTORY"
Limit 10;

SELECT Query_id, query_text, start_time, EXECUTION_STATUS FROM "TEMP_DB"."PUBLIC"."TEMP_HISTORY";

CREATE SECURE View Student_queries_view
   comment='this' 
AS 
SELECT Query_id, query_text, start_time, EXECUTION_STATUS FROM "TEMP_DB"."PUBLIC"."TEMP_HISTORY";


CREATE OR REPLACE PROCEDURE Sorting()
   Returns string
   Language javascript
   as
   $$
    var rs = snowflake.execute( { sqlText: 
      'SELECT Query_id, query_text, start_time, execution_status FROM "TEMP_DB"."PUBLIC"."TEMP_HISTORY" Order by execution_status;'
       } );
  return rs;   
   $$;

 SELECT * FROM Student_queries_view;
Call Sorting();

####XX
USE WAREHOUSE TEMB_WH;
SHOW TASKS where state = ""  ;

#trying to create a table copy of the query history for longer rententions
Create or Replace TABLE TEMP_history(Query_id varchar, query_text varchar, database_name varchar, schema_name varchar, query_type varchar, Username varchar, role_name varchar, warehouse_name varchar, execution_status varchar, start_time timestamp_ltz, End_time timestamp_ltz); 
INSERT INTO TEMP_history(select Query_id, query_text, database_name, schema_name, query_type, user_name, role_name, warehouse_name, execution_status, start_time, End_time from table("TEMP_DB"."INFORMATION_SCHEMA".query_history())
order by start_time);

CREATE OR Replace TASK test_histor
   Warehouse = 'TEST_RACHEL_EMPLOYEE'
   Schedule = '58 Minutes'
AS
INSERT INTO TEMP_history(select Query_id, query_text, database_name, schema_name, query_type, Username, role_name, warehouse_name, execution_status, start_time, End_time from table("TEMP_DB"."INFORMATION_SCHEMA".query_history())
order by start_time);

select * from table("TEMP_DB"."INFORMATION_SCHEMA".query_history())
order by start_time;

###

USE warehouse "TEST_RACHEL_TEMP_EMPLOYEE";
USE database temp_db;

create table json_date(result variant, date datetime);

ALTER TABLE json_date ALTER COLUMN date Where date >"12-01-2019" and date < "12-30-2019" dateadd(day,2,date);

##

//https://stackoverflow.com/questions/59399646/how-can-i-share-a-clustered-materialized-view based on case
USE WAREHOUSE "TEST_RACHEL_TEMP_EMPLOYEE";
use database "TEST";
//create schema test_schema;
//grant select on all tables in schema Time_travel.test_schema to sysadmin;
Create or Replace Table ArticleLibrary (id int, Title string, Genre string, Viewed number, date_captured timestamp );
INSERT INTO ArticleLibrary VALUES
   (1,  'The man who walked the moon',   'Non-Fiction',  10, CURRENT_DATE() ),
   (2,  'The girl who went to Vegas',    'Fiction',      20 , CURRENT_DATE());
   
INSERT INTO ArticleLibrary VALUES
   (3,  'Harry Potter and the Chamber of Secrets',   'Fiction',  10, CURRENT_DATE() ),
   (4,  'DEFENCE AGAINST THE DARK ARTS; & DARK ARTS',    'Fiction',      20 , CURRENT_DATE());
   
Drop table ArticleLibrary;
undrop table ArticleLibrary; 

CREATE MATERIALIZED VIEW Article_Library_view_date
    COMMENT='Test view'
    AS
    SELECT ID, TITLE, GENRE, Date_captured FROM ArticleLibrary;


alter materialized view Article_Library_view_date cluster by(to_date(date_captured),genre);

show materialized views;

SELECT * FROM Article_Library_view_date cluster;

select *
from table(information_schema.query_history()) where query_text like '%ArticleLibrary%'
order by start_time;

##

 set x = 1;
set x = $x + 1;
select $x;
Show variables like '%x';
xx
USE warehouse "TEST_RACHEL_TEMP_EMPLOYEE";
USE database temp_db;

CREATE OR REPLACE TABLE Swap1(x int, two int); 
INSERT INTO Swap1 values(1, 3);
INSERT INTO Swap1 values(3, 2);


SELECT * FROM Swap1;

CREATE TABLE Swap2(three int, four int);
Insert INTO Swap2 Values(4, 5);
Insert INTO Swap2 Values( 5, 2);

SELECT * FROM Swap1;
SELECT * FROM Swap2;

ALTER TABLE Swap1 SWAP WITH Swap2;
Xxx not same
// for https://community.snowflake.com/s/question/0D50Z00009f85W7/how-to-get-rid-of-nanoseconds-from-currenttimestamp
//https://stackoverflow.com/questions/58550367/perform-scd2-on-snowflake-table-based-upon-oracle-input-data/58563432#58563432
USE warehouse "TEST_RACHEL_TEMP_EMPLOYEE";
USE database temp_db;

CREATE TABLE snowflake_a(key_col1 varchar(10), key_col2 varchar(10), key_col3 varchar(10), attr1 varchar(8), attr2 varchar(10), eff_ts TIMESTAMP, exp_ts TIMESTAMP, valid varchar(10)); 

DROP table oracle_a;
INSERT INTO snowflake_a VALUES('PT_1', 'DL_1', 'RPT_1', 'Address1', 'APT_1', current_date, current_date, 'Active');


CREATE TABLE oracle_a(key_col1 varchar(10), key_col2 varchar(10), key_col3 varchar(10), attr1 varchar(8), attr2 varchar(8), eff_ts TIMESTAMP, exp_ts TIMESTAMP); 

INSERT INTO oracle_a
VALUES( 'PT_1', 'DL_1', 'RPT_1', 'Address1', 'APT_1', '10/24/2019', '12/31/1999');


update snowflake_a
   set valid= 'Expired'
where valid LIKE '%Active%';

SELECT * FROM snowflake_a;

INSERT INTO snowflake_a VALUES( 'PT_1', 'DL_1', 'RPT_1', 'Address1', 'APT_1', '10/24/2019', '12/31/1999', 'Active');

SELECT * FROM snowflake_a;


select snowflake_a;

## Working with Semi-Structured Data;
//troubleshooting https://stackoverflow.com/questions/56330753/how-do-i-flatten-a-nested-json-key-value-pair-into-a-single-array-of-values

USE warehouse "TEST_RACHEL_TEMP_EMPLOYEE";
USE database temp_db; 

CREATE TABLE ORGANIZATION(Org VARIANT);

INSERT INTO ORGANIZATION SELECT parse_json('{"relationships":[
        { "name" : "mother", "value": "a" },
        { "name" : "siblings", "value": ["c", "d"] }
    ]
}');

CREATE TABLE PEOPLE(person VARIANT);
INSERT INTO PEOPLE SELECT Parse_json ('[{
    "id" : "a",
    "name": "Mary"
},
{
    "id": "b",
    "name": "Joe"
},
{
    "id": "c",
    "name": "John"
}]');

SELECT Org, ARRAY_AGG(Person) as People
FROM Organizations
INNER JOIN People ON People.id IN Org.relationships
GROUP BY Org

Desc table organization;
SELECT Org:relationships,
Org:relationships.name,
Org:relationships.value
,value:Org:relationships.value AS NAME
 ,value:value AS value
FROM ORGANIZATION,lateral flatten(input => Org:ORGANIZATION) LIMIT 1;

// like below from dave lab: 
select 
   v:city::varchar(20)
   ,v:city:name::varchar(20)
  ,value:description AS Description::varchar(20)
 ,value:id AS ID
 from SNOWFLAKE_SAMPLE_DATA.weather.weatHer_14_total 
,  lateral flatten(input => v:weather)
 
## ADMINISTRATION TOOLS
SHOW PARAMETERS IN ACCOUNT

##
SE warehouse "TEST_RACHEL_TEMP_EMPLOYEE";

create database temp_db; 

create table employee_date
(emp_id int, emp_name varchar(500), emp_addr varchar(500), emp_date DATE);

 

insert into employee_date (emp_id,emp_name,emp_addr,emp_date)

values(1,'Nala','Pride Rock', '2014-01-01'),

(2,'Simba','Pride Rock','2014-01-01');

 
select hash(*) from employee_date;
select * from employee_date;

//udf
create or replace function add5 (n number)
  returns number
  as 'n + 5';
create or replace function dateprint (n datetime)
  returns timestamp_ntz
  as 'n';
 
 //using to_date
select dateprint(to_date('1536543296'));


## Test for https://community.snowflake.com/s/question/0D50Z00009C7Di1SAF/
create table CUSTOMER_COPY
(TABLE_NAMEa string, TABLE_OWNER varchar(500), IS_TRANSIENT varchar(500), BYTES int,RETENTION_TIME int, ID int,CLONE_GROUP_ID int,IS_TRANSIENT, ACTIVE_BYTES int, TIME_TRAVEL_BYTES int, FAILSAFE_BYTES int, TABLE_CREATED datetime, TABLE_DROPPED datetime, TABLE_ENTERED_FAILSAFE datetime);

 

insert into CUSTOMER_COPY (TABLE_NAMEa, TABLE_OWNER, IS_TRANSIENT, BYTES,RETENTION_TIME, ID,CLONE_GROUP_ID,IS_TRANSIENT, ACTIVE_BYTES, TIME_TRAVEL_BYTES, FAILSAFE_BYTES, TABLE_CREATED, TABLE_DROPPED, TABLE_ENTERED_FAILSAFE);

values('CUSTOMER_COPY','ACCOUNTADMIN','YES', '152317', '1', '1026', 'NO', '0','0', '218112', '0', '20-05-2019 01:21:03', '20-05-2019 01:21:04', '20-05-2019 01:21:05');


select t.*,tsm.* from "SNFLK_TRAINING"."INFORMATION_SCHEMA"."TABLES" t

inner join "SNFLK_TRAINING"."INFORMATION_SCHEMA"."TABLE_STORAGE_METRICS" tsm

on t.TABLE_CATALOG=tsm.TABLE_CATALOG

and t.TABLE_SCHEMA=tsm.TABLE_SCHEMA

and t.TABLE_NAME=tsm.TABLE_NAME

WHERE t.TABLE_CATALOG='SNFLK_TRAINING'

## Working with dates in Snowflake
//For https://community.snowflake.com/s/question/0D50Z00009G2jNASAZ/calling-udf-in-select-statement-throws-error-unsupported-subquery-type-cannot-be-evaluated

USE warehouse "TEST_RACHEL_TEMP_EMPLOYEE";
USE database "TEMP_DB";

CREATE TABLE target (p_target_profile_id NUMBER,p_point_in_date DATE,p_on_sale_date DATE,p_dept_date DATE);


insert into target (p_target_profile_id,p_point_in_date,p_on_sale_date,p_dept_date)
  values(1,'2019-01-01 0','2019-01-02 0', '2019-01-03 0'),
  (2,'2019-01-01 0','2019-01-02 0', '2019-01-03 0');


//CREATE OR REPLACE FUNCTION get_target_from_instance (p_target_profile_id NUMBER,p_point_in_date DATE,p_on_sale_date DATE,p_dept_date DATE)
//RETURNS NUMBER
//AS

## Testing  Clustering in Snowflake
CREATE TABLE TEST (id int, name varchar(10));
 
insert into TEST values(1,'abc');
insert into TEST values(2,'mno');
 
SELECT SYSTEM$CLUSTERING_INFORMATION('TEST','(ID)');

select to_timestamp_tz('2019-06-27T11:57:36.208-0400', 'YYYY-MM-DDThh:mi:ss.FF3TZHTZM');


## Working more with timestamps and semi-structured data
//for https://community.snowflake.com/s/question/0D50Z00009Ht0aXSAR/how-to-convert-time-to-timestamp
SELECT CONCAT(TO_DATE('2019-05-11 00:22:00.000'),' 00:33:27.0000000') AS RESULT;
SELECT CONCAT(TO_DATE('2019-05-11 00:22:00.000'),'00:33:27.0000000')::TIMESTAMP AS RESULT;
SELECT SUM(TO_DATE('2019-05-11 00:22:00.000'),To_date('00:33:27.0000000'))::TIMESTAMP AS RESULT;
//from documentation
select datediff(hour, '2019-05-11 00:22:00.000'::timestamp, 
    dateadd(year, 2, to_date('00:33:27.0000000')::timestamp)) 
               as diff_hours;

To_date('08/12/2018 12:20 PM', 'YYYY-MM-DD');
SELECT ( TO_VARCHAR(DATEADD(hh,-7, CONCAT(TO_DATE('02/21/2019'),' ','12:20 PM':: TIMESTAMP ) ,'MM/DD/YYYY HH:MI AM')));
SELECT '12:20 PM':: TIMESTAMP;
SELECT CONCAT(TO_DATE('02/21/2019'),' ','12:20 PM':: TIMESTAMP );
SELECT (DATEADD(hh,-7, CONCAT(TO_DATE('02/21/2019'),' ','12:20 PM':: TIMESTAMP ) ,'MM/DD/YYYY HH:MI AM'));

## CREATE CALENDAR TABLE 
USE warehouse "TEST_EMPLOYEE";
USE database "TEMP_DB";

CREATE TABLE Holiday_2020(Holiday varchar,date_id varchar,Day varchar); 

insert into Holiday_2020 (Holiday,date_id,Day)
  values('New Year\'s Day', 'January 1', 'Wednesday'), 
('Martin Luther King Jr. Day','January 20','Monday'),
('George Washington’s Birthday','February 17','Monday'),
('Memorial Day', 'May 25','Monday'),
('Independence Day','July 4','Friday'),
('Labor Day','September 7','Monday'),
('Columbus Day','October 12','Monday'),
('Veterans Day','November 11','Wednesday'),
('Thanksgiving Day','November 26','Thursday'),
('Christmas Day','December 25','Friday');

SELECT * FROM Holiday_2020;


## Troubleshooting Sliding Window Function - borrowed code
//for https://community.snowflake.com/s/question/0D50Z00009T2fSLSAZ/how-to-apply-sliding-window-function-on-the-date-field
USE warehouse "TEST_EMPLOYEE";
USE database "TEMP_DB"
--mockup data
with test  (date,id,a,b,c,d) as(
select $1 as date, $2 as id, $3 as a, $4 as b, $5 as c, $6 as d
from values 
('2019-07-15','x_1','ps','e','US','NA'),
('2019-07-15','x_2','ps','e','US','NA'),
('2019-07-15','x_2','ps','e','CA','NA'),
('2019-07-16','x_2','ps','e','CA','NA'),
('2019-07-16','x_3','c','xb','CH','AS'),
('2019-07-17','x_4','ps','e','US','NA'),
('2019-07-17','x_5','c','ps4','CH','AS'),
('2019-07-17','x_6','c','ps4','CH','AS'),
('2019-07-17','x_7','c','ns','CH','AS'),
('2019-07-18','x_7','c','ns','CH','AS'),
('2019-07-18','x_7','c','ns','CH','AS'),
('2019-07-18','x_7','c','ns','CH','AS'),
('2019-07-18','x_7','c','ns','CH','AS'),
('2019-07-19','x_8','c','ps','CH','AS'),
('2019-07-20','x_8','c','ps','CH','AS'),
('2019-07-21','x_8','c','ps','CH','AS'),
('2019-07-22','x_8','c','ps','CH','AS'),
('2019-07-29','x_8','c','ps','CH','AS'),
('2019-08-01','x_8','c','ps','CH','AS'),
('2019-08-02','x_9','c','ps','CH','AS'),
('2019-08-03','y_1','c','ps','CH','AS'),
('2019-08-04','y_8','c','ps','CH','AS'),
('2019-08-05','z_8','c','ps','CH','AS'),
('2019-08-06','a_8','c','ps','CH','AS'),
('2019-08-07','b_8','c','ps','CH','AS'),
('2019-08-07','c_8','c','ns','CH','AS'),
('2019-08-07','e_8','ps','e','US','NA'),
('2019-08-08','f_8','c','xb','CH','AS'),
('2019-08-08','f_8','c','xb','CH','AS'),
('2019-08-08','f_8','c','xb','CH','AS'),
('2019-08-09','f_8','c','xb','CH','AS'),
('2019-08-09','f_8','c','xb','CH','AS'),
('2019-08-09','f_8','c','xb','CH','AS'),
('2019-08-10','gx_8','c','ps','CH','AS'),
('2019-08-11','v_8','c','ps','CH','AS') )
--subquery
select 1,
(
  select sum(x)
  from (
select count(distinct id) as x
from test
where
 date = '2019-07-16'
group by a,b,c,d) ) as dau,
(
  select sum(x)
  from (
select count(distinct id) as x
from test
where
 date between '2019-07-16'::date-interval '7 days' and '2019-07-16'
//    <= '2019-07-15'
//and date >= '2019-07-15'::date-interval '7 days'
group by a,b,c,d)) as w,
(select sum(x)
  from (
select count(distinct id) as x
from test
where
 date <= '2019-07-16'
and date >= '2019-07-16'::date-interval '30 days'
group by a,b,c,d)) as m
union
--window function
select 2,sum(dau),sum(w),sum(m)
from(
select 
date,
a,
b,
c,
d,
count(distinct id) over (Partition by date,a,b,c,d Order by date)as dau,
count(distinct id) over (Partition by a,b,c,d Order by date rows between 7 preceding  and current row) as w,
count(distinct id) over (Partition by a,b,c,d Order by date rows between 30 preceding  and current row) as m
from test
group by
date,
a,
b,
c,
d,
id)
where date='2019-07-16'
group by date
;
Xxlearn result scan: 
USE warehouse "TEST_RACHEL_TEMP_EMPLOYEE";
USE database "TEMP_DB";

select last_query_id(-2);

desc result '018f218c-0170-673e-0000-4325002185ae';

select * from target limit 2; 

desc result last_query_id();
select last_query_id();



select * from table(result_scan(last_query_id()));
select c2 from table(result_scan('ce6687a4-331b-4a57-a061-02b2b0f0c17c'));
xx remix example
// for https://stackoverflow.com/questions/54317671/snowflake-regular-expressions
CREATE TEMPORARY TABLE mytable  
( mystring varchar(10) );

INSERT INTO mytable values ('00000');
INSERT INTO mytable values ('  ');
INSERT INTO mytable values ('1234');


select mystring from mytable where regexp_replace(mystring, '.{1}.');

select mystring, 
      0 as parsed_string -- need to be changed to get 0 or 1 
from mytable;
