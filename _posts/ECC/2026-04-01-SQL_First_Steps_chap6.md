---
title: "[SQL 첫걸음] 6장: 데이터베이스 객체 작성과 삭제"
date: 2026-04-01 14:00:00 +0900
categories: [ECC, Team-Project, SQL_First_Steps]
tags: [dev, study, sql]
---

["SQL 첫걸음" 도서 바로가기](https://www.hanbit.co.kr/store/books/look.php?p_code=B1374950226)

# 6장: 데이터베이스 객체 작성과 삭제

## 25강. 데이터베이스 객체

### 1. 데이터베이스 객체

- 데이터베이스 객체: 테이블이나 뷰, 인덱스 등 데이터베이스 내에 정의하는 모든 것을 일컫는 말이다.
- 객체의 이름을 붙일 때 제약 사항(명명규칙)
  - 기존 이름이나 예약어와 중복하지 않는다.
  - 숫자로 시작할 수 없다.
  - 언더스코어(_) 이외의 기호는 사용할 수 없다.
  - 한글을 사용할 때는 더블쿼트(MySQL에서는 백쿼트)로 둘러싼다.
  - 시스템이 허용하는 길이를 초과하지 않는다.

### 2. 스키마

- 충돌이 가능하지 않도록 하는 기능하는 그릇을 ‘네임스페이스(namespace)’라고 부르기도 한다.
- 스키마나 테이블은 네임스페이스이기도 하다.

---

## 26강. 테이블 작성 · 삭제 · 변경

### 1. 테이블 작성

- CREATE TABLE 명령

```sql
CREATE TABLE 테이블명 (열 정의1, 열 정의2, ...)
```

- 열 정의

```sql
열명 자료형 [DEFAULT 기본값] [NULL|NUT NULL]
```

- ex) CREATE TABLE로 테이블 작성하기:
  `CREATE TABLE sample62 (
    no INTGER NOT NULL,
    a VARCHAR(30),
    b DATE);`

### 2. 테이블 삭제

- DROP TABLE 명령

```sql
DROP TABLE 테이블명
```

- 데이터 행 삭제 → TRUNCATE TABLE 명령

```sql
TRUNCATE TABLE 테이블명
```

### 3. 테이블 변경

- ALTER TABLE 명령

```sql
ALTER TABLE 테이블명 변경명령
```

- ALTER TABLE로 할 수 있는 일
  - 열 추가, 삭제, 변경
  - 제약 추가, 삭제
- ADD 하부명령을 통해 열 추가

```sql
ALTER TABLE 테이블명 ADD 열 정의
```

- NOT NULL 제약이 걸린 열을 추가할 때는 기본값을 지정해야 한다.
- 열 속성 변경: MODIFY 하부명령

```sql
ALTER TABLE 테이블명 MODIFY 열 정의
```

- 열 이름 변경: CHANGE 하부명령

```sql
ALTER TABLE 테이블명 CHANGE [기존 열 이름][신규 열 정의]
```

- 열 삭제: DROP 하부명령

```sql
ALTER TABLE 테이블명 DROP 열명
```

---

## 27강. 제약

### 1. 테이블 작성시 제약 정의

- ex) 테이블 열에 제약 정의하기:
  `CREATE TABLE sample631 (
    a INTEGER NOT NULL,
    b INTEGER NOT NULL UNIQUE,
    c VARDHAR(30) );`
- 열 제약: 열에 대해 정의하는 제약
- 테이블 제약: ‘복수열에 의한 기본키 제약’처럼 한 개의 제약으로 복수의 열에 제약을 설명하는 경우

### 2. 제약 추가

- 열 제약 추가
  - ex) c열에 NOT NULL 제약 걸기: `ALTER TABLE sample631 MODIFY c VARCHAR(30) NOT NULL;`
- 테이블 제약 추가
  - ex) 기본키 제약 추가하기: `ALTER TABLE sample631 ADD CONSTRAINT pkey_sample631 PRIMARY KEY(a);`



### 3. 제약 삭제

- 열 제약 삭제하기
  - ex) c열의 NOT NULL 제약 없애기: `ALTER TABLE sample631 MODIFY c VARCHAR(30);`
- 테이블 제약 삭제하기
  - ex) pkey_sample631 제약 삭제하기: `ALTER TABLE sample631 DROP CONSTRAINT pkey_sample631 PRIMARY KEY(a);`
  - ex) 기본키 제약 삭제하기: `ALTER TABLE sample631 DROP PRIMARY KEY;`

---

## 28강. 인덱스 구조

### 1. 인덱스

- 인덱스의 역할: 검색속도 향상
- 인덱스는 테이블과는 별개로 독립된 데이터베이스 객체로 작성된다.
- 대부분의 데이터베이스에서는 테이블을 삭제하면 인덱스토 같이 삭제된다.

### 2. 검색에 사용하는 알고리즘

- 풀 테이블 스캔(full table scan)
  - 처리방법: 테이블에 저장된 모든 값을 처음부터 차례로 조사해나가는 것
- 이진 탐색(binary search)
  - 차례로 나열된 집합에 대해 유효한 검색방법
  - 처음부터 순서대로 조사하는 것이 아니라 집합을 반으로 나누어 조사
  - 대량의 데이터를 검색할 때는 이진 탐색이 빠르다.
- 이진 트리(binary tree)
  - 고속으로 검색할 수 있는 탐색 방법
  - 데이터가 미리 정렬되어 있어야 한다.

### 3. 유일성

- 이진 트리에는 중복하는 값을 등록할 수 없다.

---

## 29강. 인덱스 작성과 삭제

### 1. 인덱스 작성

- CREATE INDEX 명령

```sql
CREATE INDEX 인덱스명 ON 테이블명(열명1, 열명2, ...)
```

### 2. 인덱스 삭제

- DROP INDEX 명령
  - 스키마 객체의 경우

    ```sql
    DROP INDEX 인덱스명
    ```

  - 테이블 내 객체의 경우

    ```sql
    DROP INDEX 인덱스명 ON 테이블명
    ```


### 3. EXPLAIN

- EXPLAIN 명령

```sql
EXPLAIN SQL 명령
```

---

## 30강. 뷰 작성과 삭제

### 1. 뷰

- 뷰는 SELECT 명령을 기록하는 데이터베이스 객체다.
- 뷰를 작성하는 것으로 복잡한 SELECT 명령을 간략하게 표현할 수 있다.

### 2. 뷰 작성과 삭제

- CREATE VIEW로 뷰 작성

```sql
CREATE VIEW 뷰명 AS SELECT 명령
```

- CREATE VIEW에서 열 지정하기

```sql
CREATE VIEW 뷰명(열명1, 열명2, ...) AS SELECT 명령
```

- DROP VIEW로 뷰 삭제

```sql
DROP VIEW 뷰명
```

### 3. 뷰의 약점

- 뷰는 데이터베이스 객체로서 저장장치에 저장된다.
- 머티리얼라이즈드 뷰(Materialized View)
  - 뷰의 근원이 되는 테이블에 보관하는 데이터양이 많은 경우, 집계처리를 할 때도 뷰가 사용된다면 처리속도가 많이 떨어질 수밖에 없다.
  - 뷰를 중첩해서 사용하는 경우에도 처리 속도가 떨어지기 쉽다.
  - 위와 같은 상황을 회피하기 위해 사용할 수 있는 것이 머티리얼라이즈드 뷰이다.
  - 머티리얼라이즈드 뷰는 데이터를 일시적으로 저장해 사용하는 것이 아니라 테이블처럼 저장장치에 저장해두고 사용한다.
- 함수 테이블
  - 부모 쿼리와 어떤 식으로든 연관된 서브쿼리의 경우에는 뷰의 SELECT 명령으로 사용할 수 없다.
  - 대신 이 같은 뷰의 약점을 함수 테이블을 사용하여 회피할 수 있다.
  - 함수테이블: 테이블을 결괏값으로 반환해주는 사용자지정 함수이다.
