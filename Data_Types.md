# Data Types

[공식문서](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-char)

## Numeric Types

|Numeric Types|설명
|-------|---------------------|
|TINYINT|1-byte signed integer, -128 ~ 127|
|SMALLINT|2-byte signed integer, -32768 ~ 32767|
|INT|4-byte signed integer, -2147483648 to 2147483,647|
|BIGINT|8-byte signed integer, -9223372036854775808 ~ 9223372036854775807|
|FLOAT|4-byte 단일 정밀도 부동 소수점|
|DOUBLE|8-byte 배정밀도 부동 소수점 숫자|
|DOUBLE PRECISION|DOUBLE의 alias, Hive 2.2.0부터 지원|
|DECIMAL|JAVA의 BigDecimal 타입과 동일, 오차없이 실수를 표기 할 수 있다|
|NUMERIC|Hive 3.0.0부터 지원, DECIMAL과 동일함|

## Date/Time Types

|Date/Time Types|설명|예시|
|---------------|----|----|
|TIMESTAMP|`yyyy-mm-dd hh:mm:ss[.f...]` 포맷의 년,달,일,시간,분,초,밀리초 로 표현한다.|2013-01-01 12:00:01.345|
|DATE|`YYYY-­MM-­DD` 포맷의 년,달,일로 날짜를 표기한다.|2013-01-01|

## String Types

|String Types|설명|예시|
|------------|----|----|
|STRING|`'` 또는 `"` 로 표현되는 문자열 포맷, 최대 길이는 255|'Boos' or "Books"|
|VARCHAR|가장 큰 문자열 포맷, 최대 길이는 65355, 길이는 넘어가면 긴 부분은 잘려나간다|'Books' or "Books"|
|CHAR|VARCHAR와 비슷하지만 255크기로 고정되어 있다. 255보다 작으면 나머지는 빈칸으로 채워 255크기를 맞춘다|'Books' or "Books"|

## Misc Types

|Misc Types|설명|
|----------|----|
|BOOLEAN|TRUE or FALSE|
|BINARY|STRING 이다|

## Complex Types

|Complex Types|설명|예시|
|-------------|----|----|
|ARRAY|`ARRAY<data_types>`, 데이터 타입을 가진 배열 집합|['apple','orange','mango']|
|MAP| `MAP<primitive_type, data_type>`, (K:V)키값형식의 데이터 집합, |{1:"apple", 2: "orange"}|
|STRUCT|`STRUCT<col_name : data_type, ...>`, 데이터 타입 상관없이 사용자 정의 가능한 배열 집합 | {1, "apple"}|
|UNION|`UNIONTYPE<data_type, data_type...>`, 모든 데이터 타입 중 하나의 데이터 타입만 갖는 집합 | {2:["apple", "orange"]}|