# Truncate

```SQL
TRUNCATE TABLE table_name [PARTITION partition_spec];
```

테이블의 모든 row를 삭제한다. 단, 해당 테이블은 내부 테이블 이어야 한다.

```SQL
--예시
TRUNCATE TABLE cte_employee;
```
