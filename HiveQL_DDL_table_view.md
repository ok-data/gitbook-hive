# 뷰

뷰는 join, subquery, filter, 평탄호와 같은 복잡성을 숨겨서 쿼리를 간단히 만들기 위해 사용되는 논리적인 데이터 구조이다. 

일부 RDBMS와는 다르게, 하이브 뷰는 데이터를 저장하거나 실체화된 데이터가 없다.

뷰를 한번 만들면 관련 schema는 즉시 고정상태로, 칼럼추가, 변경등은 반영되지 않는다.

```SQL
-- 예시
CREATE VIEW employee_skills
AS
SELECT name, skills_score['DB'] AS DB,
skills_score['Perl'] AS Perl,
skills_score['Python'] AS Python,
skills_score['Sales'] AS Sales,
skills_score['HR'] AS HR
FROM employee ;
```

뷰를 생성할 때, 메타데이터만 변경되기 때문에 map/reduce 작업은 전혀 실행되지 않는다.

하지만, 뷰를 쿼리할 때 적절한 맵리듀스 잡이 실행될 수 있다.

## 뷰 속성 변경

```SQL
ALTER VIEW employee_skills
SET TBLPROPERTIES ('comment' = 'This is a View');
```

## 뷰 재정의

```SQL
ALTER VIEW employee_skills AS
SELECT * FROM employee;
```

## 뷰 삭제

```SQL
DROP VIEW employee_skills;
```