---
title: [consul]
date: 2020-04-14 14:30:19
tags: [microservice]
categories: 
    - 개발
thumbnail: ""
permalink: ""
---

### - 사이드카 프록시 로드 밸런서는 Envoy, NGINX, HAProxy, Linkerd입니다. 
- nginx
https://learn.hashicorp.com/nomad/load-balancing/nginx
- Envoy
https://bcho.tistory.com/1297?category=731548

<!-- more -->

### - consul 프록시 설정 방법
1. 서비스 띄움.
```
$ ./stringsvc **-listen :8081 **
```

2. 서비스 등록.
```
$ vi consul/consul.d/stringsvc.json
{
    "service": {
      "name": "stringsvc",
      "port": 8081,
      "connect": { "sidecar_service": {} }
    }
}
```

3. consul dev로 띄움.
```
$ ./consul agent -dev -config-dir=./consul.d
```

4. 사이드카 등록 및 서비스와 연결.
```
$ ./consul connect proxy -sidecar-for stringsvc
```

5. 프록시 등록 및 사이드 카와 연결.
```
$ ./consul connect proxy -service proxy_stringsvc -upstream stringsvc:9191
$ ./consul connect proxy -service proxy_stringsvc2 -upstream stringsvc:9192
```

$ curl -XPOST -d'{"s":"mytest"}' http://localhost:9191/stringsvc/count
$ curl -XPOST -d'{"s":"mytest"}' http://localhost:9192/stringsvc/count
$ curl -XPOST -d'{"s":"mytest"}' http://localhost:8081/stringsvc/count

- 다른 방식

1. 서비스 등록.
```
{
    "service": {
      "name": "stringsvc",
      "port": 8081,
      "connect": {
        "sidecar_service": {
          "proxy": {
            "upstreams": [{
               "destination_name": "stringsvc",
               "local_bind_port": 9191
            },
            {
               "destination_name": "stringsvc",
               "local_bind_port": 9192
            }
            ]
          }
        }
      }
    }
}
```
```
./consul connect proxy -sidecar-for stringsvc
```
- 에러뜸(확인중)


### consul 참조
- https://www.consul.io/docs/connect/registration/service-registration.html
- https://velog.io/@tedigom/MSA-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-4Service-Mesh-f8k317qn1b
- https://www.slideshare.net/SreenivasMakam/service-discovery-using-etcd-consul-and-kubernetes
<br>
