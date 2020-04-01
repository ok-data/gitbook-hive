# SELECT

하이브를 운영하면 가장 많이 하는 것은 데이터에 대한 query,질의이다. 

주로 `FROM`, `DISTINCT`, `WHERE`, `LIMIT` 키워드를 활용해서 쿼리를 수행할 수 있다.

```SQL
SELECT * FROM employee;
```
```
|name   |work_place            |sex_age                    |skills_score        |depart_title                         |
|-------|----------------------|---------------------------|--------------------|-------------------------------------|
|Michael|["Montreal","Toronto"]|[{"sex":"Male","age":30}]  |{"DB":80}           |{"Product":["DeveloperLead"]}        |
|Will   |["Montreal"]          |[{"sex":"Male","age":35}]  |{"Perl":85}         |{"Product":["Lead"],"Test":["Lead"]} |
|Shelley|["New York"]          |[{"sex":"Female","age":27}]|{"Python":80}       |{"Test":["Lead"],"COE":["Architect"]}|
|Lucy   |["Vancouver"]         |[{"sex":"Female","age":57}]|{"Sales":89,"HR":94}|{"Sales":["Lead"]}                   |
```
```SQL
-- 칼럼 별 조회
SELECT name FROM employee;
```
```
name   |
-------|
Michael|
Will   |
Shelley|
Lucy   |
```

## DISTINCT, LIMIT

`DISTINCT` 는 테이블에서 중복에 상관없이 하나의 유일한 행만 반환한다.

`LIMIT <n>` 은 무작위 행을 선택하고 지정한 n개만큼 반환한다.

```SQL
SELECT DISTINCT name FROM employee LIMIT 2;
```
```
name   |
-------|
Michael|
Will   |
```

`hive.fetch.task.conversion = more`를 설정하면 하이브의 fetch를 활성화시켜서 map/reduce수행을 막을 수 있다. 

나름의 쿼리 실행속도를 향상시킬 수 있다.

```SQL
SET hive.fetch.task.conversion=more;
```

## WHERE

WHERE 문은 Boolean, 사용자 정의 함수로 쿼리 실행시 데이터를 비교하여 질의를 수행할 수 있다.

```SQL
SELECT name, work_place FROM employee
WHERE name = 'Michael';
```
```
name   |work_place            |
-------|----------------------|
Michael|["Montreal","Toronto"]|
```

`WITH` 문을 FROM이나 WHERE 문 뒤에 호출함으써 서브쿼리를 수행할 수 있다.

```SQL
-- 서브쿼리 예시
WITH t1 AS (
    SELECT * FROM employee
    WHERE sex_age.sex = 'Male')
SELECT name, sex_age.sex AS sex FROM t1;
```
```
name   |sex |
-------|----|
Michael|Male|
Will   |Male|
```

FROM 문 뒤에 중첩 SELECT를 활용한 예시이다.
```SQL
SELECT name, sex_age.sex AS sex
FROM (
    SELECT * FROM employee
    WHERE sex_age.sex = 'Male'
) t1;
```
```
name   |sex |
-------|----|
Michael|Male|
Will   |Male|
```

WHERE 문은 `IN`, `NOT IN`, `EXIST`, `NOT EXIST` 을 통해 데이터를 비교할 수 있는데

WHERE로 호출할 칼럼에 대한 alias(별명)를 설정하지 않으면 에러가 발생한다.

```SQL
SELECT name, sex_age.sex AS sex
FROM employee a
WHERE a.name IN
(SELECT name FROM employee
WHERE sex_age.sex = 'Male')
```
```SQL
-- 위와 동일한 결과
SELECT name
FROM employee 
WHERE sex_age.sex = 'Male';
```

## JOIN

하이브의 JOIN은 2 개 이상의 테이블을 합치기 위해 사용된다.

JOIN 관련 명령어 :

- `JOIN`
- `LEFT OUTER JOIN`
- `RIGHT OUTER JOIN`
- `FULL OUTER  JOIN`
- `CROSS JOIN`

데이터 준비 :

```SQL
CREATE TABLE IF NOT EXISTS employee_hr
(
    name string,
    employee_id int,
    sin_number string,
    start_date date
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '|'
STORED AS TEXTFILE;
```

```bash
$ vim employee_hr.txt
##
Michael|100|547-968-091|2014-01-29
Will|101|527-948-090|2013-10-02
Steven|102|647-968-598|2012-11-03
Lucy|103|577-928-094|2010-01-03
##
$ hdfs dfs -copyFromLocal employee_hr.txt /user/hive/
```

```SQL
-- txt 파일 업로드
LOAD DATA INPATH
'/user/hive/employee_hr.txt'
OVERWRITE INTO TABLE employee_hr;
```

두 테이블간의 `INNER JOIN`을 수행한다.

```SQL
SELECT emp.name, emph.sin_number
FROM employee emp
JOIN employee_hr emph ON emp.name = emph.name;
```
```
name   |sin_number |
-------|-----------|
Michael|547-968-091|
Will   |527-948-090|
Lucy   |577-928-094|
```

여러 테이블 간의 다음과 같이 다중 JOIN문을 사용할 수 있다 (3개) :

```SQL
SELECT emp.name, empi.employee_id, emph.sin_number
FROM employee emp
JOIN employee_hr emph ON emp.name = emph.name
JOIN employee_id empi ON emp.name = empi.name;
```
```
name   |employee_id|sin_number |
-------|-----------|-----------|
Michael|        100|547-968-091|
Will   |        101|527-948-090|
Lucy   |        103|577-928-094|
```

### Self JOIN

테이블 자기 자신과의 JOIN 연산이다. 구별하기 위해 alias를 사용해야 한다.

```SQL
SELECT emp.name
FROM employee emp
JOIN employee emp_b
ON emp.name = emp_b.name;
```
```
name   |
-------|
Michael|
Will   |
Shelley|
Lucy   |
```