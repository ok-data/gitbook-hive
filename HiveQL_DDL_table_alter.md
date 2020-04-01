# Alter

`ALTER`는 주로 테이블의 구조변경할 때 사용된다. 컬럼을 추가하거나, SerDe 추가, 속성등에 대한 변경이 가능하다.

## 테이블명 변경

```SQL
ALTER TABLE table_name RENAME TO new_table_name;
```

```SQL
--예시
ALTER TABLE cte_employee RENAME TO c_employee;
```

## 테이블 속성 변경

```SQL
ALTER TABLE table_name SET TBLPROPERTIES table_properties;
```
```SQL
--예시
ALTER TABLE c_employee
SET TBLPROPERTIES ('comment'='New name, comments');
```

## 테이블 구분자 변경
```SQL
ALTER TABLE table_name [PARTITION partition_spec] SET SERDE serde_class_name [WITH SERDEPROPERTIES serde_properties];
 
ALTER TABLE table_name [PARTITION partition_spec] SET SERDEPROPERTIES serde_properties;
```

```SQL
--예시
ALTER TABLE employee_internal 
SET SERDEPROPERTIES ('field.delim' = '$');
```

## 테이블 파일 포맷 변경
```SQL
--예시
ALTER TABLE c_employee 
SET FILEFORMAT RCFILE;
```

## 테이블 저장소 변경
```SQL
--예시
ALTER TABLE c_employee
SET LOCATION 'hdfs://centos-namenode-1:9000/user/hive/employee';
```

## 정책 변경

`ENABLE/DISABLE`으로 보호 정책 등을 변경할 수 있다.

`NO_DROP`은 테이블 삭제 방지, `OFFLINE` 은 쿼리로부터 테이블의 데이터를 보호한다.(메타데이터는 보호 안됨)

```SQL
ALTER TABLE c_employee ENABLE NO_DROP; --삭제 방지
ALTER TABLE c_employee DISABLE NO_DROP;
ALTER TABLE c_employee ENABLE OFFLINE; --쿼리 
ALTER TABLE c_employee DISABLE OFFLINE;
```

## 테이블 컬럼 타입 변경

```SQL
ALTER TABLE table_name [PARTITION partition_spec] CHANGE [COLUMN] col_old_name col_new_name column_type
  [COMMENT col_comment] [FIRST|AFTER column_name] [CASCADE|RESTRICT];
```


```SQL
--변경 전에 칼럼 타입 확인
DESC employee_internal;
```
```
col_name    |data_type                 |comment|
------------|--------------------------|-------|
name        |string                    |       |
work_place  |array<string>             |       |
sex_age     |struct<sex:string,age:int>|       |
skills_score|map<string,int>           |       |
depart_title|map<string,array<string>> |       |
```

```SQL
--예시
-- name 칼럼명 -> employee_name, sex_age 칼럼 뒤로 옮기기
ALTER TABLE employee_internal
CHANGE name employee_name string AFTER sex_age;

DESC employee_internal;
```
```
col_name     |data_type                 |comment|
-------------|--------------------------|-------|
work_place   |array<string>             |       |
sex_age      |struct<sex:string,age:int>|       |
employee_name|string                    |       |
skills_score |map<string,int>           |       |
depart_title |map<string,array<string>> |       |
```

```SQL
-- 원래대로 되돌리기
ALTER TABLE employee_internal
CHANGE employee_name name string FIRST;

DESC employee_internal;
```
```
col_name    |data_type                 |comment|
------------|--------------------------|-------|
name        |string                    |       |
work_place  |array<string>             |       |
sex_age     |struct<sex:string,age:int>|       |
skills_score|map<string,int>           |       |
depart_title|map<string,array<string>> |       |
```

## 칼럼 추가 및 치환

```SQL
ALTER TABLE table_name 
  [PARTITION partition_spec]
  ADD|REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...)
  [CASCADE|RESTRICT] 
```

```SQL
--예시
--work라는 string 타입 컬럼 추가
ALTER TABLE c_employee 
ADD COLUMNS (work string);
```

```SQL
--예시
--모든 칼럼을 치환
ALTER TABLE c_employee
REPLACE COLUMNS (name string);
```