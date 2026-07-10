---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 16장. 웹 페이지에서 댓글 목록 보기"
date: 2026-07-08 20:00:00 +0900
categories: [ECC, Team-Project-BE, springboot3]
tags: [dev, study, java, spring]
render_with_liquid: false
---

["코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문" 도서 바로가기](https://www.gilbut.co.kr/book/view?bookcode=BN003778)

# 16장. 웹 페이지에서 댓글 목록 보기

# 16.1 댓글 보기의 개요

15장에서 댓글 CRUD를 위한 서버의 여러 구성 요소를 만들었다. 이 장에서는 서버에서 처리한 댓글을 사용자가 볼 수 있도록 뷰 페이지를 만들고 출력해 보겠다.

보통 게시판의 특정 글을 클릭하면 상세 페이지(show)가 뜨면서 그 아래에 댓글(comments)이 보인다. 댓글 영역은 크게 두 가지로 나뉜다. 기존 댓글을 보여 주는 영역(list)과 새 댓글을 입력하는 영역(new)이다. 두 영역은 별도의 뷰 페이지를 만들어 상세 페이지 아래에 삽입하는 형태로 구현한다.

이 장에서는 기존 댓글을 보여 주는 것(list)까지 구현하고, 다음 장에서는 새 댓글을 입력하는 것(new)을 구현하겠다.

---

# 16.2 댓글 뷰 페이지 삽입하기

게시글 5번의 상세 페이지에 접속해 보겠다. 서버를 실행한 후 웹 브라우저에서 `http://localhost:8080/articles/5` 페이지에 접속하면 상세 페이지 화면이 나온다. 이 화면 아래에 댓글을 출력해 보겠다.

1. 프로젝트 탐색기의 controller 패키지에 있는 `ArticleController`를 연다. 이 컨트롤러가 상세 페이지를 보여 달라는 요청을 받아 처리한다. 여기에서 `show()` 메서드가 `/articles/{id}`로 접속했을 때 보여 주는 페이지를 반환하고 있다. `return` 문을 보면 반환하는 뷰 페이지가 articles 디렉터리의 `show` mustache 파일이다.

```java
// controller/ArticleController.java
public class ArticleController{
    // (중략)
    @GetMapping("/articles/{id}")
    public String show(@PathVariable Long id, Model model){
        // (중략)
        return "articles/show"; // 이 메서드가 반환하는 뷰 페이지 확인
    }
}
```

2. `resources/templates/articles/show.mustache` 파일을 연다. 바로 이 파일이 특정 게시글의 상세 페이지이다. 댓글은 페이지 맨 밑에 넣기로 했으니 푸터(footer) 바로 위에 댓글 뷰 파일을 삽입한다(`{{>comments/_comments}}`). 이렇게 하면 comments 디렉터리에 `_comments.mustache` 파일을 연결해 상세 페이지에서 댓글이 보이게 된다.

```html
<a href="/articles">Go to Article List</a>
{{>comments/_comments}} {{>layouts/footer}}
```

3. 망치 아이콘을 클릭해 빌드하고 웹 페이지를 새로 고침한다. 아직 `_comments.mustache` 파일을 만들지 않아 에러 페이지(Whitelabel Error Page, status 500)가 뜬다.
4. 이어서 `_comments.mustache` 파일을 만들어 보겠다. 프로젝트 탐색기의 `templates`에서 마우스 오른쪽 버튼을 누르고 **New → Directory**를 선택한다. 새 디렉터리 이름은 `comments`라고 짓는다. 디렉터리가 생성되면 `comments`에서 다시 마우스 오른쪽 버튼을 누르고 **New → File**을 클릭한다. 파일 이름은 `_comments.mustache`라고 짓는다.
5. 망치 아이콘을 다시 클릭하고 웹 페이지를 새로 고침하면 에러 페이지가 나오지 않는다. 하지만 댓글은 보이지 않는다.
6. 댓글을 볼 수 있게 `_comments.mustache` 파일에 코드를 작성해 보겠다. `_comments.mustache` 파일로 와서 `<div></div>` 태그로 레이아웃을 잡고 그 안쪽을 2개 영역으로 나누어 뷰 파일을 삽입한다. `_list.mustache` 파일은 댓글 목록을 보여 주고, `_new.mustache` 파일은 새 댓글을 작성한다. 두 파일 모두 comments 디렉터리에 존재한다.

```html
<div>
    {{>comments/_list}}
    {{>comments/_new}}
</div>
```

> **TIP**
>
>
> `<div>` 태그는 웹 페이지의 레이아웃(전체적인 틀)을 만들 때 사용한다. 웹 페이지의 영역을 논리적으로 구분한다고 생각하면 된다. `<div>` 태그를 사용하면 각 공간에 구성 요소를 배치하고 CSS를 활용해 스타일을 적용할 수 있다.
>
7. 망치 아이콘을 다시 클릭하고 새로 고침하면 에러 페이지가 뜬다. `_list.mustache` 파일과 `_new.mustache` 파일이 없기 때문이다.
8. 두 파일을 만들어 보겠다. 프로젝트 탐색기의 `comments` 디렉터리에서 마우스 오른쪽 버튼을 누르고 **New → File**을 클릭한다. 이름은 `_list.mustache`로 짓는다. 같은 방법으로 `comments` 디렉터리 아래에 `_new.mustache` 파일도 만든다.
9. `_list.mustache` 파일로 가서 댓글 목록 뷰 페이지를 만들어 보겠다.
- `<div></div>` 태그로 댓글 목록 전체를 보여 주는 영역을 만들고 id를 `"comments-list"`로 설정한다.
- 조회한 댓글 목록에서 댓글을 하나씩 꺼내 순회할 수 있도록 `{{#commentDtos}}`…`{{/commentDtos}}`를 추가한다. `commentDtos`가 여러 데이터라면 머스테치 문법 안쪽에 있는 내용을 반복하라는 뜻이다.
- `<div></div>` 태그로 댓글 하나를 보여 주는 영역을 만든다. 이때 `class="card"` 속성과 `id="comments-{{id}}"` 속성도 추가한다. `class="card"` 속성은 이 영역을 카드 구조로 만들라는 뜻이고, `id="comments-{{id}}"` 속성은 반복되는 `commentDtos`에 있는 id 값을 삽입해 이 영역의 id를 comments-1, comments-2, comments-3 등으로 설정하라는 뜻이다.
- `<div></div>` 태그로 댓글 내에서 헤더 영역(`class="card-header"`)을 만든다.
- `<div></div>` 태그로 댓글 내에서 본문 영역(`class="card-body"`)을 만든다.

```html
<div id="comments-list">
  {{#commentDtos}}
    <div class="card" id="comments-{{id}}">
      <div class="card-header">
      </div>
      <div class="card-body">
      </div>
    </div>
  {{/commentDtos}}
</div>
```

> **Note** 부트스트랩의 카드 요소 사용하기
웹 페이지를 만들 때 미리 짠 코드를 사용하면 편리하다. 앞의 코드도 부트스트랩에서 제공하는 카드(card) 요소를 사용했다. 방법은 다음과 같다.
1. `https://getbootstrap.com`에 접속한다. 이 책에서는 부트스트랩 v5.0.2를 사용했으므로 오른쪽 버전 선택 목록에서 v5.0.2를 선택한다. 화면이 바뀌면 왼쪽 상단의 검색창에서 card를 검색한다.
2. 다양한 카드 형태가 나오는데 아래로 내려 댓글 보기 용도로 쓸 만한 Header and footer 카드를 찾는다.
3. 코드에서 필요 없는 내용은 지우고 card-header와 card-body만 가져와 사용한다.
>
10. 댓글을 보여 줄 구조를 만들었으니 내용을 적는다.
- 카드의 헤더에서는 `commentDtos`에 담겨 있는 `{{nickname}}`을 보여 준다.
- 카드의 본문에서는 `commentDtos`에 담겨 있는 `{{body}}`를 보여 준다.

```html
<div class="card-header">
  {{nickname}} </div>
<div class="card-body">
  {{body}} </div>
```

11. 망치 아이콘을 클릭하고 5번 게시글의 상세 페이지를 새로 고침한다. 에러 페이지가 뜨진 않지만, 아무런 댓글도 보이지 않는다. 뷰 페이지에서 사용하는 `commentDtos`를 모델에 등록하지 않았기 때문이다. MVC 패턴에 따르면 사용자가 볼 화면(View)을 컨트롤러(Controller)가 반환한다. 이때 화면에 필요한 데이터는 모델(Model)에 등록해야 했다. 모델에 데이터를 등록한 후 5번 상세 페이지에 댓글 목록이 보이는지 확인해 보겠다.

![사진1](/assets/img/posts/springboot3/springboot3_chap16_1.png)

---

# 16.3 댓글 목록 가져오기

뷰 페이지에서 사용할 변수는 반드시 모델에 등록해야 사용할 수 있다. 댓글 목록을 모델에 등록하고 이를 화면에 출력해 보겠다.

1. `ArticleController`로 간다. `show()` 메서드를 보면 게시글(article)을 조회해 뷰 페이지를 반환하는 코드가 작성돼 있다. 이 흐름으로 댓글 목록을 가져와 반환하겠다. 먼저 `CommentService`의 `comments(id)` 메서드를 호출해 조회한 댓글 목록을 `List<CommentDto>` 타입의 `commentDtos` 변수에 저장한다.

```java
// controller/ArticleController.java
public String show(@PathVariable Long id, Model model){
    log.info("id = " + id);

    // 1. id를 조회해 데이터 가져오기
    Article articleEntity = articleRepository.findById(id).orElse(null);
    List<CommentDto> commentsDtos = commentService.comments(id);

    // 2. 모델에 데이터 등록하기
    model.addAttribute("article", articleEntity);

    // 3. 뷰 페이지 설정하기
    return "articles/show";
}
```

그런데 `CommentService`가 빨간색으로 표시된다. `CommentService`를 찾을 수 없기 때문이다.

2. 위로 올라가서 `CommentService`를 객체 주입한다. 그러면 이 컨트롤러에서 `CommentService`를 사용할 수 있게 된다.

```java
// controller/ArticleController.java
public class ArticleController{
    @Autowired
    private ArticleRepository articleRepository;

    @Autowired
    private CommentService commentService; // 서비스 객체 주입
    // (중략)
}
```

3. 받아 온 댓글 목록(`commentDtos`)을 모델에 등록한다. 모델에서 데이터를 등록할 때는 `addAttribute()` 메서드를 사용한다.

> **형식** `model.addAttribute("변수명", 변수값)` // 변수값을 "변수명"이라는 이름으로 추가

```java
// controller/ArticleController.java
public String show(@PathVariable Long id, Model model){
    // (중략)
    // 2. 모델에 데이터 등록하기
    model.addAttribute("article", articleEntity);
    model.addAttribute("commentDtos", commentsDtos); // 댓글 목록 모델에 등록

    return "articles/show";
}
```

4. 서버를 재시작하고 5번 게시글의 상세 페이지(`http://localhost:8080/articles/5`)를 새로 고침한다. 댓글 3개가 잘 보인다. 그런데 댓글이 너무 붙어 있어 보기 좋지 않다. 수정해 보겠다.
5. `_list.mustache` 파일로 가서 카드 요소의 마진(margin), 즉 바깥 여백을 2만큼 준다. 마진은 CSS 속성이지만 부트스트랩의 `m-1`, `m-2`, `m-*` 클래스로 간편히 적용할 수 있다.

```html
{{#commentDtos}}
<div class="card m-2" id="comments-{{id}}">
  </div>
{{/commentDtos}}
```

6. 망치 아이콘을 클릭하고 새로 고침하면 댓글 사이 간격이 보기 좋게 벌어진다.

![사진2](/assets/img/posts/springboot3/springboot3_chap16_2.png)
