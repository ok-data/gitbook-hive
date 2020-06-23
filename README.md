# Hive

`test`

`Hive` 는 Hadoop의 데이터를 다루는 SQL 질의의 표준이다. HDFS의 데이터를 SQL로 접근할 수 있는 방법을 제공한다.

Hive Query Language, 즉 HQL을 사용하여 RDBMS의 표준 SQL과 비슷한 의미와 함수를 가지며, RDBMS 경험자라면 손쉽게 사용할 수 있다.

또한, HDFS 상에서 High 레벨의 데이터 모델이며, 테이블과 같은 구조를 제공한다.

Hive는 `table`, `partition`, `bucket`, 세개의 데이터 구조를 지원하고 table을 HDFS 디렉토리에 대응하고, 이를 파티션으로 분할할 수 있으며, 파티션을 버킷으로 차례대로 나눌 수 있다.

`TIMESTAMP`, `STRING`, `FLOAT`, `BOOLEAN`, `DECIMAL`, `DOUBLE`, `INT`, `SMALLINT`, `BIGINT` 같은 primitive(원시) 데이터 포맷 대부분을 지원하고

`UNION`, `STRUCT`, `MAP`, `ARRAY` 와 같은 복잡한 데이터 연산을 지원한다.

![Hive Architecture](https://miro.medium.com/max/628/0*d5DOvZIR_O4PPYlb)

\[출처](https://medium.com/plumbersofdatascience/hive-architecture-in-depth-ba44e8946cbc)

위 그림은 하이브 아키텍처를 그린것으로, 

`metadata store` 는 내장, 로컬, 원격 데이터베이스를 사용할 수 있다.

`Apache Thrift`를 통해 Hive에 접근할 수 있고, CLI와 웹 인터페이스를 활용할 수 있다.

Hive의 장점을 모아 보면 다음과 같다

1. Hive는 Map/Reduce 보다 간단하고, 간단한 질의 모델을 개발방법을 제공한다.
2. HQL과 SQL 문법이 비슷하다
3. 쉽게 분석할 수 있는 여러 API, 함수를 제공한다.
4. 많은 데이터 집합에서 동일한 타입의 질의 응답 시간은 여타 질의 응답 시간보다 일반적으로 훨씬 빠르다
5. 여러 컴퓨팅 프레임워크에서 동작한다.
6. HDFS에 애드혹 질의를 할 수 있다.
7. Hive는 사용자가 정의한 함수, 스크립트, 사용자 I/O등의 확장가능한 기술을 제공한다.

> Map/Reduce의 high 패키지로 사실상 모든 쿼리는 map/reduce가 수행된다.