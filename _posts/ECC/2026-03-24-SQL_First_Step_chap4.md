---
title: "[SQL 첫걸음] 4장: 데이터의 추가, 삭제, 갱신"
date: 2026-03-24 14:00:00 +0900
categories: [ECC, Team-Project, SQL_First_Steps]
tags: [dev, study, sql]
---

["SQL 첫걸음" 도서 바로가기](https://www.hanbit.co.kr/store/books/look.php?p_code=B1374950226)


# 4장: 데이터의 추가, 삭제, 갱신

## 16강. 행 추가하기 - INSERT

### 1. INSERT로 행 추가하기

```sql
INSERT INTO 테이블명 VALUES(값1, 값2, ...)
```

### 2. 값을 저장할 열 지정하기

```sql
INSERT INTO 테이블명(열1, 열2, ...) VALUES(값1, 값2, ...)
```

### 3. NOT NULL 제약

- 테이블에 저장하는 데이터를 설정으로 제한하는 것을 통틀어 ‘제약’이라 부른다.
- NOT NULL 제약이 걸려있는 열은 NULL 값을 허용하지 않는다.

### 4. DEFAULT

: 명시적으로 값을 지정하지 않았을 경우 사용하는 초깃값

- 열을 지정하지 않으면 디폴트값으로 행이 추가된다.

---

## 17강. 삭제하기 - DELETE

### 1. DELETE로 행 삭제하기

```sql
DELETE FROM 테이블명 WHERE 조건식
```

- DELETE 명령은 WHERE 조건에 일치하는 ‘모든 행’을 삭제한다.

---

## 18강. 데이터 갱신하기 - UPDATE

### 1. UPDATE로 데이터 갱신하기

```sql
UPDATE 테이블명 SET 열=값 WHERE 조건식
```

- UPDATE 명령에서는 WHERE 조건에 일치하는 ‘모든 행’이 갱신된다.

### 2. 복수열 갱신

```sql
UPDATE 테이블명 SET 열1=값1, 열2=값2,... WHERE 조건식
```

### 3. NULL로 갱신하기

- `UPDATE sample41 SET a=NULL` 과 같이 갱신할 값으로 NULL 지정하면 된다.
- NULL로 값을 갱신하는 것을 보통 ‘NULL 초기화’라 부르기도 한다.

---

## 19강. 물리삭제와 논리삭제

### 1. 물리 삭제 (Hard Delete)

데이터베이스에서 데이터를 실제로 완전히 삭제하는 방식입니다.

- 방법: SQL의 `DELETE` 문을 사용합니다.SQL

  `DELETE FROM users WHERE id = 1;`

- 특징: * 저장 공간을 즉시 확보할 수 있다.
  - 삭제된 데이터는 복구가 매우 어렵다 (백업본 필요).
  - 개인정보 보호법 등 "완전 파기"가 필요한 경우 적합하다.

### 2. 논리 삭제 (Soft Delete)

데이터를 삭제하지 않고, 삭제되었다는 "표시"만 남기는 방식이다.

- 방법: `UPDATE` 문을 사용하여 특정 컬럼(예: `is_deleted`, `deleted_at`)의 값을 변경합니다.SQL
  - `- 삭제 플래그(flag)를 Y로 변경
    UPDATE users SET is_deleted = 'Y' WHERE id = 1;`
- 특징:
  - 데이터가 그대로 남아 있어 복구가 쉽다.
  - 데이터 분석이나 로그 기록 보존에 유리하다.
  - 주의: 조회 시 항상 `WHERE is_deleted = 'N'` 조건을 붙여야 실제 삭제된 데이터를 제외하고 볼 수 있다.
