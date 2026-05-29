---
title: "[Do it! 5일 만에 끝내는 깃&깃허브 입문] 4장. 깃허브 시작하기"
date: 2026-03-17 14:30:00 +0900
categories: [ECC, Team-Project, Do_It_Git]
tags: [dev, study, git]
---

["Do it! 5일 만에 끝내는 깃&깃허브 입문" 도서 바로가기](https://www.yes24.com/product/goods/130093155)

# 4장. 깃허브 시작하기


## 4-1. 원격 저장소와 깃허브

### 원격 저장소란

: 지역 저장소가 아닌 컴퓨터나 서버에 만든 저장소

### 깃허브로 할 수 있는 일

- 원격 저장소에서 깃을 사용할 수 있음
- 지역 저장소를 백업할 수 있음
- 온라인 개발 툴을 사용할 수 있음
- 협업 프로젝트에 사용할 수 있음
- 자신의 개발 이력을 남길 수 있음
- 다른 사람의 코드를 살펴볼 수 있고, 오픈 소스에 참여할 수도 있음

<hr>

## 4-2. 깃허브 가입하기

"www.github.com"에서 가입

### 지역 저장소와 원격 저장소

- 지역 저장소(local repository): 사용자 컴퓨터에 있는 저장소
- 원격 저장소(remote repository): 깃허브에 있는 저장소

### 원격 저장소 만들기

- 깃허브 - \[New repository]

### 원격 저장소 삭제하기

- \[Settings] - \[Delete this repository]

<hr>

## 4-3. 지역 저장소를 원격 저장소에 연결하기

### 원격 저장소에 연결하기

- `git remote add origin [복사한 주소]`
- `git remote -v` : 제대로 연결되었는지 목록 확인

<hr>

## 4-4. 지역 저장소와 원격 저장소 동기화하기

### 원격 저장소에 커밋 처음으로 올리기

- `git remote -v` : 로컬과 원격 브랜치를 연결(Upstream 설정)하여, 앞으로 `git push`라고만 쳐도 자동으로 서버에 올라가게 만드는 명령어

### 원격 저장소에 파일 올리기 - git push

- `git push` : 내 컴퓨터에 기록된 '버전(커밋)'들을 인터넷상의 공용 저장소로 업로드하여 동기화하는 명령어

### 원격 저장소에서 커밋 내려받기 - git pull

- 기본 형식: `git pull [원격별칭] [브랜치이름]`
  - 예: `git pull origin main` (서버의 main 브랜치 내용을 가져와 내 main에 합침)
- 단축 형식: 지난번에 `git push -u`로 연결 설정을 해두었다면, 그냥 `git pull`만 입력해도 됨.

<hr>

## 4-5. 깃허브에 SSH 원격 접속하기

### SSH 원격 접속이란

- SSH(secure shell): 보안이 강화된 안전한 방법으로 정보를 교환하는 방식
- SSH는 주로 ‘키 쌍(Key Pair)'을 이용해 인증.
  - 공개키 (Public Key): 누구나 볼 수 있는 키. 접속하려는 원격 서버에 미리 등록해 둠.
  - 비밀키 (Private Key): 오직 나(내 컴퓨터)만 가지고 있는 키.

### SSH 키 생성하기

- `ssh-keygen -t ed25519 -C "your_email@example.com"`
  - `ssh-keygen` : SSH key pair 생성하는 명령어
  - `t ed25519`: 최신 보안 표준이며 성능이 좋은 암호화 방식을 선택. (구형 시스템이라면 `t rsa -b 4096`을 사용.)
  - `C`: 본인을 식별할 수 있는 주석(보통 이메일)을 담.
- SSH 에이전트: SSH 키를 안전하게 저장하고 관리하는 프로그램

### 깃허브에 퍼블릭 키 전송하기

1. 공개키 내용 복사하기
  - `cat ~/.ssh/id_ed25519.pub`
2. GitHub 설정의 'SSH and GPG keys' 메뉴에서 내 컴퓨터의 `.pub` 파일 내용을 새 키로 추가
