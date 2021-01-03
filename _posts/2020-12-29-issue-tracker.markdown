---
layout: post
title: Redmine 이슈 트래커 구축 및 꾸미기	 
description: Redmine을 이용해 이슈트래커를 구축하고 플러그인을 더해 사용성을 높히는 일련의 과정에 대한 기록을 담았습니다. 
img: Lahion_redmine.jpg
tags: [Redmine, Postgres, Docker]
---

LAHion의 첫 이슈트래커, 레드마인(Redmine)에 대해 알아봅니다.\\
이슈트래커에 대한 간단한 소개와 레드마인의 장점과 단점을 소개하고 자세한 구축 방법까지 살펴보겠습니다.

## 이슈 트래커

이슈트래커는 프로젝트를 진행하는데 필요한 일련의 과정을 기록하고 관리하기 위한 도구입니다.

우리는 일상 생활에서 주어진 과업을 달성하기 몇가지 과정을 거칩니다.
주로 문제 정의, 세분화, 계획수립 및 수정 등과 같은 과업을 정확하고 효율적으로 달성하기위한 과정들이죠. 
이런 일련의 과정을 누군가는 기억만으로 누군가는 노트, 달력과 같은 도구를 이용해 기록합니다. 

복잡한 과업이 주어졌을때는 어떨까요?\\
여러개의 과업이 동시에 주어진다면?\\
심지어 협업을 통해 달성해야하는 과업이라면?\\

단순히 기록하거나 기억하는 것 만으로는 부족할지도 모릅니다.
의사소통, 공유 자료 저장, 회의록 작성 및 진행과정 공유 등 과업달성을 위해 필요한 것들을 위해 도구를 사용하게됩니다. 
셀을 사용할수도 있고 플래너를 사용할수도 있습니다. 메신저를 더해 의사소통 창구로 이용할 수도 있죠.

자 그런데 이런 과업이 잦다면 어떨까요?
심지어 이전에 완료한 과업을 참고해야할 일이 자주 있다면?

위에서 말한 도구만으로는 부족할 것이라 생각합니다.
문제점을 정의하고 공유하기 위한 도구가 아니기 때문이죠.

소프트웨어 개발 과정에서는 위와같은 과정들이 빈번하게 발생합니다.
프로젝트 진행과 유지보수 과정을 관리하기위한 좀 더 효율적인 도구가 필요했고, 이슈트래커가 등장했습니다. 

이슈트레커는 다양한 기능은 문제점 정의와 진행상황 공유에 초점이 맞추어져 있습니다.
참여자 개인의 문제 해결 과정과 진행상태등을 기록하고 공유할 수 있습니다.
관리자는 참여자의 문제가 해결되는 과정을 확인하고 프로젝트의 현 상황을 판단할 수 있습니다.
나아가 앞으로 필요한 것과 예상되는 문제를 인지할 수 있게됩니다.

아직도 이슈트래커를 사용하지 않고 있다면, 꼭 한 번 활용해 보기 바랍니다. 


## Redmine 

레드마인은 잘 알려진 이슈트래커입니다.
소규모 프로젝트부터 대규모 프로젝트까지 이미 [많은 분야](https://www.redmine.org/projects/redmine/wiki/WeAreUsingRedmine){:target="_blank"}에서 폭넓게 활용하고 있습니다.
프로젝트 관리를 위한 모든 기능을 무료로 사용할 수 있으면서도 플러그인을 기용해 기능을 확장하기 쉽기 때문에 부담없이 사용할 수 있다는 점에서 매력적인 이슈트래커라 할 수 있습니다.

다만, 운영을 위한 기초지식이 필요합니다.\\
오픈소스 프로젝트를 활용하는 경우에는 발생 가능한 대부분의 상황을  스스로 해결해야 하는 경우가 많습니다.\\
때문에 운용 전에 기초적인 지식은 꼭 학습는 것이 좋습니다.

> 이 글에서는 수월한 장애 복구를 위해 도커이미지를(Docker Image)를 사용했습니다.

레드마인의 장점과 단점은 아래와 같이 간략히 정리해 보았습니다.

#### 장점
1. 레드마인 기능은 설치후에 제약없이 모두 무료로 사용할 수 있다. 
2. 레드마인의 기능은 소규모부터 대규모 프로젝트까지 사용하는데 부족한 부분이 없이 많은 기능을 제공한다. 
3. 소스코드가 오픈되어 있기 때문에 입맛에 맞게 수정해서 사용할 수 있다.
4. 협업을 위해 필요한 다양한 플러그인을 쉽게 찾을 수 있고, 필요에 따라 플러그인을 개발 할 수 있다. 

#### 단점
1. 도구 설치에 익숙하지 않거나 배경지식이 부족하다면 설치하는데 어려움이 있다.
2. 사용중 문제가 발생한다면 스스로 해결해야 한다.
3. 플러그인에 따라 제한된 기능만을 사용할 수 있는 경우가 많고, 구매 비용이 높은 경우가 있다.
4. 사용상 자유도가 높아 룰을 정의하지 않고 사용한다면 다소 난잡해질 소지가 있다.

## LAHion에서 구축한 레드마인 사용 구조

![Redmine 구축 모식도]({{site.baseurl}}/assets/img/Lahion_redmine.jpg)

현재 LAHion에서 사용중인 시스템 구성은 위와 같습니다.\\
레드마인과 포스트그레스(Postgres)를 도커 이미지로 구성하고, 변경 알림을 위한 Slack bot과 E-mail 플러그인을 설정합니다.\\
사용자 데이터를 개별 디스크에 저장하도록 설정하여 컨테이너 장애 상황을 대비합니다. 

## 이슈트래커 시스템 구축

이 글은 CentOS7 을 기준으로 작성되었습니다.
CentOS7이외의 배포판 및 운영체제는 이 글에서 기술한 명령어를 이용해 설치를 진행할 수 없습니다.

### 3.0. Docker

####  3.0.1 Docker 설치전 필수 유틸리티를 설치합니다.

~~~ bash
sudo yum install -y yum-utils
sudo yum-config-manager \
   --add-repo \
   https://download.docker.com/linux/centos/docker-ce.repo
~~~
  <details>
    <summary markdown="span">상세보기</summary>
	```
    - yum-utils
    >  YUM 저장소 관리를 위한 유틸리티를 설치합니다.

    - --add-repo
    >  doker 저장소를 yum 도구에서 사용할 수 있도록 저장소 목록에 추가합니다.
	```
  </details>

#### 3.0.2. Docker engine 을 설치 합니다.

***최신버전을 설치하려면 아래의 명령어를 이용합니다.***
~~~ bash
sudo yum install docker-ce docker-ce-cli containerd.io
~~~
> 버전을 지정하지 않았다면 최신버전으로 설치됩니다.

***원하는 Docker engine 버전을 있을 때에는 아래의 명령어를 이용합니다.***

1. 이용 가능한 Docker engine 버전을 확인 확인합니다.
~~~ bash
yum list docker-ce --showduplicates | sort -r
docker-ce.x86_64  3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64  3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable
~~~
> yum list 명령으로 현재 이용 가능한 저장소 내에서 설치 가능한 도커 버전을 확인합니다. 
2. 버전을 선택하고 Docker engine 을 설치합니다.
~~~ bash
 sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
~~~

***Docker engine 을 동작시키고 문제 없이 서비스 되는지 확인합니다.***
1. Docker 서비스를 시작합니다.
~~~ bash
sudo systemctl start docker
~~~
2. Docker 서비스를 테스트합니다.
~~~ bash
sudo docker run hello-world
~~~
> 정상동작하면 hello-world 문구가 터미널로 출력됩니다.


### 3.1. Postgres(DB)

[Docker Image](https://hub.docker.com/_/postgres){:target="_blank"}
> DockerHub  공식 Image를 사용했으며, DockerHub의 사이트 정책에 따라 경로가 변경될 수 있습니다.

1. 포스트그레스 도커 이미지를 내려받습니다.
~~~ bash
 docker pull postgres
~~~
2. 포스트그레스 도커 이미지 받기가 완료 되었는지 확인합니다. 
~~~ bash
 docker images
REPOSITORY                               TAG                 IMAGE ID            CREATED             SIZE
docker.io/redmine                        latest              229055151ea0        3 weeks ago         543 MB
docker.io/postgres                       latest              f51c55ac75ed        3 weeks ago         314 MB
~~~
3. 내려 받은 이미지로 컨테이너를 생성합니다.
~~~ bash
 docker run -d --name redmine-postgres -v /mnt/data/redmine/postgres:/var/lib/postgresql/data -e POSTGRES_PASSWORD=secret postgres:tag
~~~
  <details>
    <summary markdown="span">상세보기</summary>
	```
    명령어 해설
    - docker run
    > 컨테이너를 생성합니다.
   
    - -d
    > 컨테이너를 detached mode 로 실행합니다. 이 옵션이 설정되면 컨테이를 백그라운드로 실행합니다.
      이 옵션을 지정하지 않으면 컨테이너는 포그라운드로 실행합니다. 따라서, 컨테이너가 터미널의 STDIN,STDOUT,STDOUT을 사용하게 되면서 현재 세션의 콘솔이 프로세스에 귀속됩니다.
    >
    > 참고: 컨테이너는 VM과 같은 가상장치와 달리 호스트 OS에서 실행되는 프로세스입니다. 단지, 자원을 직접 공유하는 독특한 프로세스라고 볼 수 있습니다.
   
    - --name
    > 컨테이너 이름을 지정합니다.
   
    - -v
    > 컨테이너의 데이터를 Host에 저장할 수 있도록 경로를 지정합니다.
   
    - -e
    > 컨테이너의 환경변수를 설정합니다. 여기서는 POSTGRES_PASSWORD 환경변수에 비밀번호를 설정합니다.
    >
    > 컨테이너에 사용할 수 있는 환경변수는 이미지를 빌드한 사용자의 의도에 따라 결정되며, DockerHub 저장소의 공식 이미지는 사용가능한 환경변수에 대하여 자세히 설명하고 있습니다.
     따라서 컨테이너를 사용할 때에는 이미지 생성자의 문서를 참고하는 것이 좋습니다.
   
    - postgres:tag
    > 로컬저장소에 적재된 이미지 중 "postgres"을 사용해 컨테이너를 생성합니다. 여기서 tag를 지정하면 지정된 이미지의 tag 버전으로 컨테이너를 생성합니다.
	```
  </details>

### 3.2. Redmine

[Docer Image](https://hub.docker.com/_/redmine){:target="_blank"}
> DockerHub  공식 Image를 사용했으며, DockerHub의 사이트 정책에 따라 경로가 변경될 수 있습니다.

1. 레드마인 도커 이미지를 내려 받습니다.
~~~ bash
 docker pull redmine
~~~
2. 레드마인 도커 이미지 받기가 완료 되었는지 확인합니다. 
~~~ bash
 docker images
REPOSITORY                               TAG                 IMAGE ID            CREATED             SIZE
docker.io/redmine                        latest              229055151ea0        3 weeks ago         543 MB
~~~
3. 내려 받은 이미지로 컨테이너를 생성합니다. 
~~~ bash
 docker run -d --name lahion-redmine -v /mnt/data/redmine:/usr/src/redmine/files --link redmine-postgres:postgres -p 3000:3000 redmine
~~~
  <details>
    <summary markdown="span">상세보기</summary>
	```
    명령어 해설
    - docker run
    > 컨테이너를 생성 합니다.
    
    - -d
    > 컨테이너를 detached mode 로 실행합니다. 이 옵션이 설정되면 컨테이를 백그라운드로 실행합니다.
    >
    > 이 옵션을 지정하지 않으면 컨테이너는 포그라운드로 실행됩니다. 따라서, 컨테이너가 터미널의 STDIN,STDOUT,STDOUT을 사용하게 되면서 현재 세션의 콘솔이 프로세스에 귀속됩니다.
    >
    >  참고: 컨테이너는 VM과 같은 가상장치와 달리 호스트 OS에서 실행되는 프로세스입니다. 단지, 자원을 직접 공유하는 독특한 프로세스라고 볼 수 있습니다.
    
    - --name
    > 컨테이너 이름을 지정 합니다.
    
    - -v
    > 컨테이너의 데이터를 Host에 저장할 수 있도록 경로를 지정합니다.
    
    - --link
    > 레드마인 컨테이너를 포스트그레스 컨테이너와 연결합니다.
    
    - -p
    > Host의 3000번 포트를 컨테이너의 3000번 와 매핑 합니다.
    
    - redmine
    > 로컬저장소에 적재된 이미지 중 "redmine"을 사용해 컨테이너를 생성합니다.
	```
  </details>

### 3.3. Redmine plugin

#### 주의 

* 레드마인 버전 확인 및 플러그인 지원 여부를 반드시 확인합니다.

~~~ text
플러그인이 설치할 때에는 사용하고 있는 레드마인 버전을 확인하고, 설치하고자 하는 플러그인이 사용중인 레드마인을 버전을 지원하는지 꼭 확인 해야합니다.
이를 확인하지 않고 설치하게 되면 설치도중 지원하지 않는 라이브러리 또는 api로 인해 오류가 발생합니다.

오류가 발생한 상황에서 도커 컨테이너 재시작하면 컨테이너가 동작하지 않습니다.
재시작이 실패하면, bash환경을 이용할 수 없기 때문에 오류 수정이 어렵습니다. 
따라서, 컨테이너를 재시작하지 않고 문제를 해결하는 것이 좋습니다. 
~~~

* 플러그인 설치 전 오류상황을 대비합니다.

~~~ text
플러그인 설치 전 현재 사용중인 컨테이너를 이미지로 백업해두는 것이 좋습니다. 이미지를 백업해 놓으면, 백업한 이미지로 컨테이너를 복구 할 수 있기 때문입니다.
~~~

####  Dashboard

데시보드 플러그인은 칸바보드와 유사한 플러그인으로 내게 할당된 일감을 카드 형태로 보여줍니다.

1. [Redmine Plugin](https://github.com/jgraichen/redmine_dashboard/releases){:target="_blank"} 을 다운받습니다.
> tar, zip 모두 사용 가능합니다.
2. 패키지 또는 압축 해제 방법은 아래를 참고합니다.
  * 해제 방법
    * .zip
~~~ bash
 unzip YOURFILE.zip
~~~
    * .tar
~~~ bash
 tar xvf YOURFILE.tar
~~~
    * .tar.gz
~~~ bash
 tar xvfz YOURFILE.tar.gz
~~~
3. 압축이 해제된 redmine_dashboard 디렉토리를 Redmine container로 복사합니다.
~~~ bash
 docker cp redmine_dashboard /usr/src/redmine/plugins/
~~~
4. Redmine 컨테이너의 bash 환경을 실행합니다.
~~~ bash
docker exec -it lahion-redmine /bin/bash
~~~
> "lahion-redmine"은 컨테이너 이름을 의미합니다. 
5. Bundler를 이용해 Ruby Gem을 설치합니다.
~~~ bash
cd /usr/src/redmine/
 bundle install --without development test
~~~

#### Contacts

고객 연락처와 기본 정보를 기록할 수 있는 플러그인입니다. 고객 정보와 레드마인 일감을 연결할 수 있는 점이 유용합니다.

1. [Redmine Plugin](https://www.redmine.org/plugins/redmine_contacts){:target="_blank"} 을 다운 받습니다.
2. 압축을 해제합니다.
3. 압축이 해제된 redmine_contacts 디렉토리를 Redmine container로 복사합니다.
~~~ bash
docker cp redmine_contacts /usr/src/redmine/plugins/
~~~
4. Redmine 컨테이너의 bash 환경을 실행합니다.
~~~ bash
docker exec -it lahion-redmine /bin/bash
~~~
5. Bundler를 이용해 Ruby Gem을 설치합니다.
~~~ bash
cd /usr/src/redmine/
bundle install --without development test --no-deployment
~~~
  <details>
    <summary markdown="span">상세보기</summary>
	```
    명령어 해설
    - bundle install
    > bundler를 이용해 Ruby Gem을 설치합니다.

    - --without development test
    > --with 로 포함시켰거나 Gemfile에 등록된 Gem group중 development와 test를 제외하고 설치합니다.

    -  --no-deployment
    > deployment mode 해제합니다.
    >
    > Gemfile.lock에는 Gem관련 dependency가 기록되어 있습니다.
    > deployment mode를 사용하게 되면 Gemfile.lock파일에 file lock이 설정됩니다.
    > 이러면 Gemfile을 변경하여도 Gemfile.lock을 업데이트하지 않아 변경 Gemfile과 연관된 dependency를 업데이트하지 못하기 때문에 사용상 문제가 발생합니다.
	```
  </details>

6. Rake 테스크 러너를 이용해 설치된 플러그인의 마이그레이션을 진행합니다.
~~~ bash
bundle exec rake redmine:plugins NAME=redmine_contacts RAILS_ENV=production
~~~
  <details>
    <summary markdown="span">상세보기</summary>
	```
    명령어 해설
    - bundle exec
    > bundle install로 설치된 Ruby Gem을 작성된 Gemfile에 따라 실행합니다.

    - rake
    > Gemfile에 따라 Ruby Gem을 실행하는데 사용되는 테스크 러너를 rake로 지정합니다.
    > GNU make 도구와 유사합니다.
	```
  </details>

#### Tags

레드마일 일감에 해시태그 기능을 사용할 수 있게 해줍니다. 관련 일감을 찾거나 주료 발생한 문제의 흐름을 파악에 도움을 줍니다. 

1. [Redmine plugin](https://www.redmine.org/plugins/redmine_contacts){:target="_blank"} 을 다운 받습니다.
2. 압축을 해제합니다.
3. 압축이 해제된 redmineup_tags 디렉토리를 Redmine container로 복사합니다.
~~~ bash
docker cp redmineup_tags /usr/src/redmine/plugins/
~~~
4. Redmine 컨테이너의 bash 환경을 실행합니다.
~~~ bash
docker exec -it lahion-redmine /bin/bash
~~~
  * ***주의***
> Plugin 설치를 위해 구성된 Gemfile의 설정과 디렉토리 이름이 같아야하므로 디렉토리 이름을 임의로 변경하면 오류가 발생할 수 있습니다.
5. Bundler를 이용해 Ruby Gem을 설치합니다.
~~~ bash
cd /usr/src/redmine/
  bundle install --without development test --no-deployment
~~~
6. Rake 테스크 러너를 이용해 설치된 플러그인에 필요한 마이그레이션을 진행합니다.
~~~ bash
bundle exec rake redmine:plugins NAME=redmineup_tags RAILS_ENV=production
~~~

### 3.4. Redmine Theme

레드마인 UI개선을 위해 "zenmine":https://bestredminetheme.com/zenmine/ theme 구매 후 적용했습니다.\\
레드마인이 UI적용 방법은 플러그인 설치보다도 더 쉬울 뿐 더러 사용성 개선에 큰 도움이 되었습니다. 레드마인을 프로젝트에 사용한다면
한번 쯤 고려해 보셔도 좋습니다.

#### zenmine 테마 적용
1. 압축을 해제합니다.
2. 압축 해제된 zenmine-163 디렉토리를 Redmine container로 복사합니다.
~~~ bash
docker cp zenmine /usr/src/redmine/public/themes/
~~~
3. Redmine 컨테이너의 bash 환경을 실행합니다.
~~~ bash
docker exec -it lahion-redmine /bin/bash
~~~
4. Redmine container의 재시작이 필요하지 않습니다.
5. Redmine 웹 페이지 접속 후 테마 설정합니다.

#### 스킨 적용
1. 압축을 해제합니다.
~~~ bash
unzip customize-gradient-blue-green-15.zip
~~~
2. customize-gradient-blue-green-15.css 파일 이름을 customize.css로 변경합니다.
~~~ bash
mv customize-gradient-blue-green-15.css customize.css
~~~
3. 압축 해제된 파일을 Redmine container로 복사합니다.
~~~ bash
docker cp customize.css /usr/src/redmine/public/themes/zenmine-163/customize/
~~~
4. Redmine container의 재시작은 필요하지 않습니다.
5. 레드마인 설정 도구에서 설치한 테마를 선택합니다.
6. 적용된 스킨이 웹 브라우저에서 적용되지 않았다면, 웹 브라우저의 Hard reload 기능을 실행합니다.

### 3.5. Notification

알림 기능은 유용하지만 잘못 사용하면 프로젝트가 구성원에게 보내는 신종 스팸이 되기도 합니다.\\
이런 이유로 LAHion에서는 알림을 받아야 하는 상황을 제한해야 한다고 생각했습니다. 더불어 알림을 받을 매체 또한 슬랙과 이메일로 간소화 했습니다.\\
\\
두 매체에서 알림을 발생시키는 상황 또한 중요하다고 생각했습니다.\\
모든 일감위 변동과정을 슬랙 또는 이메일로 받아야하는 것은 아니라고 판단했고 일감 생성등 꼭 필요한 상황이 아니라면 알림을 보내지 않도록 했습니다.

#### Slack
슬랙 연동을 위한 다양한 plugin중 [redmine_messenger](https://www.redmine.org/plugins/redmine_messenger){:target="_blank"} 플러그인을 선택했습니다.
슬랙 플러그인은 redmine_messenger외에도 다양하므로 원하는 플러그인을 사용해도 좋습니다.

1. [redmine_messenger](https://www.redmine.org/plugins/redmine_messenger){:target="_blank"} 플러그인을 다운받습니다.
2. 압축을 해제합니다.
3. 압축이 해제된 redmine_messenger 디렉토리를 Redmine container로 복사합니다.
~~~ bash
docker cp redmine_messenger /usr/src/redmine/plugins/
~~~
4. Redmine 컨테이너의 bash 환경을 실행합니다.
~~~ bash
docker exec -it lahion-redmine /bin/bash
~~~
5. Rake 테스크 러너를 이용해 설치한 플러그인에서 필요로하는 마이그레이션을 진행합니다.
~~~ bash
bundle exec rake redmine:plugins:migrate RAILS_ENV=production
~~~

#### E-mail

이메일 알림을 위해서는 별도의 플러그인이 필요하지 않으며, 메일링 서비스와의 연결만으로 동작합니다.
현재 사내 Notification email용도로는 sendgrid mailing service를 이용하고 있기 때문에 sendgrid를 기준으로 설정을 진행합니다.

#### Redmine config를 설정합니다.
1. Redmine 컨테이너의 bash 환경을 실행합니다.
~~~ bash
docker exec -it lahion-redmine /bin/bash
~~~
2. Redmine configuration 파일 위치로 이동합니다.
~~~ bash 
cd /usr/src/redmine/config/
~~~
3. Redmine configuration 파일을 생성합니다.
~~~ bash 
cp configuration.yml.example configuration.yml
~~~
> 여기서는 예제로 기본 제공되는 파일을 이용했습니다.
4. Redmine configuration 파일을 설정합니다.
~~~ yaml 
# - BEFORE - 
# specific configuration options for production environment
# that overrides the default ones
production:
# - AFTER -
# specific configuration options for production environment
# that overrides the default ones
production:
  delivery_method: :smtp
  smtp_settings:
    enable_starttls_auto: true   
    address: 'smtp.sendgrid.net'
      port: 587
      domain: 'lahion.com'       
      authentication: :plain
      user_name: 'apikey'        
      password: "COMODE-FOR-PASSWORD"
~~~
> address: sendgrid 서비스로 address를 지정합니다.
> domain: 도메인을 지정합니다.
> user_name: sendgrid의 경우에는 user_name을 'apikey'로 지정해야 동작합니다.

## 4. 백업 및 복구

### 4.1. Backup & Restore

도커 컨테이너를 주기적으로 백업해두면 장애 발생 상황을 좀 더 수월하게 해결할 수 있습니다.\\
이 글에서는 유저 데이터를 별도의 공간에 저장했습니다. 때문에 컨테이너의 동작에 문제가 발생했더라도 새로운 컨테이너로 대체하면 문제를 해결할 수 있기 때문입니다.

#### 도커 컨네이더 백업 방법은 아래와 같습니다.
1. 현재 사용중인 컨테이너로 이미지를 생성합니다.
~~~ bash
docker commit -p CONTAINERID IMGAENAME
~~~
2. 생성한 이미지를 .tar 파일로 저장(Archive)합니다.
~~~ bash
docker save -o /PATH/TO/STORE/IMAGE.tar IMGAGENAME
~~~

#### 저장한 이미지(.tar)로 컨테이너를 생성하는 방법은 아래와 같습니다.
1. 저장된 .tar 이미지 파일 load합니다.
~~~ bash
docker load -i /PATH/TO/RESTORE/IMAGE.tar
~~~
2. load된 이미지를 확인합니다.
~~~ bash
docker image
~~~

### 4.2. Known Issue & Trouble shutting

#### Trouble shutting

컨테이너 로그 확인 방법은 아래와 같습니다.
~~~ bash
docker logs --tail 100 --follow --timestamps CONTAINER-NAME 
~~~
<details>
    <summary markdown="span">상세보기</summary>

      명령어 해설

      - docker logs 
      > logs 명령어로 컨테이너 로그를 확인합니다. 
      > 

      - --tail 100
      > 최근 로그부터 100줄을 보여줍니다 
      >

      - --follow
      > 새로운 로그가 발생하면 계속적으로 보여줍니다. 리눅스에서 사용하는 tial도구의 -f 옵션과 동일합니다. 
      >

      - --timestamps
	  > 로그에 timestamp를 포함하여 출력합니다.

</details>

#### 컨테이너로 파일 복사 또는 컨테이너에서 파일 복사하는 방법은 아래와 같습니다.

Stop 상태인 컨테이에서도 양방향 파일 복사가 가능합니다.

***Host to Container***
~~~ bash
docker cp ./HOSTFILE /PATH/OF/CONTAINER
~~~
***Container to Host***
~~~ bash
docker cp /PATH/OF/CONTAINER ./HOSTDIRECTORY
~~~
