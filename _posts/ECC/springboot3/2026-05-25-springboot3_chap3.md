---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 3장. 게시판 만들고 새 글 작성하기: Create"
date: 2026-05-25 17:00:00 +0900
categories: [ECC, Team-Project, springboot3]
tags: [dev, study, java, spring]
---

["코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문" 도서 바로가기](https://www.gilbut.co.kr/book/view?bookcode=BN003778)

# 3장. 게시판 만들고 새 글 작성하기: Create

# 3.1 폼 데이터란

웹 애플리케이션을 사용하다 보면 회원가입을 하거나 게시글을 작성하는 등, 사용자가 입력한 데이터를 서버로 전송해야 하는 상황이 자주 발생한다. 이때 사용하는 것이 바로 폼 데이터(Form Data)다.

- **폼 데이터(Form Data):** HTML의 주요 요소 중 하나인 `<form>` 태그에 실려 서버로 전송되는 데이터를 의미한다.
- **`<form>` 태그의 역할:** 데이터를 보낼 때 **어디로(Where - action 속성)** 보낼지, 그리고 어떤 방식 즉 **어떻게(How - method 속성)** 보낼지 등을 속성에 적어서 데이터를 실어 보낸다.

이렇게 사용자가 `<form>` 태그에 실어 보낸 데이터는 무작정 데이터베이스로 갈 수 없다. 서버의 컨트롤러(Controller)가 이 데이터를 받아주어야 하는데, 이때 데이터를 빈 상자처럼 담아서 받아주는 객체가 필요하다. 이 객체를 DTO(Data Transfer Object, 데이터 전송 객체)라고 부른다.

컨트롤러가 DTO로 안전하게 전달받은 데이터는 최종적으로 가공 및 처리를 거쳐 **데이터베이스(DB)에 저장**된다.

---

# 3.2 폼 데이터를 DTO로 받기

웹 페이지에서 사용자가 입력한 게시글 데이터를 서버에서 DTO 객체로 온전히 받아내는 흐름을 한 단계씩 실습해 보겠다.

## 3.2.1 입력 폼 만들기

가장 먼저 사용자가 제목과 내용을 입력할 수 있는 화면(View)인 입력 폼을 만들어야 한다.

### **1. 디렉터리 생성**

- `src/main/resources/templates/` 디렉터리 하위에 게시판 관련 파일들을 체계적으로 관리할 수 있도록 **`articles`** 디렉터리를 생성한다.

### **2. 뷰 템플릿 파일 생성**

- 생성한 `articles` 디렉터리 하위에 **`new.mustache`** 파일을 생성하여 글쓰기 입력 폼 페이지를 구축한다. 파일 내부에는 2장에서 구현해 둔 공통 레이아웃(헤더/푸터)을 불러오고, 데이터를 전송할 기초적인 `<form>` 태그를 아래 코드와 같이 작성한다.

```html
{{>layouts/header}}

<form action="">
    <input type="text">
    <textarea></textarea>
    <button type="submit">Submit</button>
</form>

{{>layouts/footer}}
```
![사진1](/assets/img/posts/springboot3/springboot3_chap3_1.png)

## 3.2.2 컨트롤러 만들기

앞에서 작성한 입력 폼 페이지(`new.mustache`)를 웹 브라우저 화면에 띄우기 위해서는 브라우저의 URL 요청을 받아 처리해 줄 컨트롤러가 필요하다.

### **1. 컨트롤러 클래스 생성**

- `src/main/java/com.example.firstproject/controller/` 패키지 위치에 **`ArticleController`** 클래스를 새로 생성한다.

### **2. 코드 작성 및 URL 매핑**

클래스 내부에 뷰 페이지를 열어줄 메서드를 추가하고 어노테이션을 작성한다.

![사진2](/assets/img/posts/springboot3/springboot3_chap3_2.png)

- **`@Controller` 선언:** 클래스 상단에 어노테이션을 입력하여 이 파일이 스프링 부트의 컨트롤러 역할을 하는 파일임을 선언한다.
- **메서드 정의 및 파일 경로 반환:** 뷰 페이지를 보여주기 위해 `newArticleForm()` 메서드를 추가한다. 이때 반환값(`return`)으로는 보여줄 뷰 페이지의 이름을 적는데, `new.mustache` 파일이 `articles` 디렉터리 하위에 존재하므로 폴더 경로까지 포함하여 `return "articles/new";`라고 정확히 입력해 주어야 한다.
- **URL 요청 접수:** 브라우저에서 `localhost:8080/articles/new` 주소로 접속했을 때 이 뷰 페이지를 반환할 수 있도록 메서드 상단에 **`@GetMapping("/articles/new")`** 어노테이션을 부여한다.

### **3. 뷰 페이지 출력 확인**

- 스프링 부트 서버를 실행(또는 재시작)한 뒤, 웹 브라우저를 열고 `localhost:8080/articles/new`에 접속해 본다.
- 상/하단에는 우리가 이전에 모듈화해 둔 부트스트랩 내비게이션 바와 푸터가 깨짐 없이 등장하고, 중앙에는 아직 디자인이 입혀지지 않은 날 것 그대로의 입력 상자 2개와 버튼이 정상적으로 노출되는지 확인한다.

![사진3](/assets/img/posts/springboot3/springboot3_chap3_3.png)

### **4. 부트스트랩 CSS 코드로 입력 폼 디자인 입히기**

밋밋한 화면에 생기를 불어넣기 위해 부트스트랩 서식을 적용해 보겠다. `new.mustache` 파일의 `<form>...</form>` 태그 부분을 다음과 같이 세련된 스타일로 수정한다.

> **💡 참고:** 여기서는 어디까지나 폼 데이터를 전송하는 실습이 주목적이므로, 외형을 꾸미는 세부 CSS 코드(클래스 속성 등)에 대한 장황한 설명은 생략하겠다.

![사진4](/assets/img/posts/springboot3/springboot3_chap3_4.png)

### **5. 빌드 프로젝트 진행**

- 이번에는 서버 전체를 껐다 켜는 대신, 인텔리제이 상단의 망치 아이콘(Build Project)을 클릭해 변경 사항을 가볍게 컴파일 빌드한다.
- 자바 코드(`.java`)가 바뀔 때는 스프링 서버를 재시작해야 새로운 로직이 반영되지만, 머스태치(`.mustache`) 파일이나 스크립트 같은 화면단 코드가 바뀔 때는 서버 재시작 없이 망치 아이콘을 눌러 빌드만 해주어도 실시간 반영이 가능하여 시간을 절약할 수 있다.

### **6. 최종 화면 새로고침 확인**

- 웹 브라우저로 돌아와 페이지를 새로고침 하면, 부트스트랩 마진과 폼 컨트롤 디자인이 결합되어 '제목' 입력창과 '내용' 입력창이 명확히 구분된 글쓰기 양식 폼을 만나볼 수 있다.

![사진5](/assets/img/posts/springboot3/springboot3_chap3_5.png)

## 3.2.3 폼 데이터 전송하기

웹 페이지에서 사용자가 입력한 내용을 서버로 안전하게 던져주는 전송 예제를 실습해 보겠다.

앞서 작성한 입력 폼이 제 역할을 하려면, HTML의 `<form>` 태그에 다음 **2가지 핵심 속성**을 반드시 추가해야 한다. 이 속성들은 데이터를 **'어디로(Where)'**, **'어떻게(How)'** 보낼지 결정하는 이정표 역할을 한다.

- **`action` (어디로 보낼지):** 데이터가 도착할 서버의 URL 연결 주소를 적어준다. 여기서는 `action="/articles/create"`로 설정한다. 이는 내 컴퓨터에서 돌아가는 서버의 `localhost:8080/articles/create` 주소로 폼 데이터를 전송하겠다는 의미다.
- **`method` (어떻게 보낼지):** 속성값으로 크게 `get`과 `post` 2가지를 설정할 수 있다. 여기서는 새로운 리소스를 생성하는 작업이므로 **`method="post"`** 방식으로 설정한다. (GET과 POST 방식의 명확한 차이점은 추후 '7.3.1 HTTP 메서드'에서 더욱 자세히 다루겠다.)

`new.mustache` 파일을 열고 다음과 같이 `<form>` 태그의 속성을 수정한다.

```html
<form class="container" action="/articles/create" method="post">
(중략)
</form>
```

## 3.2.4 폼 데이터 받기

HTML 폼에 `action`과 `method` 속성을 올바르게 설정했으니, 이제 서버의 컨트롤러가 이 정보들을 조합하여 사용자가 전송한 폼 데이터를 넘겨받을 수 있도록 연동해 주어야 한다.

`ArticleController.java` 파일을 열고 다음과 같이 코드를 수정 및 추가한다.

**1. `createArticle()` 메서드 추가**

- 컨트롤러 내부에 웹 전송을 받아줄 `createArticle()` 메서드를 새롭게 작성한다. 우선은 에러 없이 메서드 형태(형식)만 맞추기 위해 반환값(`return`)에는 빈 문자열 `""`을 적어둔다.

**2. `@PostMapping` 어노테이션 매핑**

- 지금까지 사용했던 `@GetMapping()`은 웹 브라우저의 주소창을 통해 페이지를 조회할 때 사용하는 방식이다. 반면, 뷰 페이지(HTML 폼)에서 데이터를 `post` 방식으로 전송했기 때문에 서버의 컨트롤러에서 데이터를 받을 때도 반드시 `@PostMapping()`을 사용해야 한다.
- 이때 괄호 안에는 폼이 전송된 URL 주소를 똑같이 매칭해 준다. `new.mustache`에서 `action="/articles/create"`로 넘겼으므로, 컨트롤러에서도 `@PostMapping("/articles/create")`로 적어주어야 두 컴포넌트가 정확하게 연결된다.

![사진6](/assets/img/posts/springboot3/springboot3_chap3_6.png)

## 3.2.5 DTO 만들기

폼 데이터를 받아내기 위해, 컨트롤러의 메서드가 파라미터로 삼을 데이터 전송 객체인 DTO(Data Transfer Object)를 설계해 보겠다.

### **1. dto 패키지 생성**

- DTO 클래스들을 깔끔하게 격리하여 관리하기 위해 `com.example.firstproject` 패키지 하위에 **`com.example.firstproject.dto`** 패키지를 새로 생성한다.

### **2. `ArticleForm` 클래스 생성**

- 생성한 `dto` 패키지 아래에 **`ArticleForm`** 클래스를 만든다. 이 파일이 바로 사용자가 입력한 폼 데이터를 담아올 '그릇', 즉 DTO 역할을 수행하게 된다.

### **3. 필드(Field) 선언**

- 우리가 만든 입력 폼에서 '제목'과 '내용'을 전송할 예정이므로, DTO 내부에도 이를 각각 담아줄 필드 2개가 필요합니다. 자바 클래스 내에 `title`과 `content`를 문자열(`String`) 타입으로 선언한다.

![사진7](/assets/img/posts/springboot3/springboot3_chap3_7.png)

### **4. 생성자(Constructor) 추가**

- 필드에 값을 채워줄 생성자를 직접 작성해도 되지만, 인텔리제이의 자동 생성 기능을 활용하면 편리하다. `content` 필드 아래 행에서 마우스 오른쪽 버튼을 누르고 `Generate` ➔ `Constructor`를 선택한다.
- `Ctrl` 키(Mac의 경우 `Cmd` 키)를 누른 채 `title:String`과 `content:String`을 모두 선택한 뒤 **[OK]** 버튼을 클릭하면 생성자 코드가 자동으로 완성된다.

![사진8](/assets/img/posts/springboot3/springboot3_chap3_8.png)

### **5. `toString()` 메서드 추가**

- 향후 폼 데이터가 DTO에 올바르게 잘 담겼는지 콘솔 로그로 육안 확인하기 위해 `toString()` 메서드를 오버라이딩한다.
- 생성자 코드 아래 행에서 다시 마우스 오른쪽 버튼을 클릭하고 `Generate` ➔ `toString()`을 선택한 뒤, 두 필드가 모두 선택된 것을 확인하고 [OK]를 누른다.

![사진9](/assets/img/posts/springboot3/springboot3_chap3_9.png)

## 3.2.6 폼 데이터를 DTO에 담기

이제 준비된 DTO 객체를 활용해 전송받은 폼 데이터를 실제로 담아보고, 콘솔 창에 출력하여 값이 제대로 들어왔는지 검증해 보겠다. `ArticleController.java`로 돌아가 코드를 다음과 같이 수정한다.

### **1. 메서드 매개변수로 DTO 선언**

- 폼에서 전송한 데이터를 받아오기 위해, 아까 만들어 둔 `createArticle()` 메서드의 매개변수 자리에 DTO 클래스 타입인 **`ArticleForm form`** 객체를 선언해 준다.

### **2. 검증을 위한 출력문 추가**

- 데이터가 DTO 객체의 필드 안에 누락 없이 잘 담겼는지 확인하기 위해 출력문을 추가한다. (실무 환경이나 이후 4장에서는 시스템 성능을 위해 전문 로깅 방식으로 바꿀 예정이지만, 아직 배우기 전이므로 임시로 **`System.out.println()`** 문을 사용하겠다.)
- 매개변수로 넘어온 `form` 객체의 `toString()` 메서드를 호출하여 콘솔에 찍어본다.

![사진10](/assets/img/posts/springboot3/springboot3_chap3_10.png)

## 3.2.7 입력 폼과 DTO 필드 연결하기

여기까지 마친 후 테스트를 해보면 값이 정상적으로 전달되지 않고 `null`이 뜨는 현상이 발생한다. 그 이유는 **입력 폼의 HTML 태그**와 **자바 DTO 클래스의 필드**가 서로 어떤 이름으로 매핑되어야 하는지 명시해 주지 않았기 때문이다.

이 둘을 견고하게 연결해 주기 위해 `new.mustache` 파일의 입력 폼에 `name` 속성을 지정해 주어야 한다.

### **1. `new.mustache` 속성 추가**

- **제목 입력창:** 제목을 입력하는 `<input>` 태그에 **`name="title"`** 속성을 추가한다.
- **내용 입력창:** 내용을 입력하는 `<textarea>` 태그에 **`name="content"`** 속성을 추가한다.
![사진11](/assets/img/posts/springboot3/springboot3_chap3_11.png)

> **🔑 중요 포인트**
HTML 태그에 적어준 `name="속성명"`의 이름과 자바 DTO 클래스(ArticleForm)에 선언한 `private String 필드명`의 **이름이 완벽히 일치해야만** 스프링 부트가 데이터를 자동으로 매칭하여 DTO 내부에 집어넣어 준다.
>

### **2. 최종 전송 테스트 및 콘솔 확인**

- 자바 소스 코드가 수정되었으므로, **스프링 부트 서버를 반드시 재시작**해 준다.
- 브라우저를 새로고침한 뒤 제목창에 **`abcd`**, 내용창에 `1234`를 입력하고 **[Submit]** 버튼을 클릭한다.
- 인텔리제이 하단 실행창의 **[Run]** 탭 콘솔 로그를 확인해 보면, 우리가 추가한 출력문에 의해 다음과 같이 데이터가 전달되어 DTO 객체에 성공적으로 안착했음을 확인할 수 있다.

![사진12](/assets/img/posts/springboot3/springboot3_chap3_12.png)

---

# 3.3 DTO를 데이터베이스에 저장하기

사용자가 보낸 데이터를 DTO로 받는 것까지 완수했으니, 이제 이 데이터를 영구적으로 보관할 수 있도록 데이터베이스(DB)로 안전하게 옮겨 심어 볼 차례다.

## 3.3.1 데이터베이스와 JPA

본격적인 저장 프로세스에 들어가기 앞서, 데이터베이스 환경과 이를 제어하는 자바 기술의 유기적인 메커니즘을 명확하게 짚고 넘어가겠다.

- **데이터베이스(DB):** 쉽게 말해 데이터를 체계적으로 쌓아두고 관리하는 안전한 '데이터 창고'다. 컴퓨터 내부의 메모리와 달리 서버가 꺼져도 사라지지 않는 특징이 있으며, 모든 데이터를 행(Row)과 열(Column)로 구성된 격자 모양의 **테이블(Table)** 구조에 매핑하여 저장 및 관리한다.
- **사용할 DB 종류:** 시장에는 MySQL, Oracle, MariaDB 등 수많은 DB 프로그램들이 존재하지만, 본 프로젝트에서는 실습 및 테스트 단계에서 매우 가볍고 강력한 내장형 데이터베이스인 **H2 DB**를 사용한다. (1장에서 프로젝트 의존성을 설정할 때 함께 추가해 두었던 가벼운 인메모리 DB다.)

### 자바와 DB의 통역관: JPA(Java Persistence API)

자바 언어와 데이터베이스 언어(SQL)는 데이터를 바라보는 구조적 패러다임이 완전히 다르다. 이때 **자바 언어로 데이터베이스에 명령을 내릴 수 있도록 징검다리를 놓아주는 도구가 바로 JPA**다. JPA는 자바의 객체 지향적인 특성을 그대로 유지하면서 DB 데이터를 다룰 수 있게 도와주는 객체 관계 매핑(ORM) 기술 표준이다.

JPA를 활용해 데이터를 다루기 위해서는 아래의 **2가지 핵심 도구**를 반드시 이해해야 한다.

- **엔티티(Entity):** 자바 객체를 데이터베이스 창고가 직접 이해할 수 있도록 규격을 맞춘 특별한 클래스다. 이 엔티티 클래스를 기반으로 데이터베이스 내부에 실제 데이터를 채워 넣을 테이블(Table)이 자동으로 생성된다.
- **리파지터리(Repository):** 엔티티로 변환된 자바 객체가 실제 DB 속 테이블에 무사히 저장되고, 수정되고, 삭제되고, 또 관리될 수 있도록 DB에 직접적인 접근 명령을 수행하는 **핵심 인터페이스**다. DTO를 데이터베이스에 영구 저장하려면, 반드시 DTO를 엔티티로 변환한 후 이 리파지터리라는 일꾼에게 넘겨주어야 한다.

## 3.3.2 DTO를 엔티티로 변환하기

1. 현재 ArticleController에는 articles/create 주소로 URL 요청이 들어왔을 때 폼 데이터를 받아 출력하는 코드가 있다. 여기에다가 앞으로 할 작업을 주석으로 작성해 놓고 시작하겠다.
 ![사진13](/assets/img/posts/springboot3/springboot3_chap3_13.png)

2. DTO를 엔티티로 변환하기 위해 form 객체의 toEntity()라는 메서드를 호출해서 그 반환값을 Article 타입의 article 엔티티에 저장한다.
 ![사진14](/assets/img/posts/springboot3/springboot3_chap3_14.png)
- 그런데 오류 표시가 뜬다. 아직 Article 클래스와 toEntity() 메서드를 만들지 않았기 때문이다.

### Article 클래스 만들기

1. 빨간색으로 표시된 Article 클래스에 마우스를 올리고 조금 기다리면 Article 클래스를 만들 수 있는 링크가 뜬다. Create class ‘Article’을 클릭한다. 클래스 만들기 창에서 패키지 생성 위치를 기본 패키지(com.example.firstproject) 아래 controller가 아닌 entity라고 수정하고 [OK] 버튼을 클릭한다.

![사진15](/assets/img/posts/springboot3/springboot3_chap3_15.png)

  1. Article 클래스를 작성해 보자.
    1. 이 클래스가 엔티티임을 선언하기 위해 @Entity 어노테이션을 붙인다. @Entity는 JPA에서 제공하는 어노테이션으로, 이 어노테이션이 붙은 클래스를 기반으로 DB에 테이블이 생성된다. 테이블 이름은 클래스 이름과 동일하게 Article로 생성된다.
    2. DTO 코드를 작성할 때와 같이 title, content 필드를 선언한다. 두 필드도 DB에서 인식할 수 있게 @Column 어노테이션을 붙인다. 두 필드가 DB 테이블의 각 열과 연결된다.
    3. 마지막으로 엔티티의 대푯값을 넣는다. 대푯값을 id로 선언하고 @Id 어노테이션을 붙인다. 이어서 @GeneratedValue 어노테이션도 붙여서 대푯값을 자동으로 생성하게 한다(예: 1, 2, 3). Article 엔티티 중에 제목과 내용이 같은 것이 있더라도 대푯값 id로 다른 글임을 구분할 수 있다.

![사진16](/assets/img/posts/springboot3/springboot3_chap3_16.png)

  2. Article 객체의 생성 및 초기화를 위해 생성자를 추가한다. 코드 자동 생성 기능을 사용해 보겠다. content 필드 아래에서 마우스 오른쪽 버튼을 누르고 Generate → Constructor를 선택한다. ctrl을 누른 채 id:Long, title:String, content:String을 모두 선택한 후 [OK] 버튼을 클릭한다.
     이어서 toString() 메서드도 추가한다. 생성자 아래에서 마우스 오른쪽 버튼을 누르고 Generate → toString()을 선택한다. id:Long, title:String, content:String이 모두 선택된 것을 확인하고 [OK] 버튼을 클릭한다.

![사진17](/assets/img/posts/springboot3/springboot3_chap3_17.png)

### toEntity() 메서드 추가하기

다음으로 toEntity()메서드를 만들어 보겠다. toEntity() 메서드는 DTO인 form 객체를 엔티티 객체로 변환하는 역할을 한다.

1. 빨간색으로 표시된 toEntity()에 마우스를 올리고 잠시 기다리면 해당 메서드를 만들 수 있는 링크가 뜬다. Create method ‘toEntity’ in ‘ArticleForm’을 클릭한다. ArticleForm(DTO 클래스)이 열리면서 toEntity() 메서드가 추가된다.
2. toEntity() 메서드에서는 폼 데이터를 담은 DTO 객체를 엔티티로 반환한다(return new Article();). 전달값은 Article.java에서 생성자를 확인해 보면 id, title, content를 매개변수로 받고 있다. 아직 ArticleForm 객체에 id 정보는 없으므로 첫 번째 전달값은 null, 두 번째 전달값은 title, 세 번째 전달값은 content를 입력한다.

![사진18](/assets/img/posts/springboot3/springboot3_chap3_18.png)

3. ArticleController에 가 보면 toEntity() 메서드의 오류 표시가 사라진 것을 확인할 수 있습니다. 이렇게 해서 DTO를 엔티티로 변환하는 과정을 마쳤다.

## 3.3.3 리파지터리로 엔티티를 DB에 저장하기

ArticleController에 리파지터리(articleRepository)가 있다는 가정하에 코드를 작성하겠다.

1. articleRepository.save() 메서드를 호출해 article 엔티티를 저장한다. save() 메서드는 저장된 엔티티를 반환하므로 이를 Article 타입의 saved라는 객체에 받아 온다. 그런데 articleRepository 부분에 빨간색 오류 표시가 뜬다. 이는 articleRepository 객체를 선언하지 않고 사용했기 때문이다.
2. 필드 선언부에 ArticleRepository 타입의 articleRepository 객체를 선언한다.

![사진19](/assets/img/posts/springboot3/springboot3_chap3_19.png)

### 리파지터리 만들기

1. 프로젝트 탐색기의 com.example.firstproject에서 마우스 오른쪽 버튼을 누르고 New → Package를 선택한다. 끝에 repository를 추가해 패키지 이름을 com.example.firstproject.repository로 짓는다.
2. repository 패키지가 생성되면 마우스 오른쪽 버튼을 누르고 New → Java Class를 선택한다. 이번에는 Interface를 선택하고 이름을 ArticleRepository라고 입력한다.
3. 리파지터리는 직접 구현할 수도 있지만 JPA에서 제공하는 리파지터리 인터페이스를 활용해 만들 수도 있다. ArticleRepository 다음에 extends Crud를 입력하면 펼침 목록이 나타나는데, 그중에서 CrudRepository<T, ID>를 선택한다.

![사진20](/assets/img/posts/springboot3/springboot3_chap3_20.png)

   ### 객체 주입하기

   @Autowired는 스프링 부트에서 제공하는 어노테이션으로 이를 컨트롤러의 필드에 붙이면 스프링 부트가 만들어 놓은 객체를 가져와 주입해 준다. 이를 의존성 주입(DI, Dependency Injection)이라 한다.

   ### 데이터 저장 확인하기

  1. article 엔티티가 정말 DB에 테이블로 저장됐는지 확인해 보겠다.
    1. DTO가 엔티틸 잘 변환되는지 article.toString() 메서드를 호출해 확인한다.
    2. article이 DB에 잘 저장되는지 saved.toString() 메서드를 호출해 확인한다.

![사진21](/assets/img/posts/springboot3/springboot3_chap3_21.png)
---

# 3.4 DB 데이터 조회하기

## 3.4.1 H2 DB 접속하기

1. src > main > resources에 가면 application.properties라는 파일이 있다. 이 파일을 연다.
2. 먼저 spring.h2.console.enabled=true를 작성한다. 이는H2 DB에 웹 콘솔로 접근할 수 있도록 허용하는 설정이다.
3. [localhost:8080/h2-console](http://localhost:8080/h2-console) 페이지에 접속해서 JDBC URL을 사진과 같이 변경한다. 그리고 [Connect] 버튼을 클릭한다.

![사진22](/assets/img/posts/springboot3/springboot3_chap3_22.png)

## 3.4.2 데이터 조회하기

1. ARTICLE 테이블을 클릭하여 SELECT문을 실행해보면 아무런 데이터가 없다고 나온다.
![사진23](/assets/img/posts/springboot3/springboot3_chap3_23.png)

2다른 탭에서 [localhost:8080/articles/new](http://localhost:8080/articles/new) 페이지로 이동한다. 제목은 aaaaa, 내용은 11111을 입력한 후 [Submit] 버튼을 클릭하고 DB로 가서 보면 잘 들어간 것을 확인할 수 있다.

![사진24](/assets/img/posts/springboot3/springboot3_chap3_24.png)
