# Database 관리

## Create

```SQL
CREATE (DATABASE|SCHEMA) [IF NOT EXISTS] database_name
  [COMMENT database_comment]
  [LOCATION hdfs_path]
  [WITH DBPROPERTIES (property_name=property_value, ...)];
```

`DATABASE|SCHEMA` , 둘중 하나를 고르면 되며 같은 의미이다.

새로운 데이터베이스가 생성될 때마다 `hive.metastore.warehouse.dir`에 정의한 `/user/hive/warehouse`에 개별 데이터베이스를 위한 디렉토리가 생성된다. default 데이터베이스는 디렉토리를 생성하지 않는다.

```SQL
-예시
CREATE DATABASE myhivebook;

-- 존재여부 확인 후 생성
CREATE DATABASE IF NOT EXISTS myhivebook;
```

저장 위치, 주석, 메타데이터 정보를 포함한 데이터베이스 생성 :

```SQL
CREATE DATABASE IF NOT EXISTS myhivebook
COMMENT 'hive database demo'
LOCATION '/hdfs/path'
WITH DBPROPERTIES ('creator'='dayongd', 'date'='2020-01-01')
```

## Drop

```SQL
DROP (DATABASE|SCHEMA) [IF EXISTS] database_name [RESTRICT|CASCADE];
```

DROP 실행시 데이터베이스에 테이블이 있는 경우, `RESTRICT` 상태로 대체한다.

테이블과 같이 날리고 싶을 경우 `DROP DATABASE ...CASCADE`를 실행 해주면 된다.

```SQL
-- 빈 데이터베이스 삭제
DROP DATABASE IF EXISTS myhivebook;
-- 테이블 포함 영구 삭제
DROP DATABASE IF EXISTS myhivebook CASCADE;
```

## Alter

```SQL
-- 구조 변경 시
ALTER (DATABASE|SCHEMA) database_name SET DBPROPERTIES (property_name=property_value, ...);
-- 소유권 변경
ALTER (DATABASE|SCHEMA) database_name SET OWNER [USER|ROLE] user_or_role;
-- 파일 저장 경로 변경
ALTER (DATABASE|SCHEMA) database_name SET LOCATION hdfs_path;
```

The ALTER DATABASE ... SET LOCATION statement does not move the contents of the database's current directory to the newly specified location. It does not change the locations associated with any tables/partitions under the specified database. It only changes the default parent-directory where new tables will be added for this database. This behaviour is analogous to how changing a table-directory does not move existing partitions to a different location.

```SQL
-- 데이터베이스 속성 변경
ALTER DATABASE myhivebook
SET DBPROPERTIES ('edited-by'='Dayong');

ALTER DATABASE myhivebook SET OWNER user dayongd;
```

## Use

```sql
USE database_name;
USE DEFAULT;
```

USE는 현재 HiveQL 명령을 실행할 데이터베이스를 지정한다. 

## Show

테이블, 테이블 속성, 테이블 DDL, 인덱스, 파티션, 칼럼, 함수 등과 같은 하이브 객체의 넓은 범위를 지원한다.

```SQL
SHOW DATABASES LIKE  'de*';
```

## Describe

데이터 베이스, 테이블, 뷰, 칼럼 파티션과 같은 하이브 객체의 작은 범위를 지원한다.

```SQL
DESCRIBE DATABASE default;
```
```
db_name|comment              |location                                         |owner_name|owner_type|parameters|
-------|---------------------|-------------------------------------------------|----------|----------|----------|
default|Default Hive database|hdfs://centos-namenode-1:9000/user/hive/warehouse|public    |ROLE      |          |
```