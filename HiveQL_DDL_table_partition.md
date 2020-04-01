# 파티션, Partition

하이브는 파일 기반 테이블로 테이블의 모든 row 정보를 읽기 때문에 데이터가 많은 테이블을 쿼리를 돌리면 성능저하가 발생한다. 

하이브 파티션은 RDBMS의 파티션과 매우 유사하며, 각 파티션은 미리 정의된 파티션 칼럼에 해당되고 HDFS 테이블의 디렉토리의 하위 디렉토리로 나눠서 저장한다. 테이블이 쿼리를 받으면, 테이블 데이터 중 필요한 파티션(디렉토리)만 쿼리되기 때문에 I/O가 많이 줄 수 있다.

## 파티션 생성
```SQL
--예시
CREATE TABLE employee_partitioned
(
    name string,
    work_place ARRAY<string>,
    sex_age STRUCT<sex:string, age:int>,
    skills_score MAP<string, int>,
    depart_title MAP<string, ARRAY<string>>
)
PARTITIONED BY (Year INT, Month INT) -- 파티션 정의
OW FORMAT DELIMITED
FIELDS TERMINATED BY '|'
COLLECTION ITEMS TERMINATED BY ','
MAP KEYS TERMINATED BY ':'
```
```SQL
--파티션 확인
SHOW PARTITIONS employee_partitioned;
```
```
|partition|
|---------|
```

```SQL
-- 다중 파티션 추가
ALTER TABLE employee_partitioned ADD
PARTITION (year=2019, month=11)
PARTITION (year=2019, month=12);
```
```SQL
--파티션 확인
SHOW PARTITIONS employee_partitioned;
```
```
|partition         |
|------------------|
|year=2019/month=11|
|year=2019/month=12|
```

파티션 테이블에 데이터 적재 :

```SQL
LOAD DATA INPATH 'employee.txt'
OVERWRITE INTO TABLE employee_partitioned
PARTITION (year=2019, month=12);
```

HDFS의 데이터저장소를 열어보면 다음과 같이 디렉토리가 생성된다.

```bash
warehouse/employee_partitioned/year=2019/month=12/employee.txt
```
select 해보면 year, month 칼럼이 추가된다.
```SQL
SELECT name, year, month FROM employee_partitioned;
```
```
|name   |year|month|
|-------|----|-----|
|Michael|2019|   12|
|Will   |2019|   12|
|Shelley|2019|   12|
|Lucy   |2019|   12|
```
## 고정 파티션

고정 파티션을 이용한 데이터 입력은 INSERT문에 파티션 정보를 고정된 값으로 전달하여 입력한다.

다음은 yymmdd 파티션에 '20190510' 값을 직접적으로 전달하여 파티션을 생성하는 예제이다.

```SQL
INSERT INTO TABLE tbl(yymmdd='201910510')
SELECT name FROM temp;
```
명령을 수행하면 다음과 같은 폴더 구조로 데이터를 생성한다.

```bash
hdfs://[tbl 테이블 로케이션]/yymmdd=20180510/
```

## 동적 파티션

동적 파티션을 이용한 데이터 입력은 INSERT문에 파티션 정보를 조회하는 칼럼을 전달하여 입력한다.

다음은 yymmdd 파티션에 yymmdd 칼럼을 전달하여 파티션을 동적으로 생성하는 예제이다.

```SQL
INSERT INTO TABLE tbl(yymmdd)
SELECT name, yymmdd FROM temp;
```

위와 같이 데이터를 입력할 때 temp 테이블의 yymmdd 칼럼에 20180510,20180511 두개의 데이터가 있으면 다음과 같이 생성된다.

```bash
hdfs://[tbl 테이블 로케이션]/yymmdd=20180510/
hdfs://[tbl 테이블 로케이션]/yymmdd=20180511/
```

## 파티션 수정/삭제

`ALTER TABLE ADD PARTITION` 으로 기존 테이블에 파티션작업을 수행할 수 있다.

파일 포맷, 위치, 보호, 접합에 대한 ALTER TABLE/PARTITION 문은 ALTER TABLE 과 동일한 문법을 가진다.

```SQL
ALTER TABLE table_name PARTITION partition_spec SET FILEFORMAT file_format;
ALTER TABLE table_name PARTITION partition_spec SET LOCATION 'full URI';
ALTER TABLE table_name PARTITION partition_spec ENABLE/DISABLE NO_DROP;
ALTER TABLE table_name PARTITION partition_spec CONCATENATE;
```