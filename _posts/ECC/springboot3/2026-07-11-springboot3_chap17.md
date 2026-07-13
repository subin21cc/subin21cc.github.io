---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 17장. 웹 페이지에서 댓글 등록하기"
date: 2026-07-11 20:00:00 +0900
categories: [ECC, Team-Project-BE, springboot3]
tags: [dev, study, java, spring]
render_with_liquid: false
---

["코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문" 도서 바로가기](https://www.gilbut.co.kr/book/view?bookcode=BN003778)

# 17장. 웹 페이지에서 댓글 등록하기

# 17.1 댓글 등록의 개요

16장에서 댓글 목록을 게시글 아래에 출력했으나 새 댓글을 등록하는 부분은 아직 구현하지 않았다. 이 장에서는 새 댓글을 등록하기 위한 뷰 페이지를 만들고 자바스크립트 코드로 REST API를 호출해 본다.

그동안 REST API 요청을 보낼 때 Talend API Tester를 이용했으나, 실제 게시판에서는 해당 웹 페이지에서 바로 요청을 보낸다. 이를 위해 사용하는 자바스크립트 API는 다음과 같다.

- **document.querySelector()**: 웹 페이지에서 특정 요소(버튼)를 찾아 반환한다.
- **addEventListener()**: 특정 요소에 이벤트가 발생(버튼 클릭)했을 때 특정 동작(댓글 객체 전달)을 수행한다.
- **fetch()**: 웹 페이지에서 REST API 요청(POST 요청)을 보낸다.

실습은 다음 2단계로 진행한다.

1. 댓글 생성 뷰 페이지(new)에 댓글 입력 폼 만들기
2. [댓글 작성] 버튼을 클릭해 REST API 요청 보내기

---

# 17.2 댓글 생성 뷰 페이지 만들기

16장에서 댓글 뷰 페이지(`_comments.mustache`)에 댓글 목록 뷰 페이지(`_list.mustache`)와 댓글 생성 뷰 페이지(`_new.mustache`)를 삽입하고 댓글 목록 뷰 페이지를 완성했다. 여기서는 댓글 생성 뷰 페이지를 완성한다.

### 1. 부트스트랩 카드 스타일 복사 및 수정

부트스트랩 홈페이지에서 card로 검색해 Body 스타일 코드를 복사한다. `_new.mustache` 파일을 열고 복사한 코드를 붙여 넣은 후 다음과 같이 수정한다.

- `<div>` 태그에 class 속성을 `"card m-2"`로 수정하고 `id="comments-new"` 속성을 추가한다. `"card m-2"`는 card 스타일을 사용하고 여백을 2만큼 주는 것이며, `id="comments-new"`는 이 요소를 유일하게 구분하기 위한 id이다.
- 카드 본문 텍스트(`This is...card body.`)는 지운다.

```html
<div class="card m-2" id="comments-new">
  <div class="card-body">
    </div>
</div>
```

### 2. 댓글 작성 폼 만들기

부트스트랩에서 form으로 검색해 Overview 스타일 코드를 복사하여 카드 본문(`card-body`) 영역에 붙여 넣는다. 코드 구성은 다음과 같이 수정한다.

- **닉네임 입력 폼**: 첫 번째 `<div>` 요소를 수정한다. `<label>` 요소의 `for` 속성을 지우고 제목을 닉네임으로 변경한다. `<input>` 요소의 속성을 `type="text"`, `id="new-comment-nickname"`으로 수정하고 `aria-describedby` 속성은 삭제한다.
- **댓글 본문 입력 폼**: 두 번째 `<div>` 요소를 수정한다. `<label>` 요소의 `for` 속성을 지우고 제목을 댓글 내용으로 변경한다. 여러 줄을 입력할 수 있도록 `<input>` 대신 `<textarea></textarea>`로 수정하고 속성은 `rows="3"`, `id="new-comment-body"`로 설정한다.
- **히든 인풋**: 댓글은 부모 게시글에 종속되므로 부모 게시글의 id 값을 가지고 있어야 한다. 웹 페이지에 표시되지 않으면서 값을 가지는 히든 인풋(`hidden input`)을 넣는다. 머스테치 문법으로 변수 범위를 지정하고(`{{#article}} {{/article}}`) 그 안에 `<input>` 요소의 속성을 `type="hidden"`, `id="new-comment-article-id"`, `value="{{id}}"`로 설정한다.
- **전송 버튼**: `<button>` 요소의 속성을 `type="button"`, `id="comment-create-btn"`으로 설정하고 제목을 댓글 작성으로 수정한다.
- 세 번째 `<div>`(체크 박스) 코드는 필요 없으므로 지운다.

```html
<div class="card m-2" id="comments-new">
  <div class="card-body">
    <form>
      <div class="mb-3">
        <label class="form-label">닉네임</label>
        <input type="text" class="form-control" id="new-comment-nickname">
      </div>
      <div class="mb-3">
        <label class="form-label">댓글 내용</label>
        <textarea class="form-control" rows="3" id="new-comment-body"></textarea>
      </div>
      {{#article}}
      <input type="hidden" id="new-comment-article-id" value="{{id}}">
      {{/article}}
      <button type="button" class="btn btn-primary" id="comment-create-btn">댓글 작성</button>
    </form>
  </div>
</div>
```

### 결과

![사진1](/assets/img/posts/springboot3/springboot3_chap17_1.png)

---

# 17.3 자바스크립트로 댓글 달기

웹 페이지에서 자바스크립트를 사용하면 REST API를 호출할 수 있다. 작업 중인 `_new.mustache` 코드 아래에 `<script></script>` 태그를 추가하여 자바스크립트 코드를 작성한다.

## 17.3.1 버튼 클릭 이벤트 감지하기

`querySelector()` 메서드는 웹 페이지에서 특정 요소를 선택할 때 사용한다 . id 값으로 대상을 찾을 때는 id 값 앞에 `#`을 붙이는 아이디 선택자(`#id`)를 사용한다.

`addEventListener()` 메서드는 HTML 문서의 특정 요소가 이벤트를 감지하여 이벤트가 발생하면 지정된 함수를 실행한다.

```html
<script>
{
  // 댓글 생성 버튼 변수화
  const commentCreateBtn = document.querySelector("#comment-create-btn");

  // 댓글 클릭 이벤트 감지
  commentCreateBtn.addEventListener("click", function(){
    console.log("버튼을 클릭했습니다.");
  });
}</script>
```

위 코드를 작성하고 브라우저에서 개발자 도구(F12)의 [Console] 탭을 확인하면 버튼을 클릭했을 때 메시지가 출력되는 것을 볼 수 있다.

![사진2](/assets/img/posts/springboot3/springboot3_chap17_2.png)

## 17.3.2 새 댓글 자바스크립트 객체 생성하기

새로 작성한 닉네임과 댓글 본문을 객체 리터럴 방식으로 자바스크립트 객체로 만든다. 객체 리터럴 방식은 객체를 변수로 선언해 사용하는 방식이다 . input 요소에 입력된 값은 `document.querySelector("#id_값").value` 형식으로 가져온다.

테스트용으로 적은 `console.log("버튼을 클릭했습니다.");`를 삭제하고 다음 코드를 작성한다.

```jsx
commentCreateBtn.addEventListener("click", function(){
  // 새 댓글 객체 생성
  const comment = {
    // 새 댓글의 닉네임
    nickname: document.querySelector("#new-comment-nickname").value,
    // 새 댓글의 본문
    body: document.querySelector("#new-comment-body").value,
    // 부모 게시글의 id
    articleId: document.querySelector("#new-comment-article-id").value
  };

  // 댓글 객체 출력
  console.log(comment);
});
```

닉네임과 댓글 내용을 입력하고 [댓글 작성] 버튼을 클릭하면 개발자 도구 콘솔에 자동으로 부모 게시글의 id(`articleId`)가 포함된 새 객체가 생성된 것을 확인할 수 있다.

![사진3](/assets/img/posts/springboot3/springboot3_chap17_3.png)

## 17.3.3 자바스크립트로 REST API 호출하고 응답 처리하기

`fetch()` 함수는 웹 페이지에서 HTTP 통신을 하는 데 사용하며, GET, POST, PATCH, DELETE 같은 요청을 보내고 응답을 받을 수 있다.

![사진4](/assets/img/posts/springboot3/springboot3_chap17_4.png)

POST 요청을 보낼 때 첫 번째 전달값에는 API 주소가 들어가고 두 번째 전달값에는 요청 메서드(`method`), 헤더 정보(`headers`), 전송 본문(`body`)이 들어간다. 전송 본문은 JSON 형태로 보내야 하므로 `JSON.stringify()` 함수를 사용하여 객체를 JSON 문자열로 변환한다. 헤더 정보에는 전송 본문의 데이터 타입이 JSON임을 명시한다.

요청을 보낸 후 응답 객체인 `response`를 받아 처리할 때는 `.then(response => { ... })` 구문을 사용한다. 응답 객체의 상태가 ok면 팝업 메시지를 창에 띄우고 `window.location.reload()`로 웹 페이지를 새로 고침한다.

```jsx
// 댓글 생성 API 주소
const url = "/api/articles/" + comment.articleId + "/comments";

// fetch() - 비동기 통신을 위한 API
fetch(url, {
  method: "POST", // POST 요청
  headers: {
    "Content-Type": "application/json" // 전송 본문의 데이터 타입 정보
  },
  body: JSON.stringify(comment) // comment 객체를 JSON 문자열로 변환해 전송
}).then(response => {
  // HTTP 응답 코드에 따른 메시지 출력
  const msg = (response.ok) ? "댓글이 등록됐습니다." : "댓글 등록 실패..!";
  alert(msg);
  // 현재 페이지 새로 고침
  window.location.reload();
});
```

코드를 완성하고 댓글을 작성하면 "댓글이 등록됐습니다." 메시지 창이 나타나며, [확인] 버튼을 클릭하면 페이지가 새로 고침되면서 입력한 댓글이 정상적으로 등록된다. 인텔리제이 로그의 `INSERT` 문과 H2 콘솔의 `COMMENT` 테이블 조회를 통해서도 데이터가 삽입된 것을 확인할 수 있다.

![사진5](/assets/img/posts/springboot3/springboot3_chap17_5.png)

![사진6](/assets/img/posts/springboot3/springboot3_chap17_6.png)
