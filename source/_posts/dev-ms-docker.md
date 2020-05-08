---
title: [docker]
date: 2020-04-27 14:30:19
tags: [microservice]
categories: 
    - 개발
thumbnail: ""
permalink: ""
---
## 0. 도커 배경.
 - 수많은 서버 관리(설치, 설정, 프로세스 실행)를 용이하게 합니다.
 - DevOps의 등장으로 개발주기가 짧아지면서 배포는 더 자주 이루어지고 마이크로서비스 아키텍쳐가 유행하면서 프로그램은 더 잘게 쪼개어져 관리는 더 복잡해집니다. 새로운 툴은 계속해서 나오고 클라우드의 발전으로 설치해야 할 서버가 수백, 수천대에 이르는 상황에서 도커(Docker) 가 등장하고 서버관리 방식이 완전히 바뀌게 됩니다.
<!-- more -->  \

## 1. 도커 개요.
 - 도커는 컨테이너 기반의 오픈소스 가상화 플랫폼입니다.
 - 컨테이너는 다양한 프로그램, 실행환경을 컨테이너로 추상화하고 동일한 인터페이스를 제공하여 프로그램의 배포 및 관리를 단순하게 해줍니다. 백엔드 프로그램, 데이터베이스 서버, 메시지 큐등 어떤 프로그램도 컨테이너로 추상화할 수 있고 조립PC, AWS, Azure, Google cloud등 어디에서든 실행할 수 있습니다.
 
## 2. 도커 구조
![](/images/vm-vs-docker.png)

 - 프로세스를 격리시키기 때문에 가볍고 빠르게 동작합니다. CPU나 메모리는 딱 프로세스가 필요한 만큼만 추가로 사용하고 성능적으로도 거의 손실이 없습니다..

![](/images/docker-image.png)

- 이미지는 컨테이너 실행에 필요한 파일과 설정값등을 포함하고 있는 것으로 상태값을 가지지 않고 변하지 않습니다(Immutable). 컨테이너는 이미지를 실행한 상태라고 볼 수 있고 추가되거나 변하는 값은 컨테이너에 저장됩니다.
- 같은 이미지에서 여러개의 컨테이너를 생성할 수 있고 컨테이너의 상태가 바뀌거나 컨테이너가 삭제되더라도 이미지는 변하지 않고 그대로 남아있습니다.
- ubuntu이미지는 ubuntu를 실행하기 위한 모든 파일을 가지고 있고 MySQL이미지는 debian을 기반으로 MySQL을 실행하는데 필요한 파일과 실행 명령어, 포트 정보등을 가지고 있습니다.

## 3. 도커 설치 및 설정.
 - https://luckygg.tistory.com/165
 - 도커 허브(https://cloud.docker.com) 가입.

## 4. 도커 명령어 간단 사용법.(우분투 컨테이너로 도커 익히기.)  
- docker ps -a (등록된 컨테이너 리스트 보기)
- docker search ubuntu (도커 허브에서 이미지 찾기)
- docker pull ubuntu:latest (우분투 이미지 다운로드)
- docker run -i -t -v D:\doc\docker\ubuntu:/home/ --name ubuntu_18.4.3 ubuntu /bin/bash
(도커 이미지 컨테이너 등록과 실행. /home 폴더에 D:\cube\doc\docker\ubuntu 해당 디렉토리와 연결함.
컨테이너의 Bash 셸로 옵션 -i는 표준 입력(stdin)을 활성화. -t는 TTY 모드(pseudo-TTY)를 사용. -v share volumes .vBash 환경을 쓰려면 -i와 -t 옵션은 기본)

- docker start  ubuntu_18.4.3(컨테이너 시작)
- docker attach  ubuntu_18.4.3(터미널 하나만 사용. 또 들어오면 attach(붙음)됨.)
- docker exec -it  ubuntu_18.4.3 /bin/bash (컨테이너 접속하기)
- docker inspect ubuntu_18.4.3(컨테이너 세부 정보)
- docker rm ubuntu_18.4.3(컨테이너 삭제)
- docker commit -m "메시지" 컨테이너명 이미지명:태그
docker를 사용하다 보면 docker container에 특정 프로그램을 설치 한 후 다시 image로 만들고 container를 실행해야 하는 경우가 생긴다.
docker container에 특정 프로그램을 설치하고 그대로 사용해도 문제는 없지만 혹여나 설치한 프로그램이 문제가 되어 docker container를 다시 생성해야 할 경우 이전 시점의 docker image가 없다면 복구하기가 어려워 보인다
- ctrl + p , ctrl + q - 서버 죽이지 않고 나가기.

## 5. 도커파일 명령어
- FROM: 베이스 이미지를 지정하는 명령어입니다. FROM ubuntu:14.04와 같은 방식으로 사용하고, 버전으로는 latest를 넣을 수도 있습니다. 하지만 가능하면 14.04와 같은 정확한 버전명을 기입하는 것이 좋습니다.

- RUN: 이미지 상의 리눅스 커맨드를 실행하도록 해주는 명령어입니다.(RUN 명령어는 Dockerfile로부터 이미지를 만들어 낼 때에 실행되는 것입니다.)
  예를 들어 RUN apt-get -y fortune을 하면 리눅스에서 fortune 라이브러리를 다운로드 받게 됩니다. 이 때 RUN 명령어는 일반적으로 한 번 사용될 때마다 레이어가 하나씩 추가됩니다. 그래서 RUN 명령어를 어떻게 쓰냐에 따라서 이미지의 크기가 달라질 수 있습니다. 그러므로 여러 개의 명령어가 이어지는 경우 다음과 같이 &&를 이용해 하나의 라인에 명령어를 쓰는 것이 좋습니다.
  RUN apt-get update && apt-get install -y fortune
- ADD: build 명령 중간에 호스트의 파일 시스템으로부터 파일을 가져오는 것이다. 말 그대로 이미지에 파일을 더한다(ADD).

- CMD : 컨테이너 시작 시, 실행될 명령어를 정하는 커맨드이기에 build로 이미지가 만들어지고, 그 이미지로 컨테이너를 run할 때 효력을 갖는다. 컨테이너 시작 시 실행될 명령어이기에 한번만 사용 가능한걸로 알고 있다. 

예시를 들어보자. 아래를 입력해 테스트 파일을 생성한다


## 6. 참조
https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html
https://www.bsidesoft.com/?p=7851
https://willi.am/blog/2016/07/30/docker-for-windows-sharing-host-volumes/

# docker image
https://subicura.com/2017/02/10/docker-guide-for-beginners-create-image-and-deploy.html

```
# 1. ubuntu 설치 (패키지 업데이트 + 만든사람 표시)
FROM       ubuntu:16.04
MAINTAINER subicura@subicura.com
RUN        apt-get -y update

# 2. ruby 설치
RUN apt-get -y install ruby
RUN gem install bundler

# 3. 소스 복사
COPY . /usr/src/app

# 4. Gem 패키지 설치 (실행 디렉토리 설정)
WORKDIR /usr/src/app
RUN     bundle install

# 5. Sinatra 서버 실행 (Listen 포트 정의)
EXPOSE 4567
CMD    bundle exec ruby app.rb -o 0.0.0.0
```

docker build -t app .


## docker compose

일반적으로 DOCKER를 사용하여 컨테이너를 띄우면 포트 설정이나 볼륨 설정 등을 아래와 같이 할 수 있다.
docker run --name sang12-maria -e MYSQL_ROOT_PASSWORD=mypw -d mariadb:latest  -p 3306:3306
관리하는 컨테이너가 하나이거나 올려야 하는 서비스가 한 종류라면 위와 같이 사용해도 무방할 것이다. 하지만 올려야 하는 컨테이너가 다수이고 서비스도 많다면 일일이 위처럼 명령어를 이용해서 사용하기엔 매우 불편할 것이다. 그래서 Docker Compose가 나온 듯 하다.

DOCKER-COMPOSE를 사용하면 아래와 같이 내가 사용하는 서비스를 docker-compose.yml에 저장하여 이용할 수 있다. docker-compose.yml을 작성하는 문법은 version에 따라 다르니, version에 맞는 문법을 사용해야 한다.

~~~
version: '3'

services:
  db:
    image: postgres
    volumes:
      - ./docker/data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=sampledb
      - POSTGRES_USER=sampleuser
      - POSTGRES_PASSWORD=samplesecret
      - POSTGRES_INITDB_ARGS=--encoding=UTF-8

  django:
    build:
      context: .
      dockerfile: ./compose/django/Dockerfile-dev
    environment:
      - DJANGO_DEBUG=True
      - DJANGO_DB_HOST=db
      - DJANGO_DB_PORT=5432
      - DJANGO_DB_NAME=sampledb
      - DJANGO_DB_USERNAME=sampleuser
      - DJANGO_DB_PASSWORD=samplesecret
      - DJANGO_SECRET_KEY=dev_secret_key
    ports:
      - "8000:8000"
    command: 
      - python manage.py runserver 0:8000
    volumes:
      - ./:/app/
~~~

## 도커 컴포즈의 주요 명령어
- docker-compose --version
버전 확인
- docker-compose up -d

docker-compose.yml 파일의 내용에 따라 이미지를 빌드하고 서비스를 실행합니다. 자세한 진행 과정은 다음과 같습니다.
1. 서비스를 띄울 네트워크 설정
2. 필요한 볼륨 생성(혹은 이미 존재하는 볼륨과 연결)
3. 필요한 이미지 풀(pull)
4. 필요한 이미지 빌드(build)
5. 서비스 의존성에 따라 서비스 실행

- docker-compose ps
현재 환경에서 실행 중인 각 서비스의 상태를 보여줍니다.



## 참고
http://sang12.co.kr/127/CENTOS-%EB%A6%AC%EB%88%85%EC%8A%A4-DOCKER-COMPOSE-%EC%84%A4%EC%B9%98;jsessionid=260D2ACF81D01681912DA629760B5646
https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose
https://docs.docker.com/compose/compose-file/