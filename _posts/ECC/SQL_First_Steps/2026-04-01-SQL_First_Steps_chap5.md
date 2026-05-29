---
title: "[SQL 첫걸음] 5장: 집계와 서브쿼리"
date: 2026-04-01 13:00:00 +0900
categories: [ECC, Team-Project, SQL_First_Steps]
tags: [dev, study, sql]
---

["SQL 첫걸음" 도서 바로가기](https://www.hanbit.co.kr/store/books/look.php?p_code=B1374950226)

# 5장: 집계와 서브쿼리

## 20강. 행 개수 구하기 - COUNT

### 1. COUNT로 행 개수 구하기

- 일반적인 함수는 인수로 하나의 값을 지정하는 반면 집계함수는 인수로 집합을 지정한다.

```sql
COUNT(집합)
```

- ex) sample51의 행 개수 구하기: `SELECT COUNT(*) FROM sample51;`
- COUNT 집계함수로 행 개수를 구할 수 있다.

### 2. 집계함수와 NULL 값

- 집계함수는 집합 안에 NULL 값이 있을 경우 무시한다.

### 3. DISTINCT로  중복 제거

- DISTINCT로 중복값을 제거할 수 있다.
- 예약어로 열명이 아니다.
- 생략할 경우에는 ALL로 간주된다.
- ex) `SELECT DISTINCT name FROM sample51;`

### 3. 집계함수에서 DISTINCT

- COUNT 함수, DISTINCT, WHERE 구의 조건을 지정해 구할 수 있을까? 할 수 없다.
  ⇒ WHERE구에서는 검색할 조건을 지정하는 것밖에 할 수 없다.

---

## 21강. COUNT 이외의 집계함수

### 1. SUM으로 합계 구하기

- SUM 집계함수를 사용해 집합의 합계를 구할 수 있다.
- 지정되는 집합은 수치형 뿐이다. 문자열형이나 날짜시간형의 집합에서 합계를 구할 수 는 없다.
- COUNT와 마찬가지로 NULL 값을 무시한다. NULL 값을 제거한 뒤에 합계를 낸다.
- ex) SUM으로 quantitiy열의 합계 구하기: `SELECT SUM(quantity) FROM sample51;`

### 2. AVG로 평균내기

- AVG 집계함수로 집합의 평균값을 구할 수 있다.
- 지정되는 집합은 수치형 뿐이다.

### 3. MIN · MAX로 최솟값 · 최댓값 구하기

- 문자열과 날짜시간형에도 사용할 수 있다.
- NULL 값을 무시한다.
- ex) `SELECT MIN(quantity), MAX(quantity) FROM sample51;`

---

## 22강. 그룹화 - GROUP BY

```sql
SELECT * FROM 테이블명 GROUP BY 열1, 열2, ...
```

### 1. GROUP BY로 그룹화

- GROUP BY 구에 열을 지정하여 그룹화하면 지정된 열의 값이 같은 행이 하나의 그룹으로 묶인다.
- WHERE 구에서는 집계함수를 사용할 수 없다.

### 2. HAVING 구로 조건 지정

- HAVING 구를 사용하면 집계함수를 사용해서 조건식을 지정할 수 있다.
- ex) HAVING구로 걸러내기:
  `SELECT name, COUNT(name) FROM sample51 
  GROUP BY name HAVING COUNT(name) = 1;`
- 내부처리 순서
  - WHERE 구 → GROUP BY 구 → HAVING 구 → SELECT 구 → ORDER BY 구

### 3. 복수열의 그룹화

- GROUP BY에서 지정한 열 이외의 열은 집계함수를 사용하지 않은 채 SELECT 구에 지정할 수 없다.

---

## 23강. 서브쿼리

- 서브쿼리: SELECT 명령에 의한 데이터 질의로, 상부가 아닌 하부의 부수적인 질의를 의미한다.

### 1. DELETE의 WHERE 구에서 서브쿼리 사용하기

- ex) 괄호로 서브쿼리를 지정해 삭제: `DELETE FROM sample54 WHERE a = (SELECT MIN(a) FROM sample54);`

### 2. 스칼라 값

- SELECT 명령이 하나의 값만 반환하는 것을 ‘스칼라 값을 반환한다’고 한다.
- = 연산자를 사용하여 비교할 경우에는 스칼라 값끼리 비교할 필요가 있다.
- ex) 서브쿼리의 패턴
  - 패턴 1) 하나의 값을 반환하는 패턴: `SELECT MIN(a) FROM sample54;`
  - 패턴 2) 복수의 행이 반환되지만 열은 하나인 패턴: `SELECT no FROM sample54;`
  - 패턴 3) 하나의 행이 반환되지만 열이 복수인 패턴: `SELECT MIN(a), MAX(no) FROM sample54;`
  - 패턴 4) 복수의 행, 복수의 열이 반환되는 패턴: `SELECT no, a FROM sample54;`

### 3. SELECT 구에서 서브쿼리 사용하기

- ex)
  - `SELECT
     (SELECT COUNT(*) FROM sample51) AS sq1,
     (SELECT COUNT(*) FROM sample54) AS sq2;`

### 4. SET 구에서 서브쿼리 사용하기

- ex) `UPDATE sample54 SET a = (SELECT MAX(a) FROM sample54);`

### 5. FROM 구에서 서브쿼리 사용하기

- ex) `SELECT * FROM (SELECT * FROM sample54) sq;`

### 6. INSERT 명령과 서브쿼리

- INSERT 명령의 사용방법 두 가지
  - VALUES 구의 일부로 서브쿼리를 사용하기
  - VALUES 구 대신 SELECT 명령을 사용하기
- INSERT SELECT: SELECT 명령의 결과를 INSERT INTO 로 지정한 테이블에 전부 추가한다. 이 때문에 데이터의 복사나 이동을 할 때 자주 사용하는 명령이다.

---

## 24강. 상관 서브쿼리

### 1. EXISTS

```sql
EXISTS (SELECT 명령)
```

- ex) EXISTS를 사용해 ‘있음’으로 갱신하기
  `UPDATE sample551 SET a = ‘있음’ WHERE
   EXISTS (SELECT * FROM sample552 WHERE no2 = no);`

### 2. NOT EXISTS

- ex) NOT EXISTS를 사용해 ‘없음’으로 갱신하기
  `UPDATE sample551 SET a = ‘없음’ WHERE
   NOT EXISTS (SELECT * FROM sample552 WHERE no2 = no);`

### 3. 상관 서브쿼리

- 상관 서브쿼리에서는 부모 명령과 연관되어 처리되기 때문에 서브퀄 부분만을 따로 떼어내어 실행시킬 수 없다.

### 4. IN

- IN을 사용하면 집합 안의 값이 존재하는지 조사할 수 있다

```sql
열명 IN(집합)
```

- IN은 NULL값을 비교할 수 없다.
