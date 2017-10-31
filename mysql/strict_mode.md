# Mysql Strict mode

MySQL은 유효하지 않거나 올바르지 않은 데이터를 유효한 데이터로 강제 변환함.
이를 허용하고 strict SQL 모드를 사용해서 에러를 발생시킬 수 있음.

이하는 실제로 어떻게 동작이 다른지에 대해서 설명. -> non strict일때

- 표현 범위에서 벗어나는 숫자값의 경우, 가장 가까운 표현 가능한 숫자를 저장한다.
- 문자열은 빈 문자열이나, 저장가능한 가장 긴 값을 저장한다.
- 숫자로 시작하지 않는 문자열을 숫자 타입의 컬럼에 저장하는 경우, MySQL은 0을 저장.
- DATE, DATETIME의 유효성을 검증하지 않고, 애플리케이션에 위임함. MySQL이 봐도 명확히 잘못된 값은 특별한 "zero"값('0000-00-00
')이 저장됨.
- 단일 INSERT인 경우, NULL을 받지 않지만, 다중 INSERT나 `INSERT INTO ... SELECT`는 묵시적인 기본값(숫자라면 0, 문자열이라면 '' 등)
- INSERT가 값을 가지지 않고, 그 컬럼이 명시적인 값을 가지면 그걸 사용, 없다면 묵시적인 값을 사용.

이런 동작을 하는 건, SQL은 실행하기 전까지는 이러한 조건을 전부 체크할 수 없기 때문. 도중에 문제가 발생한다고 그냥 롤백하면 해결될 문제는 아님. 롤백을 지원하지 않는 엔진도 존재하기 때문에. 처리 도중에 종료하는 것도 좋은 아이디어는 아님. 이 경우에는 갱신 작업이 되다 만 상태이기 때문에 최악의 시나리오중 하나임. 이러한 경우에는 가능한 최선을 다해서 처리하는 것이 낫다.

다음처럼 STRICT모드를 활성화할 수 있음

```sql
SET sql_mode = 'STRICT_TRANS_TABLES';
//Transactional engine -> 어디서 문제가 발생하든 롤백
// None transactional engine -> 처음에 문제가 생기면 롤백, 아니라면 기존대로 처리
SET sql_mode = 'STRICT_ALL_TABLES';
// 어느쪽이든 칼같이 적용
```

`INSERT IGNORE` 나 `UPDATE IGNORE` 로 모드를 무시할 수 있음.