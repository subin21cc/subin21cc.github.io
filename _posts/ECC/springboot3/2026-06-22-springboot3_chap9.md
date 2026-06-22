---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 9장. CRUD와 SQL 쿼리 종합"
date: 2026-06-22 12:00:00 +0900
categories: [ECC, Team-Project-BE, springboot3]
tags: [dev, study, java, spring]
render_with_liquid: false
---

["코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문" 도서 바로가기](https://www.gilbut.co.kr/book/view?bookcode=BN003778)

# 9장. CRUD와 SQL 쿼리 종합

서버에서 데이터의 생성, 조회, 수정, 삭제(CRUD) 등을 요청하면 JPA의 리파지터리가 데이터베이스(DB)에 해당 요청을 전달한다. 요청을 받은 데이터베이스는 자신의 언어인 SQL로 쿼리를 작성해 테이블 속 데이터를 관리한다. 쿼리(Query)란 데이터베이스에 정보를 요청하는 구문을 말하며, 데이터 생성은 `INSERT`, 조희는 `SELECT`, 수정은 `UPDATE`, 삭제는 `DELETE` 문을 사용한다.

# 9.1 JPA로깅 설정하기

로깅(Logging)이란 시스템이 작동할 때 당시의 상태와 작동 정보를 기록하는 것을 말한다. JPA의 로깅 설정은 `src > main > resources > application.properties` 파일에서 수행한다.

1. **`application.properties` 파일 열기 및 로깅 레벨 설정**파일 하단에 `logging.level.org.hibernate.SQL=DEBUG` 코드를 추가하여 하이버네이트가 생성하는 실행 SQL 쿼리를 출력하도록 설정한다. 로깅 레벨을 `DEBUG`로 지정하면 응용 프로그램을 디버깅하는 데 필요한 세부 정보가 기록된다.
2. **서버 실행 및 SQL 쿼리 로그 확인**서버를 구동한 후 인텔리제이의 실행창 콘솔에서 `org.hibernate.SQL` 문구와 함께 데이터베이스 테이블을 제어하는 `drop table`, `create table` 등의 SQL 쿼리가 출력되는지 확인한다.
3. **쿼리 줄바꿈 정렬 설정 추가**한 줄로 길게 출력되는 SQL 쿼리의 가독성을 높이기 위해 `spring.jpa.properties.hibernate.format_sql=true` 코드를 추가하여 줄바꿈 정렬을 적용한다 .
4. **서버 재시작 및 줄바꿈 결과 확인**서버를 재시작하고 실행창의 로그를 조회하여 SQL 쿼리가 가독성 있게 줄바꿈되어 출력되는지 확인한다.
5. **매개변수 값 확인 코드 추가**SQL 쿼리 로그 내 물음표(`?`)로 표시되는 부분에 전달되는 실제 매개변수 값을 확인하기 위해 `logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE` 코드를 추가한다 .
6. **서버 재시작 및 동작 확인**서버를 다시 시작하여 시스템이 정상적으로 가동되는지 확인한다.
7. **H2 데이터베이스 JDBC URL 고정 설정**매번 가동 시마다 임의로 변경되는 H2 데이터베이스의 JDBC 주소를 고정하기 위해 유니크 이름 생성을 해제하고 고정 URL 경로를 지정하는 코드를 추가한다 .
8. **서버 재시작 및 고정 URL 확인**서버를 재시작한 후 콘솔 로그 창에서 설정한 고정 주소(`jdbc:h2:mem:testdb`)로 데이터베이스가 준비되었는지 확인한다.
9. **H2 콘솔 접속 및 확인**웹 브라우저에서 `localhost:8080/h2-console` 페이지에 접속하여 설정한 고정 URL을 입력한 뒤 [Connect] 버튼을 클릭하여 데이터베이스 세션에 정상적으로 연결되는지 확인한다.

---

# 9.2 SQL 쿼리 로그 확인하기

JPA 로깅 설정을 완료하면 서버의 CRUD 동작에 따라 내부에 어떤 SQL 쿼리가 작동하는지 로그로 추적할 수 있다.

## 9.2.1 데이터 생성 시: INSERT 문

- **id 자동 생성 전략 추가하기:** 서버 재시작 시 데이터베이스 초기화 파일(`data.sql`)에 명시된 더미 데이터의 id 값과, 서버 내에서 새로 생성하려는 데이터의 id 값이 충돌하여 `Unique index or primary key violation`(기본키 중복 위반 예외) 에러가 발생한다. 테이블 내 각 데이터를 유일하게 구분하도록 지정한 속성을 기본키(Primary Key)라고 부른다.
- 주입하려는 데이터의 id 중복 문제를 해결하기 위해 `Article.java` 엔티티 파일의 id 멤버 변수 위에 `@GeneratedValue(strategy = GenerationType.IDENTITY)` 어노테이션을 부착하여 데이터베이스가 시스템적으로 식별 키를 자동 발급하도록 전략을 수정한다.
- 데이터베이스가 id를 주체적으로 자동 생성하게 되었으므로, `resources/data.sql` 파일 내부에 선언되어 있던 수동 id 속성과 값(1, 2, 3)을 모두 삭제하여 정돈한다.
- **데이터 생성 시 동작하는 SQL 로그 확인하기:** `localhost:8080/articles/new` 경로에서 새로운 데이터를 입력하고 제출하면, 콘솔창에 default 키워드 및 매개변수 바인딩 값이 매핑된 `INSERT INTO article` 문이 동작하는 것을 로그로 확인할 수 있다.

## 9.2.2 데이터 조회 시: SELECT 문

- **전체 데이터 조회:** 목록 페이지(`localhost:8080/articles`)에 접속하여 모든 게시글을 가져올 때, 조회할 속성을 명시한 `SELECT` 절과 대상 테이블을 지정하는 `FROM article` 구문으로 구성된 SQL 문이 수행된다.
- **단일 데이터 조회:** 특정 게시글 상세 페이지에 접근하여 한 건의 데이터만 조회할 때는 전체 조회 쿼리 후미에 가변 경로 변수 id 값이 바인딩되는 선별 조건절인 `WHERE id = ?`이 추가되어 실행된다.

## 9.2.3 데이터 수정 시: UPDATE 문

- 수정 페이지에서 데이터를 수정한 후 제출하면 `UPDATE` 쿼리가 수행된다. article 테이블에서 대상 id를 찾아 content와 title 값을 전달받은 매개변수로 수정하도록 요청하는 `UPDATE article SET content=?, title=? WHERE id=?` 문이 로그에 기록된다.

## 9.2.4 데이터 삭제 시: DELETE 문

- 상세 페이지에서 특정 데이터를 파기하기 위해 삭제 버튼을 클릭하면 `DELETE` 쿼리가 수행된다. 전체 데이터 유실을 방지하고 선택한 대상 행만 영속성 레이어에서 제거하기 위해 조건절이 결합된 `DELETE FROM article WHERE id=?` 문이 작동한다.

![사진1](/assets/img/posts/springboot3/springboot3_chap9_1.png)

---

# 9.3 기본 SQL 쿼리 작성하기

## 9.3.1 coffee 테이블 만들기

새로운 테이블인 `coffee`를 가상으로 생성하고, H2 콘솔 데이터베이스 편집기 도구 내에서 원천 SQL 쿼리를 직접 작성하여 가동하는 실습 과정이다. SQL 구문의 끝에는 항상 문장을 완결 짓는 세미콜론(`;`)을 붙여야 한다.

![사진2](/assets/img/posts/springboot3/springboot3_chap9_2.png)

## 9.3.2 coffee 테이블 생성하기

테이블에 데이터를 주입하고 적재할 때는 `INSERT 문`을 사용한다.

- 단일 행 데이터를 생성하는 구문은 다음과 같다.

    ```sql
    INSERT INTO coffee (id, name, price) VALUES (1, '아메리카노', 4100);
    ```

- 대량의 연속 데이터를 한꺼번에 다중 적재하고자 할 때는 VALUES 절 이하의 데이터 세트를 쉼표(`,`) 기호로 연결하여 일괄 실행한다.

    ```sql
    INSERT INTO coffee (id, name, price)
    VALUES
        (2, '라떼', 4600),
        (3, '모카', 5100),
        (4, '오늘의 커피', 3800);
    ```

![사진3](/assets/img/posts/springboot3/springboot3_chap9_3.png)

## 9.3.3 coffee 테이블 조회하기

테이블에 생성한 데이터를 조회할 때는 `SELECT 문`을 사용한다.

- id 변수가 3번에 해당하는 특정 데이터만 선별 조희하고자 할 때는 조건 필터 명령어 구조인 `WHERE id = 3`을 결합한다.

    ```
    SELECT id, name, price FROM coffee WHERE id = 3;
    ```

- 조건절 요소를 거두어내고 테이블 내에 보관 중인 모든 데이터를 조건 없이 인출할 때는 다음과 같이 조회한다.

    ```
    SELECT id, name, price FROM coffee;
    ```


## 9.3.4 coffee 테이블 수정하기

테이블에 저장된 데이터를 수정할 때는 `UPDATE 문`을 사용한다 . id가 4번인 오늘의 커피 가격을 9,900원으로 변경 수정하는 제어 질의문은 다음과 같다.

```sql
UPDATE coffee SET price = 9900 WHERE id = 4;
```

![사진4](/assets/img/posts/springboot3/springboot3_chap9_4.png)

## 9.3.5 coffee 테이블 삭제하기

테이블 내부의 특정 데이터 행 레코드를 파기 소멸시키고자 할 때는 `DELETE 문`을 사용한다. 전체 데이터 오염을 예방하기 위해 조건절 명세를 부착하여 id가 4번에 매칭되는 대상 레코드만 파기하는 질의 구문은 다음과 같다.

```sql
DELETE FROM coffee WHERE id = 4;
```
