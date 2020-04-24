---
title: [BridgeWar]
date: 2020-01-21 14:30:19
tags: [game svr]
categories:
    - 프로젝트
thumbnail: ""
permalink: ""
---
## 1. 서버 구성도 
![](/images/404d06966e444c82bfd0082847a86424.png)
<!-- more -->
![](/images/e809b4238e6543858afdec7ce557e30c.png)
## 2. 개발 환경
 -  AWS(ec2, Elastic Load balancer, DBS) + Centos 7.0 + apache + CI(php) + mysql + memcached + mcrypt

## 3. 프로세스
1. Rest API로 통신을 했으며 json으로 데이터 형식을 표현.
2. 데이터는 레퍼런스, 유저, 로그, 통합 데이터로 구분.
3. 분산 디비를 위해 몇십만 이상시에 로그와 유저 디비가 생성하도록 하였고 맴캐쉬에 레퍼런스 데이터를 저장하도록 함.(유저 키로 구분)
4. 세션마다 키를 주어 암호화 함.(mcrypt)
5. 클라이언트에서 서버 에러시에 서버에 로그를 남기도록 함.
6. 매칭 및 랭킹은 통합 디비에서 실시간 매칭과 랭킹으로 구현.
7. 채팅 서버를 채널별 로드 밸런싱을 하였고 여러개의 프로세스로 분산화 함
8. 이벤트와 각종 서비스를 셀로 구현
 8.1 프로세스 죽었을시에 sms 날리고 다시 살리는 셀.(DB, 아파치, 채팅)
 8.2 매일 통계 데이터 넣는 부분 셀.
 8.3 health check 셀.
 8.4 오래된 맬 삭제 셀.
 8.5 각종 이벤트 셀.
<br>

## 4. 미들웨어

## 5. 아마존 서버 구성
DB(2대) db.m4.xlarge - cpu 4, memory 16 SSD 100G
WEB(3대) m4.xlarge - cpu 4, memory 16 SSD 80G
Elastic Load Balancer

## 6. 설치 및 설정
### memcache 설치
0. 맴케쉬는 web01에서만 설치한다.
1. [memcache 설치](http://www.liquidweb.com/kb/how-to-install-memcached-on-centos-7/)
   yum install zlib-devel
   1.1 [PHP memcache 설치](http://egloos.zum.com/jonnychoe/v/5561398)
       /home/bridgewar/util/memcache-2.2.6
       tar -xvf memcache-2.2.6.tgz
       phpize,  ./configure, make, make install

### mcrypt 설치
1. yum -y install php-mcrypt

### snappy 설치
yum install snappy
sudo 계정
cd php-ext-snappy-master(rpm -qa | grep snappy-1.1.0-3.el7.x86_64)
1. cd php-snappy
2. phpize
3. ./configure
4. make
5. make install

### /etc/php.ini 설정
extension=mcrypt.so
extension=snappy.so
extension=memcache.so
timezone 확인.

## 7. cron
## 8. back up
