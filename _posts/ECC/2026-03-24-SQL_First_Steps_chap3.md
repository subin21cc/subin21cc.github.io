---
title: "[SQL 첫걸음] 3장: 정렬과 연산"
date: 2026-03-24 14:00:00 +0900
categories: [ECC, Team-Project, SQL_First_Steps]
tags: [dev, study, sql]
---

["SQL 첫걸음" 도서 바로가기](https://www.hanbit.co.kr/store/books/look.php?p_code=B1374950226)

# 3장: 정렬과 연산

## 9강. 정렬 - ORDER BY

SELECT명령의 ORDER BY구 사용하면 검색결과의 행 순서 변경 가능

```sql
SELECT 열명 FROM 테이블명 WHERE 조건식 ORDER BY 열명
```

### 1. ORDER BY로 검색 결과 정렬하기

```sql
SELECT 열명 FROM 테이블명 WHERE 조건식 ORDER BY 열명
SELECT 열명 FROM 테이블명 ORDER BY 열명
```

### 2. ORDER BY DESC로 내림차순으로 정렬하기

```sql
SELECT 열명 FROM 테이블명 ORDER BY 열명 DESC
```

```sql
SELECT 열명 FROM 테이블명 ORDER BY 열명 ASC
```

ORDER BY의 기본 정렬방법은 오름차순

### 3. 대소관계

- 문자열형 데이터의 대소관계는 사전식 순서에 의해 결정됨
- 수치형과 문자열형 데이터는 대소관계의 계산 방법이 다름
-

### 4. ORDER BY는 테이블에 영향을 주지 않는다

- SELECT 명령은 데이터를 검색하는 명령. 이는 테이블의 데이터를 참조만 할 뿐이며 변경은 하지 않는다.

---

## 10강. 복수의 열을 지정해 정렬하기

### 1. 복수 열로 정렬 지정

- ORDER BY로 복수 열 지정하기

```sql
SELECT 열명 FROM 테이블명 ORDER BY 열명1, 열명2...
```

### 2. 정렬방법 지정하기

```sql
SELECT 열명 FROM 테이블명 
						ORDER BY 열명1 [ASC|DESC], 열명2[ASC|DESC]...
```

### 3. NULL 값의 정렬순서

- DB 종류에 따라 다르지만, 보통 '무한대' 혹은 '가장 작은 값'으로 취급된다
- DB별 차이 (오름차순 기준)
  - NULL을 가장 큰 값으로 취급 (맨 뒤로): Oracle, PostgreSQL
  - NULL을 가장 작은 값으로 취급 (맨 앞으로): MySQL, SQL Server

---

## 11강. 결과 행 제한하기 - LIMIT

### 1. 행수 제한

```sql
SELECT 열명 FROM 테이블명 WHERE 조건식 ORDER BY 열명 LIMIT 행수
```

- LIMIT 구로 반환될 행수를 제한할 수 있다.

### 2. 오프셋 지정

```sql
SELECT 열명 FROM 테이블명 LIMIT 행수 OFFSET 위치
```

---

## 12강. 수치 연산

### 1. 사칙 연산

- 연산자의 우선순위
  - 우선순위 1: * / %
  - 우선순위2: + -
  - 우선순위 같다면, 왼쪽에서 오른쪽으로 계산

### 2. SELECT 구로 연산하기

```sql
SELECT 식1, 식2... FROM 테이블명
```

### 3. 열의 별명

- 예약어 AS를 사용해 지정한다.
- SELECT 구에서는 콤마(,)로 구분해 복수의 식을 지정할 수 있으며 각각의 식에 별명을 붙일 수 있다.
- 이름에 ASCII 문자 이외의 것을 포함할 경우는 더블쿼트로 둘러싸서 지정한다.
- 이름을 지정하는 경우 숫자로 시작되지 않도록 한다.

### 4. WHERE 구에서 연산하기

```sql
SELECT *, price * quantity AS amount FROM sample34 
						WHERE price * quantity >= 2000;
```

- WHERE구와  SELECT구의 내부처리 순서
  - WHERE 구 → SELECT 구
  - SELECT 구에서 지정한 별명은 WHERE 구 안에서 사용할 수 없다.

### 5. NULL 값의 연산

- NULL로 연산하면 결과는 NULL이 된다.

### 6. ORDER BY 구에서 연산하기

```sql
SELECT *, price * quantity AS amount FROM sample34 ORDER BY price * quantity DESC;
```

- ORDER BY 구에서는 SELECT 구에서 지정한 별명을 사용할 수 있다.

### 7. 함수

```sql
함수명 (인수1, 인수2...)
```

- 함수도 연산자도 표기 방법이 다를 뿐, 같은 것이다.

### 8. ROUND 함수

: 반올림하는 함수

```sql
SELECT amount, ROUND(amount) FROM sample341;
```

- 반올림 자릿수 지정

```sql
SELECT amount, ROUND(amount, 1) FROM sample341;
```

---

## 13강. 문자열 연산

### 1. 문자열 결합

| 연산자/함수 | 연산 | 데이터베이스 |
| --- | --- | --- |
| + | 문자열 결합 | SQL Server |
| || | 문자열 결합 | Oracle, DB2, PostgreSQL |
| CONCAT | 문자열 결합 | MySQL |

## 2. SUBSTRING 함수

: 문자열의 일부분을 계산해서 반환해주는 함수

- 예) 앞 4자리(년) 추출
  - SUBSTRING(’20140125001’, 1, 4) → ‘2014’
- 예) 5째 자리부터 2자리(월) 추출
  - SUBSTRING(’20140125001’, 5, 2) → ‘01’

### 3. TRIM 함수

: 문자열 앞뒤로 여분의 스페이스가 있을 경우 이를 제거해주는 함수

- 문자열 도중에 존재하는 스페이스는 제거되지 않는다.
- 고정길이 문자열형에 대해 많이 사용하는 함수이다.
- TRIM으로 스페이스 제거하기
  - TRIM(’ABC   ‘) → ‘ABC”

### 4. CHARACTER_LENGTH 함수

: 문자열의 길이를 계산해 돌려주는 함수

- OCTET_LENGTH함수: 문자열의 길이를 바이트 단위로 계산해 돌려주는 함수

---

## 14강. 날짜 연산

### 1. SQL에서의 날짜

- 시스템 날짜
  - CURRENT_TIMSTAMP 함수로 시스템 날짜 확인
- 날짜 서식
  - TO_DATE(’3014/01/25’, ‘YYYY/MM/DD’)

### 2. 날짜의 덧셈과 뺄셈

- MySQL에서는 DATEDIFF(’2014-02-28’, ‘2014-01-01’)로 계산 가능

---

## 15강. CASE문으로 데이터 변환하기

### 1. CASE 문

```sql
CASE WHEN 조건식1 THEN 식1
[WHEN 조건식2 THEN 식2 ...]
[ELSE 식3]
END
```

### 2. 또 하나의 CASE 문

- 검색 CASE
  - `CASE WHEN 조건식 THEN 식…`
- 단순 CASE
  - `CASE WHEN 식 THEN 식…`

### 3. CASE를 사용할 경우 주의사항

- ELSE 생략
  - ELSE를 생략하면 ELSE NULl이 되는 것에 주의할 것.
  - CASE문의 ELSE는 생략하지 않는 편이 낫다.
