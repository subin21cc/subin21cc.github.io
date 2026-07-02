---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 12장. 서비스 계층과 트랜잭션"
date: 2026-07-01 12:00:00 +0900
categories: [ECC, Team-Project-BE, springboot3]
tags: [dev, study, java, spring]
render_with_liquid: false
---

["코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문" 도서 바로가기](https://www.gilbut.co.kr/book/view?bookcode=BN003778)

# 12장. 서비스 계층과 트랜잭션

# 12.1 서비스와 트랜잭션의 개념

### 서비스(Service)의 정의

- **역할**: 컨트롤러(Controller)와 리파지터리(Repository) 사이에 위치하는 계층이다.
- **기능**: 서버의 핵심 기능인 비즈니스 로직을 처리하는 순서를 총괄한다.
- **필요성**: 컨트롤러가 클라이언트의 요청·응답 처리와 리파지터리 명령을 동시에 수행하는 비효율을 줄이고, 역할 분업을 명확히 하기 위해 서비스 계층을 둔다.

### 트랜잭션(Transaction)의 정의

- **의미**: 모두 성공해야만 하는 일련의 과정을 뜻한다.
- **특징**: 더 이상 쪼갤 수 없는 업무 처리의 최소 단위이다. 과정 중 일부가 실패하면 데이터의 일관성을 위해 이전까지 진행된 모든 기록은 취소되어야 한다.
- **롤백(Rollback)**: 트랜잭션 수행 중 실패가 발생했을 때, 진행 초기 단계의 상태로 데이터를 되돌리는 작업을 의미한다.

---

# 12.2 서비스 계층 만들기

기존 REST 컨트롤러(`ArticleApiController`)의 CRUD 로직을 컨트롤러, 서비스, 리파지터리로 분업화하기 위해 다음 단계를 수행한다.

1. `ArticleApiController` 내의 기존 CRUD 메서드 코드를 모두 주석 처리한다.
2. 컨트롤러 내부의 리파지터리 객체 주입 코드를 제거하고, 서비스 객체를 주입하도록 수정한다.

    ```java
    @Autowired
    private ArticleService articleService;
    ```

3. `com.example.firstproject.service` 패키지를 새로 생성한다.
4. 해당 패키지 내부에 `ArticleService` 클래스를 생성한다.
5. `ArticleService` 클래스 상단에 `@Service` 어노테이션을 부착한다. 이 어노테이션은 해당 클래스를 서비스로 인식하여 스프링 부트가 서비스 객체를 생성하고 관리하게 만든다.
6. `ArticleService` 내부에 `ArticleRepository` 필드를 선언하고 `@Autowired`를 통해 리파지터리 객체를 주입한다.

![사진1](/assets/img/posts/springboot3/springboot3_chap12_1.png)

## 12.2.1 게시글 조회 요청 개선하기

### 모든 게시글 조회 요청 개선

- **컨트롤러**: `@GetMapping("/api/articles")` 요청 시 `articleRepository.findAll()` 대신 `articleService.index()`를 호출하고 결과를 반환하도록 변경한다.
- **서비스**: `ArticleService`에 `index()` 메서드를 구현하고, 내부에서 `articleRepository.findAll()`을 호출하여 DB 조회 결과를 반환한다.

### 단일 게시글 조회 요청 개선

- **컨트롤러**: `@GetMapping("/api/articles/{id}")` 요청 시 `articleService.show(id)`를 호출하도록 변경한다.
- **서비스**: `ArticleService`에 `show(Long id)` 메서드를 구현하고, `articleRepository.findById(id).orElse(null)`을 실행하여 데이터를 조회한 뒤 반환한다.

![사진2](/assets/img/posts/springboot3/springboot3_chap12_2.png)

## 12.2.2 게시글 생성 요청 개선하기

- **컨트롤러**: `@PostMapping("/api/articles")` 요청 시 `articleService.create(dto)`를 호출하도록 변경한다. 반환형은 `ResponseEntity<Article>`로 지정한다.
- **응답 처리**: 삼항 연산자를 사용하여 생성 결과가 존재하면 `HttpStatus.OK` 상태와 데이터를 응답하고, 실패하여 `null`이 반환되면 `HttpStatus.BAD_REQUEST`를 응답한다.
- **서비스 구현**: `ArticleService`에 `create(ArticleForm dto)` 메서드를 생성하고 DTO를 엔티티로 변환한 뒤 `articleRepository.save(article)`을 통해 데이터를 저장한다.
- **기존 데이터 덮어쓰기 문제 해결**: POST 요청 본문에 이미 존재하는 데이터의 `id`가 포함되어 있으면 기존 데이터가 수정되는 문제가 발생한다. 이를 방지하기 위해 서비스의 `create()` 메서드 내부에 조건문을 추가하여 `article.getId() != null`인 경우 `null`을 반환하도록 예외 처리를 진행한다.

![사진3](/assets/img/posts/springboot3/springboot3_chap12_3.png)

## 12.2.3 게시글 수정 요청 개선하기

- **컨트롤러**: `@PatchMapping("/api/articles/{id}")` 메서드 내의 복잡한 로직을 서비스로 위임한다. 컨트롤러는 `articleService.update(id, dto)`를 호출한 뒤 결과의 유무에 따라 정상 응답(200 OK) 혹은 오류 응답(400 Bad Request)만을 수행한다.
- **서비스 구현**: `ArticleService` 클래스 상단에 로깅을 위한 `@Slf4j` 어노테이션을 추가하고 `update(Long id, ArticleForm dto)` 메서드를 구현한다.
  1. DTO를 수정용 엔티티로 변환한다.
  2. `articleRepository.findById(id).orElse(null)`을 사용하여 수정 대상이 되는 타깃 엔티티를 조회한다.
  3. 타깃 엔티티가 존재하지 않거나, 요청된 `id`와 엔티티의 `id`가 일치하지 않는 잘못된 요청일 경우 로그를 남기고 `null`을 반환한다.
  4. 정상적인 요청인 경우 타깃 엔티티에 변경 사항을 반영(`target.patch(article)`)하고, `articleRepository.save(target)`을 실행하여 DB를 업데이트한 후 최종 데이터를 반환한다.

![사진4](/assets/img/posts/springboot3/springboot3_chap12_4.png)

## 12.2.4 게시글 삭제 요청 개선하기

- **컨트롤러**: `@DeleteMapping("/api/articles/{id}")` 메서드에서 `articleService.delete(id)`를 호출한다. 삭제 완료 시 `HttpStatus.NO_CONTENT`를 반환하고, 실패 시 `HttpStatus.BAD_REQUEST`를 반환하도록 흐름을 단순화한다.
- **서비스 구현**: `ArticleService`에 `delete(Long id)` 메서드를 구현한다.
  1. `articleRepository.findById(id).orElse(null)`로 삭제 대상을 조회한다.
  2. 대상이 없으면 `null`을 반환하여 컨트롤러가 오류 응답을 처리하도록 한다.
  3. 대상이 존재하면 `articleRepository.delete(target)`를 통해 DB에서 삭제를 수행하고, 삭제된 대상을 반환한다.

![사진5](/assets/img/posts/springboot3/springboot3_chap12_5.png)

---

# 12.3 트랜잭션 맛보기

여러 데이터를 동시에 생성하는 과정에서 의도적으로 예외를 발생시켜 트랜잭션과 롤백의 작동 여부를 검증한다.

### 컨트롤러 구현

- `@PostMapping("/api/transaction-test")` 요청을 처리하는 `transactionTest()` 메서드를 추가한다.
- 여러 개의 DTO를 `List<ArticleForm>` 형태로 수신하며, 실제 생성 작업은 `articleService.createArticles(dtos)`를 호출하여 수행한다.

### 서비스 구현 (`createArticles`)

1. **DTO 변환**: 스트림(Stream) 문법을 사용하여 DTO 리스트를 엔티티 리스트로 매핑하고 변환 결과를 리스트로 수집한다.
2. **DB 저장**: 변환된 엔티티 리스트를 스트림 순회하며 `articleRepository.save(article)`를 호출해 순차적으로 DB에 저장한다.
3. **강제 예외 발생**: 유효하지 않은 음수 ID를 조회하도록 `articleRepository.findById(-1L).orElseThrow(() -> new IllegalArgumentException("결제 실패!"))` 코드를 작성하여 의도적으로 예외를 유발한다.
4. **결과 반환**: 최종 엔티티 리스트를 반환한다.

### 트랜잭션 미적용 시의 결과

- `@Transactional` 어노테이션이 없는 상태에서 요청을 전송하면 서버 내부 에러(500 Error)가 발생한다.
- 콘솔 로그 확인 결과 예외가 발생하기 전에 실행된 3번의 `insert` 쿼리가 DB에 그대로 반영되어 에러가 발생했음에도 데이터가 생성되는 문제가 발생한다.

### 트랜잭션 적용 및 검증

- `ArticleService`의 `createArticles()` 메서드 상단에 `@Transactional` 어노테이션(`org.springframework.transaction.annotation.Transactional`)을 추가한다.
- 어노테이션을 적용한 후 동일한 에러 발생 요청을 보내면 이전과 같이 500 에러가 발생하지만, 데이터베이스를 조회했을 때 데이터가 추가되지 않고 실행 전의 초기 상태로 안전하게 롤백된 것을 확인할 수 있다.

![사진6](/assets/img/posts/springboot3/springboot3_chap12_6.png)
