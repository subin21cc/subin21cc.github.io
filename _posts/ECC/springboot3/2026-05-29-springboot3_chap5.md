---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 5장. 게시글 읽기: Read"
date: 2026-05-29 14:00:00 +0900
categories: [ECC, Team-Project, springboot3]
tags: [dev, study, java, spring]
---

["코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문" 도서 바로가기](https://www.gilbut.co.kr/book/view?bookcode=BN003778)

# 5장. 게시글 읽기: Read

# 5.1 데이터 조회 과정

데이터베이스(DB)에 저장된 데이터를 웹 페이지에 출력하기까지의 전체적인 아키텍처 흐름이다. 클라이언트의 요청부터 최종 화면 렌더링까지의 단계별 수행 과정은 다음과 같다.

1. **사용자의 URL 요청**
   사용자가 특정 데이터를 조회하기 위해 웹 페이지에서 서버로 **URL 요청**을 보낸다.
2. **컨트롤러의 요청 수신 및 전달**
   서버의 컨트롤러(Controller)가 사용자의 요청을 수신하고, 해당 URL에서 식별하려는 데이터 정보를 리파지터리(Repository)에 전달한다.
3. **리파지터리의 DB 조회 요청**
   리파지터리는 컨트롤러로부터 넘겨받은 정보를 가지고 데이터베이스(DB)에 데이터 조회를 요청한다.
4. **DB의 엔티티(Entity) 반환**
   데이터베이스는 요청에 부합하는 데이터를 검색한 후, 이를 **엔티티(Entity)** 형태로 변환하여 반환한다.
5. **모델을 통한 데이터 전달**
   반환된 엔티티는 데이터 전달 객체인 모델(Model)을 통해 화면을 표현할 뷰 템플릿(View Template)으로 전달된다.
6. **결과 뷰 페이지 출력**
   최종적으로 데이터가 결합된 **결과 뷰 페이지가 완성**되어 사용자의 화면에 출력된다.

![사진1](/assets/img/posts/springboot3/springboot3_chap5_1.png)
(위 내용을 나타낸 Gemini 생성 이미지)

---

# 5.2 단일 데이터 조회하기

## 5.2.1 URL 요청하기

클라이언트가 요청하는 URL 경로에 포함된 식별자(id)를 받아, 서버에서 처리하는 컨트롤러 구현 단계다.

### 1. 컨트롤러 파일 열기

조회할 데이터 대상이 엔티티 `Article`이므로, 이를 제어하는 `com.example.firstproject > controller > ArticleController` 파일을 열어 코드를 추가할 준비를 한다.

### 2. `@GetMapping` 매핑 추가

URL 요청을 받아오기 위해 기존 클래스 코드의 하단에 `@GetMapping()` 어노테이션을 작성한다. 요청되는 URL이 id 값에 따라 `/articles/1`, `/articles/2` 등 가변적으로 변하므로, 중괄호를 사용하여 `"/articles/{id}"` 형태로 경로를 지정한다. 중괄호 안에 지정된 `id`는 변수로 인식된다.

```java
@GetMapping("/articles/{id}") // 데이터 조회 요청 접수
```

> **💡 TIP**
>
> 스프링 컨트롤러 내에서 URL 경로 변수를 정의할 때는 중괄호 하나(`{}`)만 사용한다.
>

### 3. `show()` 메서드 생성 및 `@PathVariable` 적용

URL 요청을 수신하여 실제 로직을 수행할 `show()` 메서드를 작성한다. URL 경로에 명시된 `id` 값을 메서드의 매개변수로 가져오기 위해, 매개변수 앞에 `@PathVariable` 어노테이션을 부여한다. `@PathVariable`은 URL 요청으로 들어온 경로 변수의 값을 컨트롤러 메서드의 매개변수로 매핑해 주는 역할을 한다.

```java
@GetMapping("/articles/{id}")
public String show(@PathVariable Long id){
    return "";
}
```

### 4. 로그를 통한 수신 데이터 검증

컨트롤러가 클라이언트로부터 id 값을 정상적으로 전달받았는지 확인하기 위해 로깅 기능을 적용한다. 메서드 본문에 `log.info("id = " + id);` 코드를 추가하여 수신된 값을 로그로 출력하도록 설정한다.

```java
public String show(@PathVariable Long id){
    log.info("id = " + id); // id를 정상적으로 받았는지 확인하기 위한 로그 등록
    return "";
}
```

![사진2](/assets/img/posts/springboot3/springboot3_chap5_2.png)

### 5. 서버 재시작 및 로그 확인

현재까지의 구현 과정을 검증하기 위해 로컬 서버를 재시작한다. 브라우저를 열고 `localhost:8080/articles/1000` 주소로 접근한다. 아직 결과 페이지(뷰 템플릿)를 처리하지 않았기 때문에 브라우저 화면에는 결과가 표시되지 않지만, 통합개발환경(IDE)의 콘솔 로그 화면을 확인하면 `id = 1000` 출력을 통해 데이터가 정상 전달되었음을 확인할 수 있다.

## 5.2.2 데이터 조회해 출력하기

URL 요청을 통해 확보한 식별자(id)를 사용하여 데이터를 조회하고, 이를 화면에 출력하는 작업은 크게 다음의 3단계로 진행된다.

1. id를 조회해 DB에서 해당 데이터 가져오기
2. 가져온 데이터를 모델에 등록하기5. 전체 흐름을 테스트하기 위해 로컬 애플리케이션 서버를 재시작하고 웹 브라우저에서 `localhost:8080/articles` 경로로 접속한다. 이 시점에는 화면에 연동 데이터가 출력되지 않는다. 이는 휘발성 메모리 DB 환경 특성상 서버 프로세스가 재가동되면서 기존 적재 레코드가 모두 초기화되었기 때문이다.
6. 출력 검증용 데이터를 재생성하기 위해 `localhost:8080/articles/new` 경로에 접근하여 '가가가가/1111', '나나나나/2222', '다다다다/3333' 총 3개의 테스트 데이터를 차례대로 입력 및 제출한다.
7. 개발 툴 하단의 실행 콘솔 로그 레이어를 조회하면 식별자 id 값이 1번, 2번, 3번으로 할당된 엔티티 레코드가 순차적으로 정상 영속화되었음을 확인할 수 있다.
8. 데이터 적재 후 다시 목록 페이지(`localhost:8080/articles`)로 접속하면, 입력했던 3개의 행 데이터가 정형화된 표 구조에 맞추어 브라우저 화면에 출력된다.

### 💡핵심 머스테치 문법 특징

템플릿 엔진 문법에 기입한 변수(`articleList`)가 **데이터의 집합(Collection/List)** 구조를 가질 경우, 블록 내부에 선언된 HTML 코드가 해당 컬렉션이 보유한 데이터의 개수만큼 자동으로 반복 실행된다. 본 예제처럼 리스트 내에 데이터가 3개 등록되어 있다면 루프를 돌며 각 내부 요소의 상세 필드 값(`id`, `title`, `content`)을 차례대로 매핑하여 동적 행을 구성하게 되며, 이는 자바 언어의 `for`반복문 메커니즘과 동일하게 동작한다.
3. 조회한 데이터를 사용자에게 보여 주기 위한 뷰 페이지 만들고 반환하기

본격적인 구현에 앞서, 컨트롤러 메서드 내부에 해당 단계를 주석으로 작성하여 흐름을 정의한다.

```java
public String show(@PathVariable Long id) {
    log.info("id = " + id); [cite: 8, 9]
    
    // 1. id 조회해 데이터 가져오기 [cite: 10]
    // 2. 모델에 데이터 등록하기 [cite: 11]
    // 3. 뷰 페이지 반환하기 [cite: 12]
    
    return ""; [cite: 13]
} [cite: 15]
```

### 1단계: id를 조회해 데이터 가져오기

데이터베이스(DB)에서 데이터를 자바 객체 형태로 가져오는 주체는 리파지터리다. `@Autowired` 어노테이션을 통해 주입받은 `ArticleRepository` 객체를 사용하여 조회를 수행한다.

1. `articleRepository` 변수 뒤에 마침표(`.`)를 입력한 후, 메서드 목록에서 `findById(Long id)`를 선택한다. `findById()`는 JPA의 `CrudRepository` 인터페이스가 제공하는 메서드로, 특정 엔티티의 id 값을 기준으로 데이터를 찾아 `Optional` 타입으로 반환한다. 반환 타입을 명시하지 않은 채 찾은 데이터를 `Article` 타입의 `articleEntity` 변수에 저장하는 코드를 우선 작성한다.

```java
// 1. id를 조회해 데이터 가져오기 [cite: 39]
Article articleEntity = articleRepository.findById(id); [cite: 40]
```

2. 해당 코드를 작성하면 타입 불일치로 인해 컴파일 에러가 발생한다. `findById(id)`가 반환하는 실제 타입은 `Article`이 아니라 `Optional<Article>`이기 때문이다. 변수 타입을 `Optional<Article>`로 수정하면 에러가 해결되지만, 여기서는 뒤에 `.orElse(null)` 메서드를 붙이는 방식을 적용한다 . 이 메서드는 id 값으로 데이터를 찾을 때 해당 데이터가 존재하지 않으면 `null`을 반환하도록 설정하는 역할을 한다. 이를 통해 값이 존재할 때는 데이터 객체가, 존재하지 않을 때는 `null`이 `articleEntity` 변수에 대입된다.

    ```java
    // 1. id를 조회해 데이터 가져오기 [cite: 54]
    Article articleEntity = articleRepository.findById(id).orElse(null); [cite: 56]
    ```


### 2단계: 모델에 데이터 등록하기

`articleEntity`에 저장된 데이터를 뷰 페이지에서 사용할 수 있도록 MVC 패턴 구조에 맞추어 모델(Model)에 저장해야 한다.

1. 모델 시스템을 활용하기 위해 `show()` 메서드의 매개변수로 `Model model` 객체를 추가하여 정의한다. (클래스 참조 에러가 발생할 경우 `org.springframework.ui.Model` 패키지를 임포트한다.)

```
public String show(@PathVariable Long id, Model model){ [cite: 74]
```

2. 모델에 데이터를 등록할 때는 `addAttribute()` 메서드를 사용한다. 해당 메서드는 `model.addAttribute(String name, Object value)` 형식을 가진다. 본 실습에서는 `"article"`이라는 식별 이름으로 조회된 객체인 `articleEntity`를 등록한다.

```
// 2. 모델에 데이터 등록하기 [cite: 83]
model.addAttribute("article", articleEntity); [cite: 84]
```

![사진3](/assets/img/posts/springboot3/springboot3_chap5_3.png)

### 3단계: 뷰 페이지 반환하기

모델에 등록한 데이터를 브라우저에 표시하기 위해 최종 뷰 페이지를 생성하고 제어권을 반환한다.

1. `articles` 디렉터리 내부에 위치한 `show` 파일을 가리키도록 문자열을 반환한다.

    ```
    // 3. 뷰 페이지 반환하기 [cite: 100]
    return "articles/show"; [cite: 101]
    ```

2. 실제 파일을 생성하기 위해 프로젝트 탐색기에서 `resources > templates > articles` 디렉터리로 이동한 후, `show.mustache` 파일을 새로 만든다.
3. 파일 내부에 공통 레이아웃 구조를 적용하기 위해 상단에는 {% raw %}`{{>layouts/header}}`, 하단에는 {% raw %}`{{>layouts/footer}}`를 작성한다.

    ```
    {{>layouts/header}}
    {{>layouts/footer}}
    ```

4. 화면에 데이터를 정돈하여 보여주기 위해 부트스트랩 웹사이트(getbootstrap.com) v5.0.2 버전에 접속하여 테이블(Tables) 구성 코드를 복사한다 . (사이트의 구조 변경 등으로 검색이 불가능한 경우 준비된 예제 소스 코드를 사용한다.)
5. 복사한 HTML `<table>` 소스 코드를 `show.mustache` 파일의 헤더와 푸터 태그 레이아웃 사이에 붙여넣는다.

![사진4](/assets/img/posts/springboot3/springboot3_chap5_4.png)

6. 검증을 위해 애플리케이션 서버를 재시작하고 브라우저에서 `localhost:8080/articles/1000` 경로로 접속하여 임베딩된 표가 정상적으로 렌더링되는지 확인한다.
7. 출력 포맷에 맞추어 표의 구조를 변경한다. 제목 행의 텍스트를 `Id`, `Title`, `Content`로 수정하고 불필요한 속성을 삭제한 뒤, 세 번째 행의 `scope="row"` 및 `colspan="2"` 설정을 제거하고 임시 테스트 데이터를 대입한다 .
8. 코드를 빌드한 후 브라우저 화면을 새로고침하여 수정한 정적 임시 데이터가 표에 정상적으로 나타나는지 확인한다. 이는 실제 데이터가 연동되었을 때 표시될 레이아웃 구조를 점검하는 과정이다.
9. 고정된 임시 데이터 대신 모델에 담긴 동적 데이터를 뷰 페이지와 연동한다. 머스테치 문법에서 모델 속성 객체를 사용하기 위해 `#` 기호로 시작하고 `/` 기호로 종료하는 문법을 적용하여 `article` 데이터의 사용 범위를 지정한다 .

{% raw %}
```
{{#article}}
<tr>
    <th>1</th>
    <td>제목1111</td>
    <td>내용1111</td>
</tr>
{{/article}}
```
{% endraw %}

10. 지정된 범위 블록 내부에서 데이터의 세부 필드인 `id`, `title`, `content` 속성을 호출하도록 이중 중괄호({% raw %}`{{ }}`) 문법을 적용하여 수정한다 .

  {% raw %}
    ```
    {{#article}}
    <tr>
        <th>{{id}}</th>
        <td>{{title}}</td>
        <td>{{content}}</td>
    </tr>
    {{/article}}
    ```
  {% endraw %}

11. 다시 서버를 재시작하고 `localhost:8080/articles/3`으로 접속을 시도하면 테이블 내에 데이터가 아무것도 출력되지 않는 현상이 발생한다.
12. 현재 프로젝트가 가동 중인 데이터베이스 환경이 메모리 기반의 휘발성 DB이므로, 시스템 재시작 시점에 기존 적재 데이터가 삭제되기 때문이다. 조회를 정상적으로 확인하기 위해 `localhost:8080/articles/new` 페이지로 이동하여 신규 데이터를 입력하고 시스템에 제출(Submit)한다 . 
13. 생성 완료 후 다시 `localhost:8080/articles/1` 페이지에 접근하면 웹 브라우저에 에러 화면이 출력된다. 개발 환경 콘솔의 로그 내용을 상단으로 확인해 보면 `No default constructor for entity: com.example.firstproject.entity.Article` 에러 메시지가 발견된다 . 이는 JPA 명세상 `Article` 엔티티 클래스 내부에 매개변수가 없는 기본 생성자가 정의되어 있지 않아 인스턴스화에 실패했음을 의미한다.

![사진5](/assets/img/posts/springboot3/springboot3_chap5_5.png)

### 추가 단계: 기본 생성자 추가하기

1. 엔티티 패키지의 `Article.java` 소스 파일을 연다. 기존 클래스에는 모든 필드를 주입받는 생성자만 선언되어 있으므로 기본 생성자 정의가 요구된다.
2. 코드를 수동 작성하는 대신 롬복 라이브러리가 지원하는 `@NoArgsConstructor` 어노테이션을 클래스 상단에 추가하여 기본 생성자 구현을 자동화한다.

```
@AllArgsConstructor
@NoArgsConstructor // 기본 생성자 추가 어노테이션 [cite: 248]
@ToString
public class Article{ ... }
```

1. 구성을 마친 후 시스템 서버를 재시작한다. 데이터 초기화 상태이므로 다시 `localhost:8080/articles/new` 경로에서 새 데이터를 전송하여 데이터베이스에 id 1번으로 레코드를 재등록한다.
2. 데이터 등록 완료 후 최종적으로 `localhost:8080/articles/1`에 다시 접속하면, 엔티티 객체로부터 조회된 실제 데이터가 정의한 뷰 페이지 표 구조에 맞추어 정상적으로 출력된다.

---

# 5.3 데이터 목록 조회하기

단일 데이터가 아닌 여러 개의 데이터 목록을 조회하는 과정이다. 단일 데이터를 조회할 때는 리파지터리가 단일 엔티티를 반환하지만, 데이터 목록을 조회할 때는 엔티티의 묶음인 리스트(List)를 반환한다는 차이점이 있다.

## 5.3.1 URL 요청받기

데이터 목록 조회를 처리할 URL 경로를 `localhost:8080/articles`로 설정하여 실습을 진행한다.

1. `ArticleController` 클래스 파일을 연다. 기존 `show()` 메서드 하단에 `index()` 메서드를 추가하고, 우선 빈 문자열을 반환하도록 설정한다.
2. 추가한 `index()` 메서드 위에 `@GetMapping("/articles")` 어노테이션을 선언하여 해당 URL 경로로 들어오는 클라이언트 요청을 수신할 수 있도록 매핑한다.

![사진6](/assets/img/posts/springboot3/springboot3_chap5_6.png)

## 5.3.2 데이터 조회해 출력하기

데이터 목록을 조회하여 화면에 출력하는 로직 역시 다음 3단계의 흐름으로 구현된다.

1. DB에서 모든 Article 데이터 가져오기
2. 가져온 Article 묶음을 모델에 등록하기
3. 사용자에게 보여 줄 뷰 페이지 설정하기

메서드 구현에 앞서 `index()` 코드 내부에 해당 단계를 주석으로 작성하여 흐름을 정립한다 .

### 모든 데이터 가져오기

1. 데이터베이스의 전체 데이터를 조회하기 위해 `articleRepository.findAll()` 메서드를 호출한다. 반환값을 대입받기 위해 변수 타입을 `List<Article> articleEntityList`로 작성하면 개발 툴에서 타입 불일치 에러를 표시한다. `findAll()` 메서드가 기본적으로 반환하는 타입은 `Iterable<Article>`인데 수신 변수 타입이 `List`로 선언되었기 때문이다.

   이를 해결하는 방법은 다음과 같이 3가지가 있다.

  1. **다운캐스팅:** 상위 인터페이스 구조인 `Iterable` 타입을 하위 인터페이스 구조인 `List` 타입으로 강제 형변환(Casting)하여 타입을 일치시킨다.
  2. **업캐스팅:** 대입받을 변수의 타입을 상위 인터페이스 규격인 `Iterable<Article>`로 변경하여 반환 데이터 구조를 넓은 범위로 수용한다.
  3. **메서드 재정의:** 리파지터리에서 `findAll()` 메서드가 `Iterable`이 아닌 많이 사용하는 구현체 타입인 `ArrayList`를 반환하도록 구조를 변경한다. 본 실습에서는 이 세 번째 방식을 적용한다.

![사진7](/assets/img/posts/springboot3/springboot3_chap5_7.png)

2. `com.example.firstproject.repository.ArticleRepository` 인터페이스 파일을 연다. 이 인터페이스는 상위 객체인 `CrudRepository<Article, Long>`를 상속받고 있다. 클래스 내부 영역에서 상위 메서드 재정의 기능(Generate -> Override Methods)을 활용해 `findAll():Iterable<T>` 메서드를 선택한다. (오버라이딩은 상위 클래스나 인터페이스가 명시한 메서드를 하위 클래스에서 재정의하는 기술 규격이다.)

![사진8](/assets/img/posts/springboot3/springboot3_chap5_8.png)

3. 오버라이드된 `findAll()` 메서드의 반환 타입을 기본 `Iterable<Article>`에서 하위 구현 클래스 구조인 `ArrayList<Article>` 타입으로 변경하여 재정의한다 .

```java
@Override
ArrayList<Article> findAll();
```

1. 변경 사항을 저장하고 `ArticleController`로 돌아오면 타입 관계가 충족되어 컴파일 에러 레이아웃이 사라진다. 변수 타입을 명확하게 `ArrayList<Article>`로 선언하여 받거나 상위 계층인 `List<Article>` 구조로 업캐스팅하여 반환받을 수 있다 .

    ```java
    ArrayList<Article> articleEntityList = articleRepository.findAll();
    ```


### 모델에 데이터 등록하기

가져온 데이터의 집합인 `articleEntityList`를 뷰 레이어로 전달하기 위해 데이터 모델을 활용한다.

1. `index()` 메서드의 파라미터 구조에 `Model model` 객체를 명시하여 받아온다.
2. `model.addAttribute()` 메서드를 호출하여 `"articleList"`라는 속성 식별자 명칭으로 조회 객체인 `articleEntityList`를 바인딩한다.

```java
model.addAttribute("articleList", articleEntityList);
```
![사진9](/assets/img/posts/springboot3/springboot3_chap5_9.png)

### 뷰 페이지 설정하기

1. `articles` 디렉터리 하위의 `index` 뷰 템플릿 파일이 호출되도록 반환 구문에 경로 문자열 `"articles/index"`를 작성한다.

![사진10](/assets/img/posts/springboot3/springboot3_chap5_10.png)

2. 실제 화면을 구현하기 위해 프로젝트 경로 내 `resources > templates > articles` 위치에서 마우스 오른쪽 버튼을 클릭하여 `index.mustache` 파일을 생성한다. 
3. 파일 상단 영역에 {% raw %}`{{>layouts/header}}`를 배치하고 하단 영역에 {% raw %}`{{>layouts/footer}}` 레이아웃 템플릿을 추가하여 페이지의 기본 외곽 틀을 구성한다.
4. 조회 목록의 정돈된 연동을 위해 이전에 구현한 `show.mustache` 파일로 이동하여 `<table>` 태그로 이루어진 레이아웃 소스 코드를 전체 복사한 뒤, `index.mustache` 파일의 헤더와 푸터 태그 사이에 붙여넣는다. 
5. 컨트롤러 모델에서 `"articleList"`라는 명칭으로 데이터를 등록했으므로, 복사해 온 코드의 머스테치 시작 범위 문법과 종료 범위 문법 변수명을 각각 {% raw %}`{{#articleList}}`와 {% raw %}`{{/articleList}}`로 수정한다

![사진11](/assets/img/posts/springboot3/springboot3_chap5_11.png)

5. 전체 흐름을 테스트하기 위해 로컬 애플리케이션 서버를 재시작하고 웹 브라우저에서 `localhost:8080/articles` 경로로 접속한다. 이 시점에는 화면에 연동 데이터가 출력되지 않는다. 이는 휘발성 메모리 DB 환경 특성상 서버 프로세스가 재가동되면서 기존 적재 레코드가 모두 초기화되었기 때문이다.
6. 출력 검증용 데이터를 재생성하기 위해 `localhost:8080/articles/new` 경로에 접근하여 '가가가가/1111', '나나나나/2222', '다다다다/3333' 총 3개의 테스트 데이터를 차례대로 입력 및 제출한다.
7. 개발 툴 하단의 실행 콘솔 로그 레이어를 조회하면 식별자 id 값이 1번, 2번, 3번으로 할당된 엔티티 레코드가 순차적으로 정상 영속화되었음을 확인할 수 있다.
8. 데이터 적재 후 다시 목록 페이지(`localhost:8080/articles`)로 접속하면, 입력했던 3개의 행 데이터가 정형화된 표 구조에 맞추어 브라우저 화면에 출력된다.

### 💡핵심 머스테치 문법 특징

템플릿 엔진 문법에 기입한 변수(`articleList`)가 **데이터의 집합(Collection/List)** 구조를 가질 경우, 블록 내부에 선언된 HTML 코드가 해당 컬렉션이 보유한 데이터의 개수만큼 자동으로 반복 실행된다. 본 예제처럼 리스트 내에 데이터가 3개 등록되어 있다면 루프를 돌며 각 내부 요소의 상세 필드 값(`id`, `title`, `content`)을 차례대로 매핑하여 동적 행을 구성하게 되며, 이는 자바 언어의 `for`반복문 메커니즘과 동일하게 동작한다.
