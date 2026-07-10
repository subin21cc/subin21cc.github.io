---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 15장. 댓글 컨트롤러와 서비스 만들기"
date: 2026-07-07 20:00:00 +0900
categories: [ECC, Team-Project-BE, springboot3]
tags: [dev, study, java, spring]
render_with_liquid: false
---

["코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문" 도서 바로가기](https://www.gilbut.co.kr/book/view?bookcode=BN003778)

# 15장. 댓글 컨트롤러와 서비스 만들기

# 15.1 댓글 REST API의 개요

댓글 기능을 완성하기 위해 REST 컨트롤러, 서비스, DTO를 구현하며 시스템 구조 및 각 컴포넌트의 역할은 다음과 같다.

### 1. 주요 컴포넌트의 역할

- **REST 컨트롤러 (`CommentApiController`)**
  - 댓글 REST API를 위한 컨트롤러이다.
  - 서비스와 협업하며 클라이언트의 요청을 받아 응답한다.
  - 뷰(View)가 아닌 JSON 형태의 데이터를 반환한다.
- **서비스 (`CommentService`)**
  - REST 컨트롤러와 리파지터리 사이에서 비즈니스 로직(처리 흐름)을 담당한다.
  - 예외 상황이 발생했을 때 데이터의 일관성을 보장하기 위해 `@Transactional`을 통해 변경 사항을 롤백한다.
- **DTO (`CommentDto`)**
  - 사용자에게 보여줄 댓글 정보를 담는 객체이다.
  - 단순히 클라이언트와 서버 간에 댓글 JSON 데이터를 전송하기 위해 사용된다.
- **엔티티 (`Comment`)**
  - DB 데이터를 담는 자바 객체이다.
  - 엔티티를 기반으로 테이블이 생성되며, 리파지터리가 DB 속 데이터를 조회하거나 전달할 때 사용된다.
- **리파지터리 (`CommentRepository`)**
  - 엔티티를 관리하는 인터페이스로 데이터 CRUD 기능을 제공한다.
  - 서비스로부터 명령을 받아 DB에 쿼리를 전송하고 응답을 받는다.

### 2. 댓글 CRUD를 위한 REST API 주소 설계

- **댓글 조회**: `GET` `/api/articles/{articleId}/comments` (특정 게시글의 모든 댓글 조회)
- **댓글 생성**: `POST` `/api/articles/{articleId}/comments` (특정 게시글에 댓글 생성)
- **댓글 수정**: `PATCH` `/api/comments/{id}` (특정 댓글 수정)
- **댓글 삭제**: `DELETE` `/api/comments/{id}` (특정 댓글 삭제)

### 3. 완성될 프로젝트 구조 (관련 클래스 목록)

- **`api` 패키지**: `CommentApiController` (REST 컨트롤러)
- **`dto` 패키지**: `CommentDto` (데이터 전송 객체)
- **`entity` 패키지**: `Comment` (댓글 엔티티)
- **`repository` 패키지**: `CommentRepository` (JPA 인터페이스)
- **`service` 패키지**: `CommentService` (트랜잭션 관리 및 처리 흐름 담당)

---

# 15.2 댓글 컨트롤러와 서비스 틀 만들기

댓글 REST API를 구현하기 위해 `api` 패키지에 REST 컨트롤러 클래스를 만들고, `service` 패키지에 서비스 클래스를 만들어 상호 의존성을 주입한다.

### 1. 댓글 컨트롤러 생성 (`CommentApiController.java`)

`CommentApiController` 클래스를 생성한 뒤 `@RestController` 어노테이션을 부여하여 선언하고, `@Autowired` 어노테이션을 통해 `CommentService` 객체를 주입한다.

```java
package com.example.firstproject.api;

import com.example.firstproject.service.CommentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class CommentApiController{
    @Autowired
    private CommentService commentService;

    // 1. 댓글 조회
    // 2. 댓글 생성
    // 3. 댓글 수정
    // 4. 댓글 삭제
}
```

![사진1](/assets/img/posts/springboot3/springboot3_chap15_1.png)

### 2. 댓글 서비스 생성 (`CommentService.java`)

`CommentService` 클래스를 생성하고 `@Service` 어노테이션을 부착한다. 비즈니스 로직 연산을 처리하기 위해 댓글 리파지터리(`CommentRepository`)와 게시글 리파지터리(`ArticleRepository`) 객체를 `@Autowired`로 주입한다.

- **게시글 리파지터리가 필요한 이유**: 댓글을 생성할 때, 해당 댓글의 대상이 되는 부모 게시글(`Article`)이 실제로 존재하는지 여부를 확인해야 하기 때문이다.

```java
package com.example.firstproject.service;

import com.example.firstproject.repository.ArticleRepository;
import com.example.firstproject.repository.CommentRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class CommentService{
    @Autowired
    private CommentRepository commentRepository;
    @Autowired
    private ArticleRepository articleRepository;
}
```

![사진2](/assets/img/posts/springboot3/springboot3_chap15_2.png)

---

# 15.3 댓글 조회하기

특정 게시글에 종속된 모든 댓글을 가져오는 기능을 구현한다.

## 15.3.1 요청을 받아 응답할 컨트롤러 만들기

`GET` 메서드 형태로 `/api/articles/{articleId}/comments` 주소의 요청을 받아 응답할 핸들러 메서드를 구현한다.

- **반환형 정의**: `ResponseEntity<List<CommentDto>>` 형태로 반환한다. DB에서 조회한 엔티티 목록(`List<Comment>`)을 보안 및 계층 간 분리를 위하여 DTO 목록(`List<CommentDto>`)으로 변환하여 보내야 하며, HTTP 응답 상태 코드를 함께 제어하기 위함이다.
- **동작 설계**: 컨트롤러는 요청을 받으면 서비스에 작업을 위임하고, 결과를 받아 성공 상태(`HttpStatus.OK`)와 함께 바디에 담아 응답한다. 예외 처리는 스프링 부트 구조에 위임하므로 삼항 연산자 없이 성공 상태만을 반환하도록 코드를 단순화한다.

```java
// CommentApiController.java 내 추가
@GetMapping("/api/articles/{articleId}/comments")
public ResponseEntity<List<CommentDto>> comments(@PathVariable Long articleId) {
    // 서비스에 위임
    List<CommentDto> dtos = commentService.comments(articleId);
    // 결과 응답
    return ResponseEntity.status(HttpStatus.OK).body(dtos);
}
```

### CommentDto 클래스 생성하기

클라이언트와 서버 간 데이터를 전송하기 위한 데이터 구조체이다. 롬복 어노테이션을 활용하여 보일러플레이트 코드를 줄이고 정적 팩토리 메서드 구현을 통해 객체 생성 역할을 캡슐화한다.

```java
package com.example.firstproject.dto;

import com.example.firstproject.entity.Comment;
import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

@AllArgsConstructor
@NoArgsConstructor
@Getter
@ToString
public class CommentDto{
    private Long id; // 댓글 고유 id

    @JsonProperty("article_id")
    private Long articleId; // 댓글의 부모 게시글 id

    private String nickname; // 댓글 작성자
    private String body; // 댓글 본문

    // 댓글 엔티티를 입력받아 DTO로 변환하여 생성하는 정적 메서드
    public static CommentDto createCommentDto(Comment comment){
        return new CommentDto(
            comment.getId(),
            comment.getArticle().getId(),
            comment.getNickname(),
            comment.getBody()
        );
    }
}
```

## 15.3.2 요청을 처리할 서비스 만들기

조회 요청 비즈니스 로직을 처리하는 메서드이다. 처음에는 기본 `for` 반복문 형태로 구성한 뒤, 자바 스트림(Stream) 문법을 도입하여 가독성을 개선하고 구조를 간소화한다.

#### 기존 반복문 형태 코드

```java
public List<CommentDto> comments(Long articleId){
    // 1. 댓글 조회
    List<Comment> comments = commentRepository.findByArticleId(articleId);
    // 2. 엔티티 -> DTO 변환
    List<CommentDto> dtos = new ArrayList<CommentDto>();
    for (int i = 0; i < comments.size(); i++) {
        Comment c = comments.get(i);
        CommentDto dto = CommentDto.createCommentDto(c);
        dtos.add(dto);
    }
    // 3. 결과 반환
    return dtos;
}
```

#### 자바 스트림(Stream)으로 개선한 코드

데이터 묶음을 순차적으로 처리할 때 중간 연산(`map`)과 최종 연산(`collect`)을 활용하여 가독성을 극대화한다. 원본 데이터를 변경하지 않고 선언형으로 변환 연산을 완료한다.

```java
// CommentService.java 내 최종 반영
public List<CommentDto> comments(Long articleId){
    return commentRepository.findByArticleId(articleId) // 댓글 엔티티 목록 조회
            .stream() // 댓글 엔티티 목록을 스트림으로 변환
            .map(CommentDto::createCommentDto) // 엔티티를 DTO로 매핑 (메서드 레퍼런스 활용)
            .collect(Collectors.toList()); // 스트림 데이터를 리스트 자료형으로 변환 및 반환
}
```

## 15.3.3 결과 확인하기

![사진3](/assets/img/posts/springboot3/springboot3_chap15_3.png)

---

# 15.4 댓글 생성하기

게시글 하단에 신규 댓글 데이터를 추가하여 데이터베이스에 등록하는 기능이다.

## 15.4.1 요청을 받아 응답할 컨트롤러 만들기

`POST` 메서드로 `/api/articles/{articleId}/comments` 요청을 수신하며, 본문에 담긴 JSON 형태의 데이터를 DTO 객체로 안전하게 역직렬화하기 위해 `@RequestBody` 어노테이션을 적용한다.

```java
// CommentApiController.java 내 추가
@PostMapping("/api/articles/{articleId}/comments")
public ResponseEntity<CommentDto> create(@PathVariable Long articleId,
                                         @RequestBody CommentDto dto){
    // 서비스에 위임
    CommentDto createdDto = commentService.create(articleId, dto);
    // 결과 응답
    return ResponseEntity.status(HttpStatus.OK).body(createdDto);
}
```

## 15.4.2 요청을 처리할 서비스 만들기

해당 비즈니스 로직은 데이터베이스의 상태를 변경하는 작업이 수반된다. 따라서 트랜잭션 관리와 일관성 유지를 위하여 메서드 레벨에 `@Transactional` 어노테이션을 선언한다.

```java
// CommentService.java 내 추가
@Transactional
public CommentDto create(Long articleId, CommentDto dto){
    // 1. DB에서 부모 게시글 조회 및 예외 발생 처리
    Article article = articleRepository.findById(articleId)
            .orElseThrow(() -> new IllegalArgumentException("댓글 생성 실패! 대상 게시글이 없습니다."));

    // 2. 부모 게시글 엔티티와 DTO를 기반으로 새 댓글 엔티티 생성
    Comment comment = Comment.createComment(dto, article);

    // 3. 생성된 엔티티를 데이터베이스에 영속화
    Comment created = commentRepository.save(comment);

    // 4. DB에 영속화한 엔티티를 DTO로 재변환하여 반환
    return CommentDto.createCommentDto(created);
}
```

### 엔티티 내부 생성 메서드 추가 (`Comment.java`)

엔티티 클래스 내에 정적 생성 메서드를 추가하고 데이터 생성 전 유효성을 직접 검증한다.

- **검증 규칙**: 클라이언트 측에서 `id` 값을 명시하여 전송했거나(자동 생성되어야 하므로 생략되어야 함), 요청 URL의 부모 게시글 id와 데이터 바디 내부의 부모 게시글 id가 일치하지 않는 경우 유효하지 않은 잘못된 요청으로 판단하고 `IllegalArgumentException`을 던진다.

```java
// Comment.java 내 추가 및 검증 로직 구현
public static Comment createComment(CommentDto dto, Article article){
    // 예외 검증 1: 생성 시 id가 이미 주입되어 있는 경우 예외 발생
    if (dto.getId() != null) {
        throw new IllegalArgumentException("댓글 생성 실패! 댓글의 id가 없어야 합니다.");
    }
    // 예외 검증 2: DTO 내 게시글 id와 실제 부모 게시글 id가 다른 경우 예외 발생
    // 안전한 Long 비교를 위하여 Objects.equals 사용
    if (!Objects.equals(dto.getArticleId(), article.getId())) {
        throw new IllegalArgumentException("댓글 생성 실패! 게시글의 id가 잘못됐습니다.");
    }
    return new Comment(dto.getId(), article, dto.getNickname(), dto.getBody());
}
```

> **JsonProperty 개념**: JSON 데이터의 키명과 매핑하려는 DTO 필드의 필드명이 다를 경우, 해당 자바 필드 선언문 위에 `@JsonProperty("키_이름")`을 작성해 주어야 오류 없이 안정적으로 매핑되어 역직렬화된다.
>

## 15.4.3 결과 확인하기

![사진4](/assets/img/posts/springboot3/springboot3_chap15_4.png)
![사진5](/assets/img/posts/springboot3/springboot3_chap15_5.png)

---

# 15.5 댓글 수정하기

존재하는 댓글에 대해 가변 데이터를 수정하여 이를 데이터베이스에 반영하는 기능이다.

## 15.5.1 요청을 받아 응답할 컨트롤러 만들기

`PATCH` 요청 방식으로 `/api/comments/{id}` 주소를 처리하는 핸들러를 구성한다. `id`는 수정 타겟이 되는 댓글 엔티티의 고유 식별값이다.

```java
// CommentApiController.java 내 추가
@PatchMapping("/api/comments/{id}")
public ResponseEntity<CommentDto> update(@PathVariable Long id,
                                         @RequestBody CommentDto dto){
    // 서비스에 위임
    CommentDto updatedDto = commentService.update(id, dto);
    // 결과 응답
    return ResponseEntity.status(HttpStatus.OK).body(updatedDto);
}
```

## 15.5.2 요청을 처리할 서비스 만들기

대상 엔티티를 조회하고, 값을 덮어쓴 후 반영 작업을 트랜잭션 단위로 묶어 처리한다.

```java
// CommentService.java 내 추가
@Transactional
public CommentDto update(Long id, CommentDto dto){
    // 1. DB에 해당 댓글을 조회하고 없는 경우 예외 처리 진행
    Comment target = commentRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("댓글 수정 실패! 대상 댓글이 없습니다."));

    // 2. 조회된 기존 댓글 내용 수정 (패치 로직 위임)
    target.patch(dto);

    // 3. 수정한 댓글 엔티티를 데이터베이스에 갱신 처리
    Comment updated = commentRepository.save(target);

    // 4. DB에 반영된 엔티티를 DTO로 변환하여 반환
    return CommentDto.createCommentDto(updated);
}
```

### 엔티티 내에 패치(patch) 메서드 작성 (`Comment.java`)

수정 데이터 DTO를 활용하여 기존 엔티티의 정보를 직접 변경하는 도메인 로직이다.

- **검증 규칙**: 수정하려는 대상 댓글의 id와 요청으로 들어온 JSON 데이터 내부의 id 값이 다른 경우 예외를 발생시킨다. `Long` Wrapper 객체 주소 값 비교 버그 방지를 위해 `Objects.equals`를 통해 내부 데이터를 엄격히 검증한다.
- **선택적 수정**: 닉네임과 본문 중 클라이언트가 수정을 위해 값을 명시하여 실어 보낸 값(`!= null`)만 선택적으로 엔티티에 반영하여 기존 내용을 갱신한다.

```java
// Comment.java 내 추가
public void patch(CommentDto dto){
    // 예외 검증: 수정 요청 바디의 id와 현재 엔티티 인스턴스의 id가 상이할 시 불일치 예외 처리
    if (dto.getId() != null && !Objects.equals(this.id, dto.getId())) {
        throw new IllegalArgumentException("댓글 수정 실패! 잘못된 id가 입력됐습니다.");
    }
    // 데이터 변경 사항 갱신 적용
    if (dto.getNickname() != null) {
        this.nickname = dto.getNickname();
    }
    if (dto.getBody() != null) {
        this.body = dto.getBody();
    }
}
```

## 15.5.3 결과 확인하기

![사진6](/assets/img/posts/springboot3/springboot3_chap15_6.png)

---

# 15.6 댓글 삭제하기

존재하는 댓글 데이터를 특정하여 데이터베이스 목록에서 영구적으로 삭제하는 기능이다.

## 15.6.1 요청을 받아 응답할 컨트롤러 만들기

`DELETE` 메서드로 수신하여 `/api/comments/{id}` 경로를 통해 들어온 삭제 식별 대상 ID를 추출하여 서비스를 실행한다.

- **오타 수정 내역**: 기존 도서 혹은 원고에 기재된 `HttpStatus.GK` 상태 코드는 스프링 프레임워크에 존재하지 않는 잘못 작성된 오타이므로 정상적인 성공 응답값인 `HttpStatus.OK` 상태 코드로 변경하여 안정적인 삭제 응답 상태 처리를 수행하게 한다.

```java
// CommentApiController.java 내 추가
@DeleteMapping("/api/comments/{id}")
public ResponseEntity<CommentDto> delete(@PathVariable Long id){
    // 서비스에 위임
    CommentDto deletedDto = commentService.delete(id);
    // 결과 응답
    return ResponseEntity.status(HttpStatus.OK).body(deletedDto);
}
```

## 15.6.2 요청을 처리할 서비스 만들기

대상 엔티티를 식별하여 삭제하고, 삭제된 흔적 데이터를 전달받을 수 있도록 응답을 구성한다.

```java
// CommentService.java 내 추가
@Transactional
public CommentDto delete(Long id){
    // 1. DB에서 해당 댓글 데이터를 조회하고 존재하지 않을 시 예외 처리
    Comment target = commentRepository.findById(id)
            .orElseThrow(() -> new IllegalArgumentException("댓글 삭제 실패! 대상이 없습니다."));

    // 2. 조회에 성공한 실제 타겟 엔티티를 JPA 영속성 관리 하에서 제거
    commentRepository.delete(target);

    // 3. 삭제 완료된 기존 엔티티를 클라이언트에게 전송할 수 있게 DTO로 복원 및 반환
    return CommentDto.createCommentDto(target);
}
```

## 15.6.3 결과 확인하기

![사진7](/assets/img/posts/springboot3/springboot3_chap15_7.png)
