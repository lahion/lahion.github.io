---
title: CentOS에서 Python 업데이트 시 이슈 (Feat. PostgreSQL)
author: 태더
date: 2024-12-19
description: CentOS 8에서 Python을 업데이트 하면서, psycopg2-binary 패키지를 새로 설치할 때 PostgreSQL 설정에서 이슈가 발생했고, 해결하는 과정을 정리합니다.
---

## 개요

최근 Python 3.6 기반의 레거시 서비스를 Python 3.9로 업데이트하였습니다. 환경이 변경하면서 서버 내 PostgreSQL Client 구성이 필요하게 되었습니다.
이 포스팅은 PostgreSQL Client의 구성과 psycopg2-binary 패키지 설치 이슈를 해결하는 과정을 정리합니다.

| 서비스   | 레거시 버전 | 업데이트 버전 |
| -----  | --- | --- |
| <center>python</center> | <center>3.6</center>  | <center>3.9</center>  |
| <center>django</center> | <center>3.0</center>  | <center>4.2</center>  |

---

## 문제

### 1. pg_config 파일을 찾지 못하는 문제

```bash
$ pip install psycopg2-binary
...
Error: pg_config executable not found.
    pg_config is required to build psycopg2 from source.  Please add the directory containing pg_config to the $PATH or specify the full executable path with the option:
```

* PostgreSQL Client가 필요한 상황으로, PostgreSQL Client 설치

#### 해결 과정

* [PostgreSQL 공식 문서 - RedHat 계열 리눅스 설치 가이드](https://www.postgresql.org/download/linux/redhat/) 를 참고
* 사용중인 OS 설정에 따라 스크립트가 자동 생성 (아래 표는 실제 선택한 설정값)
  * version은 자동 생성되는 스크립트에 표시하기 위함이고, 실제로는 12이상의 모든 버전의 Repo가 등록됨

| 항목 | 선택값|
| -----  | --- |
| <center>version</center> | <center>13</center>  |
| <center>platform</center> | <center>Red Hat Entgerprise, Rocky, AlmaLiux or Oracle version 8</center>  |
| <center>architecture</center> | <center>x86_64</center>  |

```bash
$ dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
...
$ dnf -qy module enable postgresql
...
$ dnf install -y postgresql12
...
```

* 자동생성된 스크립트는 server를 설치하기 때문에, 별도로 client만 설치하기 위해 스크립트를 변경하여 실행
* 설치된 패키지의 PATH가 자동 설정되지 않기 때문에 환경변수도 설정

```bash
$ echo 'export PATH=/usr/pgsql-12/bin:$PATH' >> ~/.bashrc
$ source ~/.bashrc
```

### 2. libpq-fe.h 파일을 찾지 못하는 문제

```bash
$ pip install psycopg2-binary
...
In file included from psycopg/adapter_asis.c:28:
      ./psycopg/psycopg.h:36:10: fatal error: libpq-fe.h: No such file or directory
         36 | #include <libpq-fe.h>
            |          ^~~~~~~~~~~~
      compilation terminated.
...
```

* 해당 이슈는 postgresql 라이브러리의 헤더가 없기 때문에 발생하는 에러로, devel 패키지가 필요
* 이 과정에서 필요 의존성이 자동으로 해결되지 않아 설치되지 않음

```bash
$ dnf install -y postgresql12-devel
...
problem: cannot install the best candidate for the job
  - nothing provides perl(IPC::Run) needed by postgresql16-devel-12.22-1PGDG.rhel8.x86_64 from pgdg16
```

#### 실패한 시도

* perl-IPC-Run 패키지를 수동으로 설치하려고 했으나 실패
* rpm 으로 직접 설치하려 시도했으나, 의존성이 지속적으로 걸려 설치가 되지 않음
* *다른 방법으로 perl-IPC-Run 패키지를 설치* 해야한다는 것을 확인


#### 해결 과정

* CRB(Code Ready Builder) 저장소라는 개념이 존재: 해당 저장소에 관련 패키지가 존재
* CRB 활성화

```bash
$ crb enable
Enabling CRB repo
CRB repo is enabled and named: powertools
```

* CRB 저장소를 활용하여 perl-IPC-Run 패키지 설치

```bash
$ dnf --enablerepo=powertools install -y perl-IPC-Run
```

* 다시 postgresql12-devel 패키지 설치

```bash
$ dnf install -y postgresql12-devel
```

* 다시 psycopg2-binary 패키지 설치

```bash
$ pip install psycopg2-binary
```

## 참고

* [PostgreSQL 설치 관련 질문](https://database.sarang.net/?inc=read&aid=10427&criteria=pgsql&subcrit=qna&id=0&limit=20&keyword=&page=1)
* [PostgreSQL 메일링 리스트 - IPC::Run 이슈](https://www.postgresql.org/message-id/798318227.2099534.1708221119012%40mail.yahoo.com)
* [PostgreSQL Linux/RHEL 다운로드 페이지](https://www.postgresql.org/download/linux/redhat/)
* [RHEL CodeReady Linux Builder 저장소 문서](https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/9/html/package_manifest/codereadylinuxbuilder-repository)