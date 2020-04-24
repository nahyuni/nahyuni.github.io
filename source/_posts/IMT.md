---
title: 117회 기출문제
date: 2020-04-24 14:30:19
tags: []
categories: 
 - 정보 관리 기술사
permalink: ""
---
<!-- more -->
## 전체 문제 
 -  https://drive.google.com/file/d/1EDwMIPl66b2EfhRbGi0Aj9F5TNljsgp7/view

## 1. HMAC(Hash-based Message Authentication Code)
- 날짜 : 0404
- 레퍼런스 : https://www.joinc.co.kr/w/man/12/hmac
- 키워드 : Hash,Shared Key,MAC Algorithm
- 답안전략 : 정확한 메커니즘 설명 및 3 단락에서의 차별화 접근
<br>

### 개념
- 송신자와 수신자만이 공유하고 있는 Key 와 Message 를 혼합하여 Hash 값을 만드는 방식
- HMAC = Hash(Hash(message + key) + key)
<br>
<!-- more -->

### 특징
- 메시지의 무결성, 기밀성 보장
- 알고리즘: MD5, SHA1
- 분류: HMAC-MD5, HMAC-SHA1
- 채널을 통해 보낸 메시지가 훼손되었는지 여부를 확인하는데 사용할 수 있음
- MAC 의 특성상 역산이 불가능 하므로, 수신된 메시지와 Hash 값을 다시 계산하여, 계산된 HMAC 과 전송된 HMAC 이 일치하는지를 확인하는 방식

### 프로세스
1. 별도 채널을 이용하여 Sender 와 Receiver 는 해시에 사용할 키(shared key)를 공유.
2. 해시 알고리즘 선정
3. Sender 
 3.1 message = 전화번호 + 유저아이디;
 3.2 hash = SHA1(message + Shared Key), message 를 보낸다.
4. Receiver
 4.1 Sender에서 보낸 message 를 SHA1(message + Shared Key) 로 해서 Sender 의 해시와 
   Receiver 에서 계산한 해시값이 같은지 확인 후 같으면 인증 그렇지 않으면 assert 된다.

### 해킹 공격에 따른 방어 정책.   
1. 다시 보내는 replay 공격에 노출될 수 있으므로 1초에 10번 이상 들어오면 assert 처리함.

