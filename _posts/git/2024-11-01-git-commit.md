---
title: Git 커밋 메시지 작성법
date: 2024-11-01 15:15:57 +0900
categories: [Git]
tags: [Git, commit, 커밋]
---

## **메시지의 형식**

```
<type>(<scope>): <subject> // 제목

<body> // 본문

<footer> // 꼬리말
```

커밋 메시지의 각 줄이 **100자를 넘지 않게** 작성해야 함

### **제목**

#### **`<type>`**

| 타입     | 내용                                  |
| -------- | ------------------------------------- |
| feat     | 새로운 기능의 추가                    |
| fix      | 오류 수정                             |
| docs     | 문서 수정                             |
| style    | 코드 스타일 혹은 포맷 수정            |
| refactor | 코드 리팩토링                         |
| test     | 테스트 코드 추가 및 수정              |
| chore    | 자잘한 수정                           |
| perf     | 성능 개선을 위한 추가 및 수정         |
| build    | 모듈 설치 혹은 삭제 등 빌드 관련 수정 |
| ci       | CI(Continuous Integration) 관련 수정  |

#### **`<scope>`**

어느 모듈이나 기능이 변경됐는지 알려주기 위한 내용이다.

프로젝트의 규모나 필요에 의해 자유롭게 작성 가능하다.
**생략 가능**하다.

#### **`<subject>`**

변경된 사항을 요약해 알려주기 위한 내용이다.

주의해야 할 사항은 다음과 같다.

- 첫 글자를 대문자로 작성하지 않는다.
- 끝에 마침표를 넣지 않는다.
- 명령형이나 현재형을 사용한다.
- what, why에 대한 내용을 설명한다.

#### **사용 예시**
`feat(auth): implement JWT authentication`

### **본문**

제목에서 작성할 수 없는 상세한 정보를 설명하기 위한 내용이다.
제목에서 이미 충분한 설명이 됐다면, **생략 가능**하다.

### **꼬리말**

#### **Github 이슈 트래킹**

해당 커밋이 프로젝트의 특정 이슈를 해결했거나 참조했을 때 사용한다.

이를 통해 코드 변경 사항이 어떤 이슈와 관련이 있는지 쉽게 추적할 수 있다.

Github 이슈 번호와 함께 아래의 키워드를 사용하면, 커밋함과 동시에 해당 이슈를 Close 상태로 전환해 해결 처리 할 수 있다.

- close
- closes
- closed
- fix
- fixes
- fixed
- resolve
- resolves
- resolved

#### **BREAKING CHANGE**

해당 커밋이 하위 버전과 호환되지 않는다는 경고를 하기 위해 사용한다.

### **커밋 메시지 예시**

```
// 제목
feat(auth): implement JWT authentication

// 본문
Added JWT-based authentication mechanism to improve security and performance. 
Users now need to use tokens for each API call instead of session cookies.

// 꼬리말
BREAKING CHANGE: removed session-based authentication.
Clients need to update their request headers with a JWT token.

Fixes #245
```

## **참조**
[AngularJS Git Commit Message Conventions](https://gist.github.com/stephenparish/9941e89d80e2bc58a153)
