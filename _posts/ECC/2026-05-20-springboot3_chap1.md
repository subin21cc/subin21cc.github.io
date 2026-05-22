---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 1장. 스프링 부트 시작하기"
date: 2026-05-20 17:00:00 +0900
categories: [ECC, Team-Project, springboot3]
tags: [dev, study, java, spring]
---

["코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문" 도서 바로가기](https://www.gilbut.co.kr/book/view?bookcode=BN003778)

# 1. 스프링 부트 시작하기

# 1.1 스프링 부트란

**스프링 부트(Spring Boot)**는 복잡한 자바 웹 애플리케이션 개발을 더욱 쉽고 빠르게 할 수 있도록 도와주는 강력한 도구이다. 기존의 스프링 프레임워크는 강력한 기능을 제공하지만, 초기에 설정해야 할 항목이 너무 많아 진입 장벽이 높다는 단점이 있었다.

스프링 부트는 이러한 **‘스프링 프레임워크(Spring Framework)’의 단점을 획기적으로 개선**하여 다음과 같은 대표적인 장점을 제공한다.

- **개발 환경 설정 간소화 (Auto Configuration):** 복잡한 XML 설정이나 반복적인 자바 설정 코드 없이, 자주 사용하는 설정들을 자동으로 처리해 준다.
- **내장 웹 애플리케이션 서버(Embedded WAS) 제공:** 별도의 Tomcat 같은 웹 서버를 설치하고 내보내기(WAR)할 필요 없이, 애플리케이션 내부에 Tomcat이 내장되어 있어 단독으로 즉시 실행할 수 있다.

---

# 1.2 스프링 부트 개발 환경 설정하기

스프링 부트 개발을 시작하기 위한 환경 설정은 크게 3단계로 나뉜다.

> **JDK 설치 ➔ IDE 설치 ➔ 스프링 부트 프로젝트 생성**
>
- **JDK (Java Development Kit):** 작성한 자바 코드를 컴퓨터가 이해할 수 있도록 번역(컴파일)하고 실행해 주는 자바 개발 도구이다.
- **IDE (Integrated Development Environment):** 코드 작성, 디버깅, 빌드 등 개발에 필요한 모든 작업을 하나의 프로그램 안에서 효율적으로 처리할 수 있게 돕는 통합 개발 환경이다. 대표적으로 이클립스(Eclipse)와 인텔리제이(IntelliJ)가 있다.

## 1.2.1 JDK 설치하기

*(본 가이드는 macOS 환경을 기준으로 작성되었습니다.)*

새로운 프로젝트를 시작하기 전에, 먼저 컴퓨터에 **JDK 17** 버전이 올바르게 설치되어 있는지 확인해야 한다.

**1. 현재 기본 자바 버전 확인하기**
터미널을 열고 아래 명령어를 입력한다.

```bash
java -version
```

> [!NOTE]
만약 터미널에 `OpenJDK 23`처럼 다른 버전이 출력된다면, 이는 현재 시스템의 기본(Default) 자바 버전이 23으로 설정되어 있기 때문이다. 하지만 이것만으로는 컴퓨터에 JDK 17이 아예 설치되지 않은 것인지, 아니면 설치는 되어 있는데 우선순위에서 밀린 것인지 확실하지 않다.
>

**2. 맥에 설치된 모든 자바 버전 확인하기**
macOS에서 설치된 모든 JDK 라인업을 정확하게 확인하려면 아래 명령어를 사용한다.

```bash
/usr/libexec/java_home -V
```

![사진1](/assets/img/posts/springboot3/springboot3_chap1_1.png)

**3. Homebrew를 이용한 JDK 17 설치 과정**
만약 JDK 17이 없다면, 맥의 패키지 관리자인 홈브루(Homebrew)를 통해 간단하게 설치할 수 있다.

1. 홈브루 공식 홈페이지의 설치 스크립트를 복사하여 터미널에 실행한다.
2. 터미널에 `brew search openjdk@17`을 입력하여 설치 가능한 패키지를 검색한다.
3. `brew install openjdk@17` 명령어를 입력하여 설치를 진행한다.
4. 설치가 완료되면 다시 `java -version` 혹은 `/usr/libexec/java_home -V`를 통해 정상적으로 반영되었는지 확인한다.

## 1.2.2 IDE 설치하기

자바 생태계에서 가장 사랑받는 IDE로는 이클립스(Eclipse)와 인텔리제이(IntelliJ IDEA)가 있습니다. 본 과정에서는 강력한 코드 자동 완성 기능과 스프링 부트 친화적인 환경을 제공하는 인텔리제이(IntelliJ IDEA)를 사용합니다.

1. [인텔리제이 다운로드 공식 페이지](https://www.jetbrains.com/ko-kr/idea/download)에 접속합니다.
2. 무료로 제공되는 **Community Edition**의 `[다운로드]` 버튼을 클릭합니다.
3. 내려받은 설치 파일(.dmg)을 실행하여 애플리케이션 폴더로 드래그 앤 드롭해 설치를 완료합니다.

## 1.2.3 스프링 부트 프로젝트 만들기

### 프로젝트 만들고 실행하기

- 스프링 부트는 **Spring Initializr**라는 웹 도구를 통해 프로젝트 초기 구조를 아주 손쉽게 생성할 수 있다. [start.spring.io](https://start.spring.io/)에 접속하여 다음과 같이 옵션을 설정한다.

![사진2](/assets/img/posts/springboot3/springboot3_chap1_2.png)

  - **Project:** Gradel - Groovy
  - **Language:** Java
  - **Spring Boot:** 3.5.14
  - **Project Metadata**
    - **Group:** com.example
    - **Artifact:** firstproject
  - **Dependencies:** Spring Web, H2 Database, Mustache, Spring Data JPA
    - **Spring Web:** 웹 애플리케이션 및 RESTful 서비스를 만들기 위한 핵심 웹 도구
    - **H2 Database:** 개발 및 테스트 단계에서 가볍게 쓰기 좋은 인메모리 관계형 데이터베이스
    - **Mustache:** 화면(View) 구성을 빠르고 간결하게 도와주는 템플릿 엔진
    - **Spring Data JPA:** 데이터베이스 객체를 자바 코드로 더 편리하게 다룰 수 있게 해주는 ORM 도구
- 설정을 마친 후 하단의 **`[GENERATE]`** 버튼을 클릭하면 프로젝트가 압축 파일 형태로 다운로드된다. 압축을 해제한 후 인텔리제이에서 해당 폴더를 열어준다. 내부 빌드가 모두 끝나고 터미널 창에 ‘BUILD SUCCESSFUL’이 뜨면 프로젝트 생성 및 오픈에 성공한 것이다.

![사진3](/assets/img/posts/springboot3/springboot3_chap1_3.png)

### 헬로 월드! 출력하기

웹 서버가 정상적으로 동작하는지 확인하기 위해 간단한 HTML 파일을 만들어 띄워보겠다.

1. `src/main/resources/static/` 디렉터리 아래에 `hello.html` 파일을 생성한다.
2. 파일 내부에 화면에 보여줄 간단한 코드를 작성한다.
   ![사진4](/assets/img/posts/springboot3/springboot3_chap1_4.png)
- 스프링 부트 애플리케이션을 실행한 뒤, 웹 브라우저를 열고 `localhost:8080/hello.html` 주소로 접속하여 정상적으로 페이지가 출력되는지 확인한다.

  ![사진5](/assets/img/posts/springboot3/springboot3_chap1_5.png)

---

# 1.3 웹 서비스의 동작 원리 이해하기

## 1.3.1 클라이언트-서버 구조

우리가 매일 사용하는 웹 서비스는 크게 클라이언트(Client)와 서버(Server)의 상호작용으로 움직인다. 웹 서비스는 항상 ‘클라이언트의 요청(Request)’이 먼저 들어오면, 이에 대해 ‘서버가 응답(Response)’하는 구조로 동작한다.

- **클라이언트(Client):** 서비스를 요청하고 사용하는 주체로, 우리가 흔히 쓰는 웹 브라우저(Chrome, Safari 등)나 스마트폰 앱이 이에 해당한다.
- **서버(Server):** 서비스를 제공하는 주체로, 클라이언트의 요청을 받아 데이터를 처리하고 결과 화면이나 데이터를 돌려주는 컴퓨터 혹은 프로그램이다.

스프링 부트를 실행할 때 로그에 찍히는 `Tomcat started on port(s): 8080` 문구는, 바로 이 서버 역할을 하는 **톰캣(Tomcat)이 8080번 포트에서 클라이언트의 요청을 받을 준비를 마쳤다**는 것을 의미한다.

## 1.3.2 [localhost:8080/hello.html의](http://localhost:8080/hello.html의) 의미

주소창에 입력한 URL 주소 조각들은 저마다 중요한 의미를 담고 있다.

- **localhost**
  - **의미:** 현재 실행 중인 서버의 주소를 가리키는 특별한 호스트 네임으로, 즉 **‘내 컴퓨터(내 로컬 환경)’** 자체를 의미한다.
  - 컴퓨터가 사용하는 IP 주소로 바꾸면 `127.0.0.1`이라는 루프백(Loopback) 주소와 동일하다.
- **8080**
  - **의미:** 포트(Port) 번호이다. 컴퓨터 안에서는 여러 프로그램이 동시에 네트워크를 사용하므로, 어떤 프로그램으로 찾아가야 하는지 문 번호를 지정해 주는 것이다.
  - 스프링 부트는 기본적으로 내장된 웹 서버인 톰캣(Tomcat)을 **8080번 포트**에서 실행하도록 설정되어 있으며, 필요에 따라 이 번호는 변경할 수 있다.
- **hello.html**
  - **의미:** 서버에 요청하는 구체적인 파일 이름이다.
  - 웹 브라우저로 `localhost:8080/hello.html`에 접속하면, 내 컴퓨터의 8080번 포트에서 대기 중인 스프링 부트 서버에 "hello.html 파일 좀 보여줘"라고 요청을 보낸다.
  - 이처럼 정적 파일을 직접 지정하여 요청할 경우, 스프링 부트는 기본 규칙에 따라 **`src > main > resources > static`** 디렉터리에서 해당 파일을 찾아서 브라우저에게 HTML 코드로 응답해 준다.
  
