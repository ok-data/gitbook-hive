# Create

```sql
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name
  [(컬럼명 data_type [제약조건] [COMMENT col_comment], ...]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
  [CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
  [SKEWED BY (col_name, col_name, ...)]
     ON ((col_value, col_value, ...), (col_value, col_value, ...), ...)
     [STORED AS DIRECTORIES]
  [
   [ROW FORMAT row_format] 
   [STORED AS file_format]
     | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]
  ]
  [LOCATION hdfs_path]
  [TBLPROPERTIES (property_name=property_value, ...)]
  [AS select_statement];
```

`data_type` 에는 [참고]({{site.baseurl}}/Data_Types.html)

## 내부 / 외부 테이블

데이터를 내부 테이블에서 유지하는 경우, 하이브는 모든 테이블과 데이터의 모든 생명 주기를 관리한다.

내부 테이블이 삭제되면 외부 테이블도 삭제되고, 외부 테이블이 삭제되면 내부 테이블도 삭제된다.

실수로 테이블과 함꼐 데이터가 삭제되는 것을 방지하기 위해 외부 테이블로 만드는 것이 권장된다.

다음은 내부/외부 테이블 생성 예시이다.

```bash
## 샘플 데이터
$ hdfs dfs -cat /user/hive/employee.txt
```
```
Michael|Montreal,Toronto|Male,30|DB:80|Product:DeveloperLead
Will|Montreal|Male,35|Perl:85|Product:Lead,Test:Lead
Shelley|New York|Female,27|Python:80|Test:Lead,COE:Architect
Lucy|Vancouver|Female,57|Sales:89,HR:94|Sales:Lead
```

```SQL
-- 내부 테이블 생성 
CREATE TABLE IF NOT EXISTS employee_internal
(
    name string,
    work_place ARRAY<string>,
    sex_age STRUCT<sex:string, age:int>,
    skills_score MAP<string, int>,
    depart_title MAP<string, ARRAY<string>>
)
COMMENT 'This is an internal table'
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '|'
COLLECTION ITEMS TERMINATED BY ','
MAP KEYS TERMINATED BY ':'
STORED AS TEXTFILE;
```

```SQL
-- 데이터 쓰기
-- LOAD DATA LOCAL INAPTH 는 로컬 경로
LOAD DATA INPATH 'employee.txt'
OVERWRITE INTO TABLE employee_internal;
```

> 데이터를 옮기고 나면 원본파일이 삭제되므로 참고!!

```SQL
-- 외부 테이블 생성
CREATE EXTERNAL TABLE employee_external
(
    name string,
    work_place ARRAY<string>,
    sex_age STRUCT<sex:string, age:int>,
    skills_score MAP<string, int>,
    depart_title MAP<string, ARRAY<string>>
)
COMMENT 'This is an internal table'
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '|'
COLLECTION ITEMS TERMINATED BY ','
MAP KEYS TERMINATED BY ':'
STORED AS TEXTFILE
LOCATION '/user/hive/employee';
```
```SQL
-- 데이터 쓰기
LOAD DATA INPATH 'employee.txt'
OVERWRITE INTO TABLE employee_external;
```

## Storage Formats

`STORED AS` 는 테이블저장 포맷을 지정할 수 있다. 아래는 지원하는 파일 포맷이다.

|Storage Format|설명|
|--------------|----|
|STORED AS TEXTFILE|default인 저장 포맷,  plain text files로 저장 된다.|
|STORED AS SEQUENCEFILE|Stored as compressed Sequence File.|
|STORED AS ORC|Stored as ORC file format. Stores column-level metadata.|
|STORED AS PARQUET|Stored as Parquet format for the Parquet columnar storage format in Hive 0.13.0 and later|
|STORED AS AVRO|AVRO 형식 파일 포맷 저장|
|STORED AS JSONFILE|Hive 4.0.0 이후 지원|

## Row Formats & SerDe

`ROW FORMAT`은 데이터를 칼럼 단위로 구분자 delimeter 와 데이터를 해석하는 방법을 지정하는 `SerDe`를 지정할 수 있다.

`DELIMITED` :

Hive는 구분자에 따라서 데이터를 분리하여 칼럼 단위로 보여준다.

- 기본 구분자
- 칼럼 구분자

## CTAS

테이블은 다른 테이블에서 조회된 결과로 생성될 수 있다. CTAS : create-table-as-select

```SQL
CREATE TABLE ctas_employee
AS SELECT * FROM employee_external;
```

다음과 같이 CTE, Common Table Expression가 포함된 CTAS를 생성할 수 있다.

```SQL
CREATE TABLE cte_employee AS
WITH r1 AS (SELECT name FROM r2 WHERE name = 'Michael'),
r2 AS (SELECT name FROM employee WHERE sex_age.sex = 'MALE'),
r3 AS (SELECT name FROM employee WHERE Sex_age.sex = 'FEMALE')
SELECT * FROM r1 UNION ALL select * FROM r3;
```
```SQL
SELECT * FROM cte_employee;
```

## 빈 테이블 만들기 

다음과 같이 where 절을 추가하여, 칼럼만 복사한 테이블을 생성할 수 있다.
```SQL
CREATE TABLE empty_ctas_employee AS 
SELECT * FROM employee_internal WHERE 1=2;
```

**LIKE를 사용하여 빈 테이블 만들기** :

칼럼만 복사한 빈 테이블을 만든다.
```SQL
CREATE TABLE empty_like_employee
LIKE employee_internal;
```