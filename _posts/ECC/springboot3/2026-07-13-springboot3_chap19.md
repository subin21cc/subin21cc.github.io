---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 19장. 웹 페이지에서 댓글 삭제하기"
date: 2026-07-12 20:00:00 +0900
categories: [ECC, Team-Project-BE, springboot3]
tags: [dev, study, java, spring]
render_with_liquid: false
---

["코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문" 도서 바로가기](https://www.gilbut.co.kr/book/view?bookcode=BN003778)

# 19장. 웹 페이지에서 댓글 삭제하기

이 장에서는 앞서 구현한 댓글 조회, 생성, 수정 기능에 이어 마지막으로 **댓글 삭제 기능**을 구현한다. 웹 페이지에서 댓글을 삭제하는 과정과 자바스크립트를 활용한 REST API 호출 방식을 정리한다.

# 19.1 댓글 삭제의 개요

댓글 삭제는 다음과 같이 2단계로 진행된다.

1. 댓글 **[삭제] 버튼 추가하기**
2. **[삭제] 버튼 클릭해 REST API 요청 보내기**

특히 2단계에서 [삭제] 버튼 클릭 이벤트를 처리할 때, 어느 댓글에서 삭제 요청을 했는지 알아야 서버를 통해 해당 댓글을 삭제할 수 있으므로 [삭제] 버튼을 통해 **댓글의 id 값**을 전달해야 한다.

---

# 19.2 댓글 삭제 버튼 추가하기

댓글 [삭제] 버튼을 [수정] 버튼 오른쪽에 추가한다. `templates/comments/_list.mustache` 파일을 열고 상단에서 `` 주석을 찾은 뒤, 바로 아래에 `<button>` 태그로 [삭제] 버튼을 만든다 . class 속성 값으로는 삭제 시 클릭 이벤트 처리를 위한 선택자로 사용할 `comment-delete-btn`을 추가한다.

```html
<button type="button"
class="btn btn-sm btn-outline-primary"
data-bs-toggle="modal"
data-bs-target="#comment-edit-modal"
data-bs-id="{{id}}"
data-bs-nickname="{{nickname}}"
data-bs-body="{{body}}"
data-bs-article-id="{{articleId}}">수정</button>

<button type="button"
class="btn btn-sm btn-outline-danger comment-delete-btn">삭제</button>
```

기존 장에서는 아이디 선택자(`#id`)를 이용해 HTML 요소를 선택했으나, 이 장에서는 클래스 선택자(`class`)를 이용해 요소를 선택한다. 클래스 선택자는 닷(`.`) 기호를 붙여 사용한다.

---

# 19.3 자바스크립트로 댓글 삭제하기

## 19.3.1 클릭 이벤트 처리하기

`_list.mustache` 파일 맨 아래 빈 공간에 `<script></script>` 블록을 작성한다.

```html
<script>
{
  // 삭제 버튼 선택
  const commentDeleteBtns = document.querySelectorAll(".comment-delete-btn");
}</script>
```

기존 `querySelector()` 메서드를 사용하면 문서에서 맨 처음 나온 [삭제] 버튼만 선택되어 두 번째, 세 번째 버튼이 동작하지 않는 문제가 발생한다. 이를 해결하기 위해 지정한 모든 요소를 찾아 반환하는 `querySelectorAll()` 메서드를 사용하여 모든 [삭제] 버튼을 선택한다.

버튼 여러 개를 순회하며 이벤트를 처리하기 위해 `forEach()` 메서드를 사용한다. `forEach()`는 배열 또는 NodeList 등의 데이터 묶음을 순회하며 매개변수로 주어진 함수를 각 요소에 적용한다.

```jsx
// 삭제 버튼 이벤트 처리
commentDeleteBtns.forEach(btn => {
    btn.addEventListener("click", (event) => {
        // 이벤트 발생 요소 선택
        const commentDeleteBtn = event.target;

        // 삭제 댓글 id 가져오기
        const commentId = commentDeleteBtn.getAttribute("data-comment-id");
        console.log(`삭제 버튼 클릭: ${commentId}번 댓글`);
    });
});
```

클릭된 댓글의 id를 가져오기 위해 HTML의 [삭제] 버튼에 데이터 속성인 `data-comment-id="{{id}}"`를 추가해야 한다. 자바스크립트에서는 `event.target`으로 클릭된 버튼을 가져온 후, `getAttribute("data-comment-id")`를 통해 댓글의 id 값을 얻는다. 문자열을 정의할 때 백틱(`)을 사용하면 `${}`을 통해 문자열 내에 변수를 편리하게 삽입할 수 있다.

## 19.3.2 자바스크립트로 REST API 호출하고 응답 처리하기

`fetch()` 함수를 사용하여 삭제 REST API를 호출하고 응답을 처리한다.

```jsx
// 삭제 REST API 호출
const url = `/api/comments/${commentId}`;
fetch(url, {
    method: "DELETE"
}).then(response => {
    // 댓글 삭제 실패 처리
    if (!response.ok) {
        alert("댓글 삭제 실패..!");
        return;
    }

    // 삭제 성공 시 메시지 창 띄우기 및 페이지 새로 고침
    const msg = `${commentId}번 댓글을 삭제했습니다.`;
    alert(msg);
    window.location.reload();
});
```

댓글을 삭제할 때는 전송 본문이 없으므로 `headers`와 `body` 속성은 작성하지 않는다. `method`는 `"DELETE"`로 지정한다. 응답 상태가 성공(`response.ok`)이면 알림창을 띄우고 `window.location.reload()`를 호출하여 현재 페이지를 새로 고침한다.

![사진1](/assets/img/posts/springboot3/springboot3_chap19_1.png)

---

# 19.4 책을 마무리하며

## 19.4.1 외부 DB 연동하기

그동안 사용한 H2 DB는 데이터를 디스크가 아닌 메모리에 저장하는 인메모리 데이터베이스(in-memory database)이다. 접근 속도가 빠르지만 서버를 재시작하면 작업한 데이터가 날아가는 휘발성 단점이 존재한다. 데이터를 영구적으로 유지하려면 PostgreSQL, MySQL, 오라클 등의 외부 DB를 연동해야 한다.

외부 DB 연동은 다음 3단계로 진행된다.

1. DB 설치
2. DB 관련 드라이버를 스프링 부트에 추가
3. 스프링 부트에 DB 연동 설정 작성

## 19.4.2 이 책을 끝내고 나면

스프링 부트 입문을 마치며 MVC 패턴, CRUD 기능, SQL, HTTP, JSON, REST API, 테스트 코드 등의 핵심 개념을 학습하였다. 이 책에서는 기초 체력을 다진 후 실력을 확장하기 위한 방법으로 자료 정리와 나만의 프로젝트 개발을 추천한다.

이후 추가로 학습할 수 있는 추천 주제는 다음과 같다.

- **회원 관리 및 소셜 로그인 권한**: 스프링 시큐리티(Spring Security)
- **데이터 관리 및 설계**: 스프링 JPA 및 SQL
- **클라이언트를 위한 프런트엔드**: 자바스크립트
- **인터넷 서비스 배포**: 리눅스 및 AWS(Amazon Web Services)
