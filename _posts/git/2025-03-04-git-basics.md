---
title: Git의 기본 개념
date: 2025-03-04 18:24:42 +0900
categories: [Git]
tags: [Git]
---

## **Git이란?**
Git은 분산 버전 관리 시스템의 한 종류이다.  
**버전 관리 시스템(Version Control System - VCS)**이란 파일의 변경 사항을 추적, 기록하고 특정 시점의 파일로 되돌릴 수 있는 시스템이다.  
**분산 버전 관리 시스템(Distributed Version Control System - DVCS)**은 중앙 서버에 접속하지 않은 상태에서도 버전 관리를 할 수 있는 시스템이다.  
단순히 파일의 마지막 스냅샷을 Checkout 하는 것이 아닌, 저장소와 그 작업 히스토리를 모두 복제하기 때문에, 서버에 문제가 발생하면 클라이언트로도 서버를 복원할 수 있다.

---
## **Git의 장점**
- 다수의 개발자가 로컬 저장소에서 개발한 뒤 원격 저장소에 Merge하는 방식으로 병렬적으로 개발 가능하다.
- 히스토리 관리를 통해 버전 간 충돌 없이 체계적으로 개발 가능하다.

---
## **Git 기본 용어**
- **Repository**: 저장소를 의미
- **Remote Repository**: 다른 작업자와 함께 공유하기 위한 저장소로 서버에 저장됨
- **Local Repository**: 개인이 작업하기 위한 저장소로 개인 단말기에 저장됨
- **Git Directory (.git)**: Git이 프로젝트의 메타데이터와 객체 데이터베이스를 저장하는 곳으로
- **Working Tree (Working Directory)**: 프로젝트의 특정 버전을 Checkout한 것으로 Git 디렉토리 안에 데이터베이스에서 파일들을 가져와 워킹 디렉토리가 만들어짐
- **Head**: 현재 작업 중인 Branch
- **Branch**: 작업할 때 특정 상태를 복사해서 별도의 작업을 하기 위해 나누는 분기점
- **Staging Area**: 저장소에 commit할 스냅샷을 준비하는 위치

---
## **Git 기본 명령어**
- **merge**: 다른 Branch의 내용을 현재 Branch와 병합시키는 작업
- **commit**: 변경된 작업 상태를 저장소에 저장하는 작업
- **switch**: 작업할 다른 branch로 Head를 변경하는 작업
- **chechout**: 특정 commit 시점으로 돌아가는 작업, Switch 작업도 포함
- **add**: 워킹 디렉토리에서 Staging area로 등록하는 작업
- **push**: 로컬 저장소에서 변경된 파일 및 히스토리를 원격 저장소에 반영시키는 작업
- **pull**: 원격 저장소에서 변경 사항을 가져와 반영하는 작업
- **fetch**: 원격 저장소에서 변경 사항을 가져오는 작업 (반영하지는 않음)
- **clone**: 원격 저장소를 복제해 로컬 저장소에 저장하는 작업
- **branch**: branch 목록 조회 및 branch 생성
- **log**: commit 히스토리 조회
- **status**: 워킹 디렉토리 상태 조회
- **init**: Git repository를 생성  
⁝

---
## **Git의 동작 원리**
![](/imgs/Git의%20기본%20개념_1.png)

1. 워킹 디렉토리에서 파일을 수정한다.
2. Staging area에 파일을 Stage해서 Commit할 스냅샷을 만든다.
3. Staging area에 있는 파일들을 Commit해서 로컬 저장소(Git directory)에 스냅샷을 저장한다.

---
## **Git의 파일 상태**
- Untracked: git의 관리 대상이 아닌 상태
- Unmodified: commit 된 파일에서 변경사항이 없는 상태
- Modified: commit 된 파일에서 변경사항이 발생해 Staging area에 올려야 하는 상태
- Staged: commit으로 저장소에 기록할 예정인 상태

---
## **Git 호스팅 웹 서비스**
웹을 통해 git 저장소 서버의 유지 및 관리를 용이하게 하는 서비스이다.  
github, gitlab 등이 있으며, CI/CD 등을 위해 원하는 작업을 자동으로 수행해주는 github actions나 gitlab CI/CD 등의 강력한 기능들을 제공한다.

## **참조**
[Scott Chacon and Ben Straub. Pro Git 2nd Edition: Apress, 2014.](https://git-scm.com/book/ko/v2)
