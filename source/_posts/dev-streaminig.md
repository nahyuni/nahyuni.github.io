---
title: [streaming 기술 이해]
date: 2020-06-24 14:30:19
tags: [streaming]
categories: 
    - 개발
thumbnail: ""
permalink: ""
---
### 1. Streaming Service 개요

영상 전송 방식은 크게 다운로드 방식 프로그레시브 다운로드 (Prog ressive Download)방식, 스트리밍(Streaming ) 방식으로 구분한다.

#### 1.1 Streaming 이란?
 - 인터넷에서 데이터를 실시간 전송, 구현할 수 있게 하는 기술.
 - 스트리밍은 '흐르다', '흘러내리다', '연속되어 끊이지 않고 흐르다' 등의 의미로,
데이터가 조금씩 연속적으로 흘러나온다는 개념과 연관.
즉, 어떤 사운드나 비디오 파일을 하나의 형태가 아닌 여러 개의 파일로 나누어
물 흐르듯이(Streaming) 연이어 보내는 것.
<!-- more --> 
 - 동영상 파일 등은 용량이 크기 때문에 한꺼번에 파일 전체를 보내주기란 힘들다.
이를 해결하기 위해서 조금씩 파일의 일부만(실제로 영상이 플레이되는 분량만
큼만) 실시간으로 전송해 주는 것이 '스트리밍'이다.
 - 사용자가 접속하고 있는 인터넷 네트워크의 속도에 맞춰 비교적 큰 크기의 스트
리밍 파일은 아주 작은 크기의 조각들로 나누어진다. 이 조각들은 각각 뒤의 조
각들과 이어질 수 있는 헤더정보를 가지고 전송된다. 구동 프로그램에서는 이
조각들을 받으면서 동시에 압축을 풀어 동영상이나 음성으로 재생하게 되는 것.
 - 인터넷에서 동영상을 다운로드가 아닌 실시간 전송에, 실시간 구현이라는 개념을
가능케한 것이 바로 스트리밍 기술이다.

#### 1.2 Progressive Download
 - 동영상 파일이 서버로부터 클라이언트에 전달될 때 파일 일부가 도착하는 대로
  먼저 재생하는 방법
 - 512Kbps의 동영상이라면 적어도 512Kbps의 네트웍 속도가 제공되어야만 동영상이 끊임 없이 제공됩니다.

## Streaming Service 종류
스트리밍 서비스는 크게 영상이나 음성을 받아 서비스하는 라이브 방송과 이미
저장된 멀티미디어 데이터를 스트리밍 하는 주문형 서비스로 구성될 수 있다.

### VoD Streaming 
### Live Streaming
 - 비디오와 오디오를 실시간으로 인코딩해 많은 사용자에게 동시에 제공
### HTTP Live Streaming
 - 라이브 스트리밍을 위한 전통적인 프로토콜로인 RTP/RTSP는 도입 비용이 높고
방화벽 환경에서 서비스가 원활하게 이루어지지 않는 단점이 있다. 이를 보완하
기 위해 **HTTP를 이용해 원활한 스트리밍 서비스가 가능하도록 Apple이 개발**
HTTP 라이브 스트리밍(HTTP Live Streaming, HLS)은 **가장 대중적인 스트리밍 포맷**으로 간주된다
**HLS는 전반적인 스트림을 크기가 작은 일련의 HTTP 기반 파일 다운로드로 분리시켜 개개의 다운로드를 통해 잠재적으로 무한한 전송 스트림의 작은 덩어리를 적재시킴으로써 동작한다**



## Streaming 서비스 구성 요소
1. Server side
 - Streaming Server(Encoding 포함)
 - Web Server
2. Client side
 - Streaming Client

## Streaming 서비스 동작 원리
사용자는 웹서버에 요청하고, 스트리밍 서버는 사용자가 요청한 사항에 대하여
멀티미디어 데이터를 보낸다.
인코딩 서버는 캠코더나 마이크 등의 장비로부터 받아들이는 아날로그 데이터를
압축 기술을 사용하여 디지털 데이터로 바꾸는 기능을 한다.

## Streaming 관련 기술
멀티미디어 스트리밍 기술을 이루는 세 축은 압축 기술, 네트워크 기술, QoS 기술이다. 이 중 압축기술과 네트워크 기술은 없어서는 안 될 중요한 요소이다.

### 데이터 전송 및 제어 프로토콜
RTP(Real-time Transport Protocol)
RTCP(Real-Time Transport Control Protocol)
RTSP(Real Time Streaming Protocol)
MMS(Microsoft Media Server) - 마이크로소프트 네트워크 스트리밍 프로토콜
RTMP(Real Time Messaging Protocol) - 어도비 시스템즈사의 독점 컴퓨터 통신 규약

### codec(데이터 압축)
* WMV/WMA
* H.264(MPEG-4/AVC)
* MPEG2-TS(Transport Stream)

### library
- libavcodec
1. libavcodec은 자유 소프트웨어이자 LGPL 라이선스가 걸린 오픈 소스 라이브러리
2. 이 코덱은 영상 데이터와 음성 데이터를 인코딩하고 디코딩할 수 있다.

### Streaming 지원 포맷
- Container Format
1. 다양한 종류의 데이터 및 표준 데이터 압축 방법을 사용하여 압축된 데이터를 저장할 수 있는 파일 형식
2. 컨테이너 파일은 다양한 종류의 데이터를 확인하고 정리할 수 있다.
3. 컨테이너 포맷은 여러 스트리밍을 재생 치유하는 데 필요한 동기화 정보와 함께 음성, 동영상, 자막, 장( 챕터 ), 자막, 메타 데이터( 태그 ) 등을 지원한다.
4. 확장자 : mp4, mov, mpg, mpeg, wmv, wma, swf, asf, ksg, flv, mkv, tp, ts, rm, mts


## Streaming Server/Client

### Streaming Server

- RTP Server
RTP로 미디어 패킷들을 클라이언트에게 전송하는 역할을 한다. RTCP를 통해 클
라이언트의 수신 대역폭에 따라 전송률을 조정한다.

- RTSP Server
RTSP를 사용해서 사용자와 통신하는 역할을 한다. RTP가 단방향 통신인데 비
해, RTSP는 양방향 통신이다. 사용자는 RTSP를 통해 비디오 요청, 스킵, 멈춤과
같은 기능을 사용할 수 있다.

- Media Packetizer
오디오나 비디오와 같은 미디어파일을 그 파일의 포맷에 따라 네트워크로 전송
할 수 있는 패킷으로 쪼개는 역할을 한다.

- Packet Builder
Media Packetizer에서 쪼갠 미디어 데이터를 RTP 패킷으로 만드는 역할을 한다.

### open source
1. Darwin Streaming Server
- 애플의 소스 공개 정책에 따라 윈도우즈와 매킨토시 양쪽을 모두 지원
- 소스까지 공개되어 있기 때문에 누구든지 무료로 설치 가능
- 동영상, MP3 등의 디지털미디어를 실시간으로 배포하고 라이브 이벤트를 실현시킬 수 있으며, Linux, Solaris, Windows NT/2000 등 가장 대중적인 엔터프라이즈급 플랫폼을 지원
2. FFmpeg
- FFmpeg은 명령어를 직접 입력하는 방식으로 동작하며 여러가지 자유 소프트웨어와 오픈 소스 라이브러리로 구성되어 있다.
- 라이브러리 중에는 libavcodec도 들어있으며 또, libavformat 라는 음성/영상 다중화, 역다중화 라이브러리도 있다.

### Streaming Client
* GStreamer, Media Player Classic, MPEG4IP, MPlayer, QuickTime

## Streaming 업체 및 기술 동향
### 국내 Streaming 개발 업체
* 솔루션박스, CDNetworks, 나우콤, 피어링 포털
### 스트리밍 호스팅
Gabia, 카페24, Godo, inames

## 참조
https://linuxism.ustd.ip.or.kr/1267
https://d2.naver.com/helloworld/7122 - HTTP Live Streaming























