# Drop

```SQL
DROP TABLE [IF EXISTS] table_name [PURGE];
```

`DROP`은 메타데이터를 완벽히 삭제하고 데이터를 `.Trash`로 이동한다. 

```SQL
-- 예시
DROP TABLE IF EXISTS empty_ctas_employee;
```
