---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 18장. 웹 페이지에서 댓글 수정하기"
date: 2026-07-12 20:00:00 +0900
categories: [ECC, Team-Project-BE, springboot3]
tags: [dev, study, java, spring]
render_with_liquid: false
---

["코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문" 도서 바로가기](https://www.gilbut.co.kr/book/view?bookcode=BN003778)

# 18장. 웹 페이지에서 댓글 수정하기

17장에서 댓글 생성 뷰 페이지를 만들고 자바스크립트로 새 댓글을 다는 예제를 실습했다. 이 장에서는 댓글 수정 뷰 페이지를 만들고 자바스크립트의 이벤트 처리를 통해 댓글을 수정해 본다.

# 18.1 댓글 수정의 개요

댓글 수정 페이지는 부트스트랩에서 제공하는 모달 기능을 이용해 만든다. 모달(modal)은 웹 페이지에 새 창을 띄우는 팝업 창과 달리 같은 웹 페이지 내부에서 상위 레이어를 띄우는 방식으로 사용하는 창이다.

모달 창이 뜨면 기존 창은 비활성 상태가 되고, 모달 창을 종료해야만 원래 화면으로 돌아갈 수 있다. 그러면 댓글 수정 화면을 모달 창으로 만들고, 자바스크립트를 활용해 수정 기능을 구현한다.

---

# 18.2 댓글 수정 뷰 페이지 만들기

수정 기능을 구현하고 나면 [수정] 버튼을 클릭해 모달 창을 띄우고 거기서 댓글을 수정할 수 있다.

## 18.2.1 수정 버튼과 모달 추가하기

- `_list.mustache` 파일을 연다. [수정] 버튼은 닉네임 옆에 있으니 코드의 `{{nickname}}` 다음 행에 `<button>` 태그를 넣는다. 부트스트랩에서 제공하는 모달 코드를 이용한다.

```html
<div class="card-header">
 {{nickname}}
 // 수정 버튼을 넣을 위치
</div>
```

*[파일 경로: templates/comments/_list.mustache]*

- 부트스트랩 페이지(https://getbootstrap.com)에 접속해 부트스트랩 v5.0.2를 선택한다. 왼쪽 상단의 검색창에 `modal`로 검색해 Modal components를 찾은 후 스크롤을 내리면 Live demo가 있다. 모달 창을 띄우는 버튼과 코드가 하나로 묶여 있는데, 이 코드의 `` 부분을 복사한다.

```html
<button type="button" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#exampleModal">
 Launch demo modal
</button>
```

- `_list.mustache` 파일로 돌아온다. 복사한 코드를 `{{nickname}}` 아래에 붙여 넣고 속성마다 줄바꿈해 정리한다. 버튼 제목을 `Launch demo modal`에서 `수정`으로 변경한다.

```html
<div class="card-header">
 {{nickname}}
 <button type="button"
         class="btn btn-primary"
         data-bs-toggle="modal"
         data-bs-target="#exampleModal">수정</button>
</div>
```

*[파일 경로: templates/comments/_list.mustache]*

이 버튼을 '모달 트리거 버튼'이라고 부른다. 모달 트리거 버튼을 클릭하면 모달이 실행된다. 버튼의 `data-bs-xx` 속성의 의미는 다음과 같다.

- `data-bs-toggle="modal"`: 클릭하면 모달이 나타나고 다시 클릭하면 사라짐(토글)
- `data-bs-target="#exampleModal"`: 해당 id의 모달 실행
- 부트스트랩으로 가서 `` 부분도 복사하고 `_list.mustache` 코드의 붙여 넣은 후 오른쪽으로 긴 코드는 줄바꿈해 정리한다.

```html
<div id="comments-list">
 (중략)
</div>
<div class="modal fade" id="exampleModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
 <div class="modal-dialog">
  <div class="modal-content">
   <div class="modal-header">
    <h5 class="modal-title" id="exampleModalLabel">Modal title</h5>
    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
   </div>
   <div class="modal-body">
   </div>
   <div class="modal-footer">
    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
    <button type="button" class="btn btn-primary">Save changes</button>
   </div>
  </div>
 </div>
</div>
```

*[파일 경로: templates/comments/_list.mustache]*

- 서버를 실행하고 5번 게시글의 상세 페이지(`http://localhost:8080/articles/5`)에 접속하면 모든 댓글에 [수정] 버튼이 추가된 것을 볼 수 있다. 그 중 하나를 클릭해 보면 모달 창이 뜨는 것을 확인할 수 있다.
- [수정] 버튼이 파란색 바탕에 흰색 글씨라 너무 튀므로 흰색 바탕의 파란색 글씨로 바꿔 준다. 다음과 같이 `<button>` 태그의 class 속성을 수정한다.

```html
<button type="button"
        class="btn btn-sm btn-outline-primary"
        data-bs-toggle="modal"
        data-bs-target="#exampleModal">수정</button>
```

*[파일 경로: templates/comments/_list.mustache]*

- 망치 아이콘을 클릭하고 페이지를 새로 고침하면 [수정] 버튼의 색상이 바뀐 것을 확인할 수 있다.
- `` 코드도 다음과 같이 수정한다.
  - 모달의 id를 `comment-edit-modal`로 바꾼다. 모달의 id를 수정했으므로 모달 트리거 버튼의 `data-bs-target` 속성 값도 같이 바꾼다.
  - 모달에 필요 없는 속성을 지운다 (`aria-labelledby="exampleModalLabel" aria-hidden="true"` 삭제).
  - 모달 제목을 변경한다 (`Modal title` $\rightarrow$ `댓글 수정`).
  - `[Close]`와 `[Save changes]` 버튼은 필요 없으므로 `modal-footer` 영역을 지운다.

```html
<button type="button"
        class="btn btn-sm btn-outline-primary"
        data-bs-toggle="modal"
        data-bs-target="#comment-edit-modal">수정</button>

<div class="modal fade" id="comment-edit-modal" tabindex="-1">
 <div class="modal-dialog">
  <div class="modal-content">
   <div class="modal-header">
    <h1 class="modal-title fs-5" id="exampleModalLabel">댓글 수정</h1>
    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
   </div>
   <div class="modal-body">
   </div>
  </div>
 </div>
</div>
```

*[파일 경로: templates/comments/_list.mustache]*

- 망치 아이콘을 클릭하고 페이지를 새로 고침한 뒤 [수정] 버튼 중 하나를 클릭해 보면 모달 창이 바뀐 것을 확인할 수 있다.

![사진1](/assets/img/posts/springboot3/springboot3_chap18_1.png)

## 18.2.2 수정 폼 삽입하기

모달 본문에 댓글 수정 폼을 넣기 위해 `<form>` 태그를 이용한다. 댓글 생성 뷰 페이지(`_new.mustache`)에서 만들었던 댓글 생성 폼을 가져와 사용한다.

- `_new.mustache` 파일로 가서 상단의 `<form>...</form>` 코드를 복사한다.

```html
<div class="card m-2" id="comments-new">
 <div class="card-body">
  <form>
   (중략)
  </form>
 </div>
</div>
```

*[파일 경로: templates/comments/_new.mustache]*

- `_list.mustache` 파일로 돌아와 모달 본문에 기존 내용을 삭제하고 복사한 코드를 붙여 넣는다.

```html
<div class="modal-body">
 <form>
  (중략)
 </form>
</div>
```

*[파일 경로: templates/comments/_list.mustache]*

- 붙여 넣은 코드를 다음과 같이 댓글 수정 폼으로 변경한다.
  - `<form>` 태그 위에 `` 주석을 추가한다.
  - 닉네임 input 박스의 id를 `edit-comment-nickname`으로 수정한다.
  - 댓글 본문 textarea 박스의 id를 `edit-comment-body`로 수정한다.
  - 히든 인풋의 머스테치 변수를 삭제한다 (`{{#article}}`, `{{/article}}`).
  - `<input>` 구문에 `value` 속성은 필요 없으므로 삭제한다 . id를 `edit-comment-id`(댓글의 id)로 수정한다.
  - `<input>` 구문을 복사해서 하나 더 붙여 넣고 id를 `edit-comment-article-id`(부모 게시글의 id)로 수정한다.
  - 전송 버튼의 id를 `comment-update-btn`으로 수정하고 버튼 제목을 변경한다 (`댓글 작성` → `수정 완료`).

```html
<form>
 <div class="mb-3">
  <label class="form-label">닉네임</label>
  <input type="text" class="form-control" id="edit-comment-nickname">
 </div>
 <div class="mb-3">
  <label class="form-label">댓글 내용</label>
  <textarea type="text" class="form-control" rows="3" id="edit-comment-body"></textarea>
 </div>
 <input type="hidden" id="edit-comment-id">
 <input type="hidden" id="edit-comment-article-id">
 <button type="button" class="btn btn-primary" id="comment-update-btn">수정 완료</button>
</form>
```

*[파일 경로: templates/comments/_list.mustache]*

- 망치 아이콘을 클릭하고 페이지를 새로 고침한 뒤 댓글 [수정] 버튼 중 하나를 클릭해 보면 모달 창에 수정 폼이 삽입된 것을 확인할 수 있다.

![사진2](/assets/img/posts/springboot3/springboot3_chap18_2.png)

---

# 18.3 자바스크립트로 댓글 수정하기

앞서 만든 수정 폼에 기존 댓글 정보를 가져오고, 이를 자바스크립트 요청에 실어 REST API를 호출한 후 처리한다.

## 18.3.1 트리거 데이터 전달하기

댓글을 수정하려면 이전 댓글 데이터를 가져와야 한다. `_list.mustache` 파일에서 댓글은 `{{#commentDtos}}…{{/commentDtos}}` 머스테치 문법으로 감싸져 있다. 이 구문을 이용해 `commentDto`에 저장된 데이터를 모달 트리거 버튼의 속성 값으로 가져온다.

- `data-`로 시작하는 데이터 속성을 추가하여 HTML 요소에 정보를 저장한다.
  - `data-bs-id` 속성을 추가하고 현재 댓글의 `{{id}}` 값을 저장한다.
  - `data-bs-nickname` 속성을 추가하고 현재 댓글의 `{{nickname}}` 값을 저장한다.
  - `data-bs-body` 속성을 추가하고 현재 댓글의 `{{body}}` 값을 저장한다.
  - `data-bs-article-id` 속성을 추가하고 현재 댓글의 `{{articleId}}` 값을 저장한다.

```html
<button type="button"
        class="btn btn-sm btn-outline-primary"
        data-bs-toggle="modal"
        data-bs-target="#comment-edit-modal"
        data-bs-id="{{id}}"
        data-bs-nickname="{{nickname}}"
        data-bs-body="{{body}}"
        data-bs-article-id="{{articleId}}">수정</button>
```

*[파일 경로: templates/comments/_list.mustache]*

- 받아 온 데이터를 모달의 각 폼에 출력하는 모달 이벤트 처리를 진행한다. 코드 맨 아래에 `<script></script>` 태그를 추가하고 영역을 잡는다.

```html
<div id="comments-list">
 (중략)
</div>
<div class="modal fade" id="comment-edit-modal" tabindex="-1">
</div>
<script>
 {
  // 이벤트 처리 내용 위치
 }</script>
```

*[파일 경로: templates/comments/_list.mustache]*

- `querySelectorAll()` 또는 `querySelector()` 메서드로 모달(`comment-edit-modal`)을 선택하고 이를 `commentEditModal` 변수에 저장한다.

```jsx
<script>
 {
  // 모달 요소 선택
  const commentEditModal = document.querySelector("#comment-edit-modal");
 }
</script>
```

*[파일 경로: templates/comments/_list.mustache]*

- 모달이 열리는 이벤트를 감지하기 위해 `addEventListener()` 메서드를 사용한다.

>
>
>
> **형식:** 요소명.addEventListener("이벤트_타입", 이벤트_처리_함수)
>

모달이 열릴 때 바로 함수를 실행하는 명령을 작성한다. `show.bs.modal`은 모달이 표시되기 직전 실행되는 이벤트 타입이다.

```jsx
<script>
 {
  const commentEditModal = document.querySelector("#comment-edit-modal");
  // 모달 이벤트 감지
  commentEditModal.addEventListener("show.bs.modal", function(event){
  });
 }
</script>
```

*[파일 경로: templates/comments/_list.mustache]*

| **이벤트 타입** | **설명** |
| --- | --- |
| **show.bs.modal** | 모달이 표시되기 직전 실행되는 이벤트  |
| **shown.bs.modal** | 모달이 표시된 후 실행되는 이벤트  |
| **hide.bs.modal** | 모달이 숨겨지기 직전 실행되는 이벤트  |
| **hidden.bs.modal** | 모달이 숨겨진 후 실행되는 이벤트  |
- `function(event)` 함수 내부의 구현 주석을 작성한다.

```jsx
commentEditModal.addEventListener("show.bs.modal", function(event){
  // 1. 트리거 버튼 선택
  // 2. 데이터 가져오기
  // 3. 수정 폼에 데이터 반영
});
```

*[파일 경로: templates/comments/_list.mustache]*

- 모달 트리거 버튼은 `event.relatedTarget`으로 선택할 수 있으며, 선택한 버튼은 `triggerBtn` 변수에 저장한다.

```jsx
// 1. 트리거 버튼 선택
const triggerBtn = event.relatedTarget;
```

*[파일 경로: templates/comments/_list.mustache]*

- `triggerBtn` 변수를 통해 `getAttribute()`를 사용하여 데이터 속성 값을 가져와 변수에 저장한다.

```jsx
// 2. 데이터 가져오기
const id = triggerBtn.getAttribute("data-bs-id");
const nickname = triggerBtn.getAttribute("data-bs-nickname");
const body = triggerBtn.getAttribute("data-bs-body");
const articleId = triggerBtn.getAttribute("data-bs-article-id");
```

*[파일 경로: templates/comments/_list.mustache]*

- 가져온 데이터를 `querySelector()`로 선택한 각 입력 폼의 `value` 속성에 반영한다.

```jsx
// 3. 수정 폼에 데이터 반영
document.querySelector("#edit-comment-nickname").value = nickname;
document.querySelector("#edit-comment-body").value = body;
document.querySelector("#edit-comment-id").value = id;
document.querySelector("#edit-comment-article-id").value = articleId;
```

*[파일 경로: templates/comments/_list.mustache]*

- 망치 아이콘을 클릭하고 페이지를 새로 고침한 후, 댓글의 [수정] 버튼을 클릭하면 모달 창의 수정 폼에 해당 댓글의 내용이 들어 있는 것을 확인할 수 있다.
- [수정 완료] 버튼에서 마우스 오른쪽 버튼을 클릭하고 [검사] 메뉴를 선택해 개발자 도구의 [Elements] 탭에서 히든 인풋 요소의 값(value)이 잘 들어 있는지 확인한다.

![사진3](/assets/img/posts/springboot3/springboot3_chap18_3.png)

## 18.3.2 자바스크립트로 REST API 호출하고 응답 처리하기

[수정 완료] 버튼을 클릭해 REST API를 호출하여 댓글을 수정한다.

- `<script>` 태그 안쪽 아래에 수정 완료 버튼을 `querySelector("#comment-update-btn")`으로 선택하여 `commentUpdateBtn` 변수에 저장한다.

```jsx
// 수정 완료 버튼 선택
const commentUpdateBtn = document.querySelector("#comment-update-btn");
```

*[파일 경로: templates/comments/_list.mustache]*

- `commentUpdateBtn` 버튼에 클릭 이벤트가 발생하는지 감지하여 이벤트를 처리하도록 `addEventListener()`를 작성한다.

```jsx
// 클릭 이벤트 처리
commentUpdateBtn.addEventListener("click", function(){
});
```

*[파일 경로: templates/comments/_list.mustache]*

- 객체 리터럴 방식으로 수정 댓글 객체를 생성하고, 각 키의 값은 폼 요소의 `value`를 가져온다. 객체 생성을 확인하기 위해 콘솔 로그를 추가한다.

```jsx
commentUpdateBtn.addEventListener("click", function(){
  // 수정 댓글 객체 생성
  const comment = {
    id: document.querySelector("#edit-comment-id").value,
    nickname: document.querySelector("#edit-comment-nickname").value,
    body: document.querySelector("#edit-comment-body").value,
    article_id: document.querySelector("#edit-comment-article-id").value
  };
  console.log(comment);
});
```

*[파일 경로: templates/comments/_list.mustache]*

- 망치 아이콘을 클릭하고 페이지를 새로 고침한 후 [수정] 버튼을 눌러 내용을 변경하고 [수정 완료] 버튼을 클릭한다.
- 개발자 도구의 [Console] 탭을 보면 수정 댓글 객체가 생성된 것을 확인할 수 있다.

![사진4](/assets/img/posts/springboot3/springboot3_chap18_4.png)

- REST API를 호출하기 위해 댓글 수정 API 주소를 `url` 변수에 저장한다. 댓글 id는 매번 바뀌므로 `comment.id`를 연결한다.

```jsx
// 수정 REST API 호출
const url = "/api/comments/" + comment.id;
```

*[파일 경로: templates/comments/_list.mustache]*

- `fetch()` 함수를 작성하여 첫 번째 인자로 `url`을 넘기고, 두 번째 인자로 요청 메서드(`PATCH`), 헤더 정보(`Content-Type`), JSON 형태로 변환한 전송 본문(`JSON.stringify(comment)`)을 넘긴다.

```jsx
fetch(url, {
  method: "PATCH",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify(comment)
});
```

*[파일 경로: templates/comments/_list.mustache]*

- 실행 후 인텔리제이 로그를 보면 `UPDATE` 문이 실행된 것을 볼 수 있다.
- `http://localhost:8080/h2-console`에 접속하여 `COMMENT` 테이블을 조회하면 데이터가 수정된 것을 확인할 수 있다.
- 사용자가 수정 결과를 확인할 수 있도록 `fetch().then(response => { ... })` 구문을 이용해 응답 처리를 추가한다. 상태가 ok면 "댓글이 수정됐습니다."를, 아니면 "댓글 수정 실패..!" 메시지를 출력한 뒤 `window.location.reload()`로 웹 페이지를 새로 고침한다.

```jsx
// 수정 REST API 호출
const url = "/api/comments/" + comment.id;
fetch(url, {
  method: "PATCH",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify(comment)
}).then(response => {
  // HTTP 응답 코드에 따른 메시지 출력
  const msg = (response.ok) ? "댓글이 수정됐습니다." : "댓글 수정 실패..!";
  alert(msg);
  // 현재 페이지 새로 고침
  window.location.reload();
});
```

*[파일 경로: templates/comments/_list.mustache]*

- 망치 아이콘을 클릭하고 페이지를 새로 고침한 뒤 댓글을 수정해 보면 메시지 창이 나타나고, [확인] 버튼을 클릭하면 최종적으로 댓글이 수정 완료된 것을 확인할 수 있다.

![사진5](/assets/img/posts/springboot3/springboot3_chap18_5.png)

![사진6](/assets/img/posts/springboot3/springboot3_chap18_6.png)
