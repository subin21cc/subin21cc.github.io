---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 13장. 테스트 코드 작성하기"
date: 2026-07-03 12:00:00 +0900
categories: [ECC, Team-Project-BE, springboot3]
tags: [dev, study, java, spring]
render_with_liquid: false
---

["코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문" 도서 바로가기](https://www.gilbut.co.kr/book/view?bookcode=BN003778)

# 13장. 테스트 코드 작성하기

# 13.1 테스트란

### 테스트의 정의와 진화

테스트(test)란 프로그램의 품질을 검증하는 것으로, 의도대로 프로그램이 잘 동작하는지 확인하는 과정이다. 테스트 초창기에는 사람이 직접 요청을 보내고 응답을 받아 일일이 확인하는 직접 검증 방식으로 진행했다. 하지만 이제는 테스트 도구를 이용해 반복적인 검증 절차를 자동화할 수 있다. 이를 통해 다양한 문제를 미리 예방하고 코드 변경 등으로 인해 발생하는 부작용도 조기에 발견할 수 있다.

### 테스트 코드 작성 3단계

테스트 도구를 활용해 코드를 검증한다는 것은 테스트 코드(test code)를 작성해 실행한다는 의미이다. 테스트 코드는 보통 다음 3단계로 작성한다.

1. **예상 데이터 작성하기**
2. **실제 데이터 획득하기**
3. **예상 데이터와 실제 데이터 비교해 검증하기**

작성한 코드가 테스트를 통과하면 지속적인 리팩터링(refactoring)으로 코드를 개선한다. 그러나 테스트를 통과하지 못하면 잘못된 부분을 찾아 고치는 디버깅(debugging)을 해야 한다.

### 테스트 케이스와 TDD

테스트 코드는 다양한 경우를 대비해 작성하며, 이를 테스트 케이스(test case)라고 한다. 테스트 케이스는 성공할 경우뿐만 아니라 실패할 경우도 고려해야 하며, 다양한 상황을 예상해 세부적으로 작성해야 한다.

테스트 주도 개발(TDD, Test Driven Development)이란 일단 테스트 코드를 만든 후 이를 통과하는 최소한의 코드부터 시작해 점진적으로 코드를 개선 및 확장해 나가는 개발 방식이다.

- **RED**: 실패하는 테스트 코드를 추가한다.
- **GREEN**: 테스트 성공을 위한 최소한의 코드를 작성한다.
- **REFACTOR**: 테스트 통과를 유지하며 코드를 개선한다.

---

# 13.2 테스트 코드 작성하기

## 13.2.1 테스트 코드 기본 틀 만들기

ArticleService를 검증하는 테스트 코드의 기본 틀을 만드는 과정이다.

1. 프로젝트 탐색기에서 `service -> ArticleService`를 열고 테스트하고 싶은 `index()` 메서드에서 마우스 오른쪽 버튼을 누른 후 `Generate - Test`를 선택한다.
2. Create Test 창이 열리면 Testing library를 `JUnit5`로 선택하고, Member에서 `index():List<Article>`에 체크한 후 [OK] 버튼을 클릭한다.
3. `test/java/com/example/firstproject/service/ArticleServiceTest` 경로에 테스트 코드가 자동으로 생성된다. `main` 디렉터리와 동일한 구조로 만들어지며 해당 메서드가 테스트를 위한 코드임을 선언하는 `@Test` 어노테이션이 붙는다.
4. 자동 임포트 설정으로 인해 필요한 패키지가 사라지는 것을 방지하기 위해 `File -> Settings -> Editor -> General -> Auto Import`에서 `Optimize imports on the fly`를 체크 해제한다.
5. 테스트 코드를 스프링 부트와 연동하기 위해 클래스 위에 `@SpringBootTest`를 붙이고, 외부 객체를 주입받기 위해 `@Autowired`를 사용하여 `ArticleService` 객체를 선언한다.

```java
package com.example.firstproject.service;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest // 해당 클래스를 스프링 부트와 연동해 통합 테스트를 수행하겠다고 선언
class ArticleServiceTest{

    @Autowired
    ArticleService articleService; // articleService 객체 주입

    @Test
    void index(){
    }
}
```

![사진1](/assets/img/posts/springboot3/springboot3_chap13_1.png)

## 13.2.2 index() 테스트하기

모든 게시글을 조회 요청하는 `index()` 메서드를 테스트하는 과정이다.

1. **예상 데이터 작성하기**: `data.sql` 파일을 확인하여 자동으로 삽입되는 데이터 3개를 기반으로 예상 데이터 객체(a, b, c)를 생성하고 `Arrays.asList()`를 사용해 새 `ArrayList`인 `expected`에 저장한다. (이때 id는 Long 타입이므로 접미사 L을 붙인다)
2. **실제 데이터 획득하기**: `articleService.index()` 메서드를 호출해 그 결과를 `List<Article>` 타입의 `articles`에 받아온다.
3. **비교 및 검증**: `assertEquals(x, y)` 메서드를 이용해 예상 데이터와 실제 데이터를 각각 `toString()` 문자열로 변환하여 비교한다.

```java
@Test
void index(){
    // 1. 예상 데이터
    Article a = new Article(1L, "가가가가", "1111");
    Article b = new Article(2L, "나나나나", "2222");
    Article c = new Article(3L, "다다다다", "3333");
    List<Article> expected = new ArrayList<Article>(Arrays.asList(a, b, c));

    // 2. 실제 데이터
    List<Article> articles = articleService.index();

    // 3. 비교 및 검증
    assertEquals(expected.toString(), articles.toString());
}
```

- 테스트를 실행하여 정상 통과하면 예상 데이터와 실제 데이터가 일치했음을 의미한다. 만약 예상 데이터의 id를 틀리게 수정(예: 3L에서 4L)한 후 실행하면 테스트가 실패하며 Expected와 Actual이 다르다는 로그가 출력된다.

![사진2](/assets/img/posts/springboot3/springboot3_chap13_2.png)

## 13.2.3 show() 테스트하기

게시글을 조회하는 `show()` 메서드는 성공하는 경우와 실패하는 경우의 테스트 케이스로 나누어 작성한다.

#### 1. show_성공_존재하는_id_입력()

존재하는 id를 입력해 게시글 조회에 성공하는 상황을 테스트한다.

- **예상 데이터**: id가 1L인 게시물의 조회를 가정하여 expected 객체에 1번 게시물의 데이터를 저장한다.
- **실제 데이터**: `articleService.show(id)` 메서드를 호출하여 얻은 결과를 `article` 객체에 저장한다.
- **비교 및 검증**: `assertEquals()` 메서드로 두 객체의 `toString()` 값을 비교한다.

```java
@Test
void show_성공_존재하는_id_입력() {
    // 1. 예상 데이터
    Long id = 1L;
    Article expected = new Article(id, "가가가가", "1111");

    // 2. 실제 데이터
    Article article = articleService.show(id);

    // 3. 비교 및 검증
    assertEquals(expected.toString(), article.toString());
}
```

![사진3](/assets/img/posts/springboot3/springboot3_chap13_3.png)

#### 2. show_실패_존재하지_않는_id_입력()

존재하지 않는 id를 입력해 게시글 조회에 실패하는 상황을 테스트한다.

- **예상 데이터**: 존재하지 않는 id인 `-1L`을 조회하면 DB에서 조회되는 내용이 없어 `null`을 반환할 것이므로 `expected` 객체에 `null`을 저장한다.
- **실제 데이터**: `articleService.show(id)` 메서드를 호출하여 결과를 `article` 객체에 저장한다.
- **비교 및 검증**: 실제 데이터와 예상 데이터의 값 `null`은 `toString()` 메서드를 호출할 수 없으므로 전달값에 문자열 변환 없이 `expected`와 `article`을 그대로 사용해 비교한다.

```java
@Test
void show_실패_존재하지_않는_id_입력() {
    // 1. 예상 데이터
    Long id = -1L;
    Article expected = null;

    // 2. 실제 데이터
    Article article = articleService.show(id);

    // 3. 비교 및 검증
    assertEquals(expected, article);
}
```

![사진4](/assets/img/posts/springboot3/springboot3_chap13_4.png)

## 13.2.4 create() 테스트하기

게시글을 생성하는 `create()` 메서드 역시 성공하는 경우와 실패하는 경우로 나누어 테스트 케이스를 작성한다.

#### 1. create_성공_title과_content만_있는_dto_입력()

가장 흔하게 생각할 수 있는 title과 content만 있는 dto를 입력한 경우이다.

- **예상 데이터**: id는 DB에서 자동으로 생성하므로 생성자에는 `null`을 넣고 title과 content 필드만 선언하여 `dto` 객체를 생성한다. `expected` 객체에는 자동으로 생성될 값인 id `4L`을 써서 예상 데이터를 저장한다.
- **실제 데이터**: `articleService.create(dto)` 메서드를 호출해서 얻은 결과를 `article` 객체에 저장한다.
- **비교 및 검증**: `assertEquals()` 메서드를 이용해 `toString()` 형태로 비교한다.

```java
@Test
void create_성공_title과_content만_있는_dto_입력() {
    // 1. 예상 데이터
    String title = "라라라라";
    String content = "4444";
    ArticleForm dto = new ArticleForm(null, title, content);
    Article expected = new Article(4L, title, content);

    // 2. 실제 데이터
    Article article = articleService.create(dto);

    // 3. 비교 및 검증
    assertEquals(expected.toString(), article.toString());
}
```

![사진5](/assets/img/posts/springboot3/springboot3_chap13_5.png)

#### 2. create_실패_id가_포함된_dto_입력()

새 게시물을 생성할 때 id를 입력하여 오류가 발생하는 상황을 테스트한다.

- **예상 데이터**: id를 넣고 게시글을 생성하면 오류가 나고 `null`이 반환되므로 `expected` 객체에 `null`을 저장한다.
- **실제 데이터**: `articleService.create(dto)` 메서드를 호출하여 결과를 `article` 객체에 저장한다.
- **비교 및 검증**: `null` 값은 `toString()` 메서드를 호출할 수 없으므로 `expected`와 `article` 객체 자체를 전달하여 비교한다.

```java
@Test
void create_실패_id가_포함된_dto_입력() {
    // 1. 예상 데이터
    Long id = 4L;
    String title = "라라라라";
    String content = "4444";
    ArticleForm dto = new ArticleForm(id, title, content);
    Article expected = null;

    // 2. 실제 데이터
    Article article = articleService.create(dto);

    // 3. 비교 및 검증
    assertEquals(expected, article);
}
```

![사진6](/assets/img/posts/springboot3/springboot3_chap13_6.png)

## 13.2.5 여러 테스트 케이스 한 번에 실행하기

#### index() 테스트 실패 원인 파악

ArticleServiceTest 클래스 자체를 실행하면 작성한 5개의 테스트 케이스를 한꺼번에 돌릴 수 있다. 그러나 한 번에 실행할 경우 단독 실행 시 통과했던 `index()` 테스트가 실패하는 현상이 발생한다.

실패 로그를 보면 예상 데이터에는 3개의 데이터가 있지만, 실제 데이터에는 4개의 데이터가 존재하는 것으로 나타난다. 이는 앞서 실행된 `create_성공_title과_content만_있는_dto_입력()` 메서드가 실행되면서 insert 문을 통해 DB에 새 데이터를 추가했으나, 테스트 종료 후 데이터가 롤백(rollback)되지 않았기 때문에 발생한 현상이다.

#### 해결 방법 및 원칙

문제를 해결하려면 테스트 메서드를 트랜잭션으로 처리하여 테스트가 끝난 후 변경된 데이터를 처음으로 되돌려야 한다.

- **원칙**: 데이터를 조회(Read)하는 테스트를 제외하고, 데이터를 생성(Create), 수정(Update), 삭제(Delete)하는 테스트를 할 때는 반드시 해당 테스트를 트랜잭션으로 묶어 테스트가 종료된 후 원래대로 돌아갈 수 있게 롤백 처리해 줘야 한다.

따라서 외부 DB 데이터에 변경을 주는 `create_성공` 및 `create_실패` 테스트 메서드 위에 각각 `@Transactional` 어노테이션을 추가한다.

```java
@Test
@Transactional // 테스트가 끝난 후 변경된 데이터를 처음으로 되돌린다
void create_성공_title과_content만_있는_dto_입력() {
    // (생략)
}

@Test
@Transactional
void create_실패_id가_포함된_dto_입력() {
    // (생략)
}
```

`@Transactional`을 설정한 후 클래스 전체를 다시 테스트하면, 데이터 생성 테스트가 끝난 후 자동으로 롤백되기 때문에 이후에 실행되는 `index()` 테스트도 정상적으로 통과한다.

