# 버킷

`bucket` 은 쿼리 성능을 최적화하기 위해 데이터 집합을 관리할 수 있는 작은 부분으로 잘게 나누는 또 다른 기술이다.

지정된 칼럼의 값을 해쉬 처리하고 지정한 수의 파일로 나누어 저장한다. join에 사용되는 키로 버킷 칼럼을 생성하면, SMB(sort merge bucket) 조인으로 처리되어 수행속도가 빨라진다.

```SQL
-- 예시
CREATE TABLE employee_id
(
    name string,
    work_place ARRAY<string>,
    sex_age STRUCT<sex:string, age:int>,
    skills_score MAP<string, int>,
    depart_title MAP<string, ARRAY<string>>
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '|'
COLLECTION ITEMS TERMINATED BY ','
MAP KEYS TERMINATED BY ':';
```

```bash
vim employee_id.txt
##
Michael|100|Montreal,Toronto|Male,30|DB:80|Product:DeveloperLead
Will|101|Montreal|Male,35|Perl:85|Product:Lead,Test:Lead
Steven|102|New York|Female,27|Python:80|Test:Lead,COE:Architect
Lucy|103|Vancouver|Female,57|Sales:89,HR:94|Sales:Lead
Mike|104|Montreal|Male,35|Perl:85|Product:Lead,Test:Lead
Shelley|105|New York|Female,27|Python:80|Test:Lead,COE:Architect
Luly|106|Vancouver|Female,57|Sales:89,HR:94|Sales:Lead
Lily|107|Montreal|Male,35|Perl:85|Product:Lead,Test:Lead
Shell|108|New York|Female,27|Python:80|Test:Lead,COE:Architect
Mich|109|Vancouver|Female,57|Sales:89,HR:94|Sales:Lead
Dayong|110|Montreal|Male,35|Perl:85|Product:Lead,Test:Lead
Sara|111|New York|Female,27|Python:80|Test:Lead,COE:Architect
Roman|112|Vancouver|Female,57|Sales:89,HR:94|Sales:Lead
Christine|113|Montreal|Male,35|Perl:85|Product:Lead,Test:Lead
Eman|114|New York|Female,27|Python:80|Test:Lead,COE:Architect
Alex|115|Vancouver|Female,57|Sales:89,HR:94|Sales:Lead
Alan|116|Montreal|Male,35|Perl:85|Product:Lead,Test:Lead
Andy|117|New York|Female,27|Python:80|Test:Lead,COE:Architect
Ryan|118|Vancouver|Female,57|Sales:89,HR:94|Sales:Lead
Rome|119|Montreal|Male,35|Perl:85|Product:Lead,Test:Lead
Lym|120|New York|Female,27|Python:80|Test:Lead,COE:Architect
Linm|121|Vancouver|Female,57|Sales:89,HR:94|Sales:Lead
Dach|122|Montreal|Male,35|Perl:85|Product:Lead,Test:Lead
Ilon|123|New York|Female,27|Python:80|Test:Lead,COE:Architect
Elaine|124|Vancouver|Female,57|Sales:89,HR:94|Sales:Lead
## wq
hdfs dfs -copyFromLocal employee_id.txt
```

```SQL
-- 데이터 삽입
LOAD DATA INPATH '/user/hive/employee_id.txt'
OVERWRITE INTO TABLE employee_id;
```

```SQL
-- 예시
-- 버킷 테이블 생성
CREATE TABLE employee_id_buckets
(
    name string,
    work_place ARRAY<string>,
    sex_age STRUCT<sex:string, age:int>,
    skills_score MAP<string, int>,
    depart_title MAP<string, ARRAY<string>>
)
CLUSTERED BY (employee_id) INTO 2 BUCKETS --버킷 개수 지정
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '|'
COLLECTION ITEMS TERMINATED BY ','
MAP KEYS TERMINATED BY `':';
```

> 버킷 개수 : 적절한 버킷수를 정의하려면 각 버킷에 너무 많거나 적은 데이터를 사용하지 않아야 한다. 2 블록으로 설정한다면, 하둡의 블록 크기가 256MB이면 각 버킷에 512MB의 데이터로 계획할 수 있다. 가능하다면 2^N을 버킷 개수로 사용한다.

데이터를 버킷 테이블에 저장하려면 일반 테이블에서 작업하는 것처럼 LOAD키를 사용하지 않고, 대신 INSERT 문을 사용한다.

```SQL
INSERT OVERWRITE TABLE employee_id_buckets
SELECT * FROM employee_id;
```

```bash
[hive@centos-namenode-1 ~]$ hdfs dfs -cat warehouse/employee_id_buckets/000000_0
Michael|\N|Male,30|DB:80,\N|Product:\N|\N
Will|\N|Male,35|Perl:85,\N|Product:\N,Test:\N|\N
Shelley|\N|Female,27|Python:80,\N|Test:\N,COE:\N|\N
Lucy|\N|Female,57|Sales:89,\N|Sales:\N|\N
```