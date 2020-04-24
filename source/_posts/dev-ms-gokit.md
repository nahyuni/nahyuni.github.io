---
title: [go-kit]
date: 2020-04-06 14:30:19
tags: [microservice]
categories: 
    - 개발
thumbnail: ""
permalink: ""
---

# reference
- http://gokit.io/examples/ 
- https://medium.com/@shijuvar/go-microservices-with-go-kit-introduction-43a757398183
- https://github.com/go-kit/kit

<!-- more -->
# Go kit Architecture
- Transport layer
 1. HTTP or gRPC, or using a pub/sub systems like NATS
 2. you can provide multiple transports for a same service
- Endpoint layer(추상화)
 1. Each service method in a Go kit service converts to an endpoint to make RPC style communication between servers and clients
 2. Each endpoint exposes the service method to outside world using Transport layer by using concrete transports like HTTP or gRPC
- Service layer
 1. The business logic
 2. Go kit service can be exposed by using multiple transports.

- Service layer
```
type StringService interface {
	Uppercase(string) (string, error)
	Count(string) int
}

type stringService struct{}

func (stringService) Uppercase(s string) (string, error) {
	if s == "" {
		return "", ErrEmpty
	}
	return strings.ToUpper(s), nil
}
```

- Endpoint layer
 1.controller 역할(request, service 호출, response)
 2.type Endpoint func(ctx context.Context, request interface{}) (response interface{}, err error)
```
func makeUppercaseEndpoint(svc StringService) endpoint.Endpoint {
return func(ctx context.Context, request interface{}) (interface{}, error) {
  req := request.(uppercaseRequest)
  v, err := svc.Uppercase(req.S)
  if err != nil {
    return uppercaseResponse{v, err.Error()}, nil
  }
  return uppercaseResponse{v, ""}, nil
}
}
```
- Transport layer
 1.어떻해 전달할 것인가.
``` 
 uppercaseHandler := httptransport.NewServer(
	makeUppercaseEndpoint(svc),
	decodeUppercaseRequest,
	encodeResponse,
)

http.Handle("/uppercase", uppercaseHandler)
```
- 미들웨어
``` 
type logmw struct {
	logger log.Logger
	StringService
}

func (mw logmw) Uppercase(s string) (output string, err error) {
	defer func(begin time.Time) {
		_ = mw.logger.Log(
			"method", "uppercase",
			"input", s,
			"output", output,
			"err", err,
			"took", time.Since(begin),
		)
	}(time.Now())

	output, err = mw.StringService.Uppercase(s)
	return
}
```

##  2. apigateway
### 1. Starting Addsvc
- go run addsvc.go -debug.addr :7080 -http-addr :7081 -grpc-addr :7082 -thrift-addr :7083 -jsonrpc-addr :7084
- curl -XPOST -d'{"a":"7","b":"4"}' http://localhost:7081/concat {"v":"74"}

### 2. Starting Stringsvc(stringsvc3 example)
- go run *.go -listen :8081
- curl -XPOST -d'{"s":"mytest"}' http://localhost:8081/count {"v":6}

### 3. Starting Consul
- First, you need to download Consul for your specific OS/architecture here.
 (https://www.consul.io/downloads.html)
- Next
```
$ mkdir ./consul.d
$ echo '{"service": {"name": "addsvc", "tags": [], "port": 7082}}' > ./consul.d/addsvc.json
$ echo '{"service": {"name": "stringsvc", "tags": [], "port": 8081}}' > ./consul.d/stringsvc.json
$ ./consul agent -dev -config-dir=./consul.d
```

### 4. Starting Api Gateway
$ go run main.go -consul.addr :8500

```
$ curl -XPOST -d'{"a":"7","b":"4"}' http://localhost:8000/addsvc/concat
{"v":"74"}
$ curl -XPOST -d'{"a":7,"b":4}' http://localhost:8000/addsvc/sum
{"v":11}
$ curl -XPOST -d'{"s":"mytest"}' http://localhost:8000/stringsvc/count
{"v":6}
$ curl -XPOST -d'{"s":"mytest"}' http://localhost:8000/stringsvc/uppercase
{"v":"MYTEST"}
```
- apigateway 는 consul 서버에 주기적으로 붙어서 서비스 이름으로 서비스 엔드 포인트 정보를 가지고 온다.
- 클라이언트 호출시에는 직접 서비스 엔드 포인트에 접속해서 데이터를 가지고 온다.
- 소스는 직접 확인.

## 참조
https://medium.com/@jwenz723/go-kit-api-gateway-4bee401e77a2

##  3. addsvc
 - 아래와 같이 패키지 되어 있다.

### 1. pkg 
 - addservice(service.go, middleware.go)
 - addendpoint(set.go, middleware.go)
 - addtransport(grpc.go, http.go, jsonrpc.go, thrift.go)

### 2. addsvc/cmd/addsvc/addsvc.go(사용방법)
```
        var (
                service        = addservice.New(logger, ints, chars)
                endpoints      = addendpoint.New(service, logger, duration, tracer, zipkinTracer)
                httpHandler    = addtransport.NewHTTPHandler(endpoints, tracer, zipkinTracer, logger)
                grpcServer     = addtransport.NewGRPCServer(endpoints, tracer, zipkinTracer, logger)
                thriftServer   = addtransport.NewThriftServer(endpoints)
                jsonrpcHandler = addtransport.NewJSONRPCHandler(endpoints, logger)
        )

```
### 3. addsvc/cmd/addsvc/addcli.go(사용방법)
```
svc, err = addtransport.NewHTTPClient(*httpAddr, otTracer, zipkinTracer, log.NewNopLogger())
svc = addtransport.NewGRPCClient(conn, otTracer, zipkinTracer, log.NewNopLogger())
```
## 4. profilesvc
- 메모리에 저장, 삭제, 조회 함.
```
 type Service interface {
	PostProfile(ctx context.Context, p Profile) error
	GetProfile(ctx context.Context, id string) (Profile, error)
	PutProfile(ctx context.Context, id string, p Profile) error
	PatchProfile(ctx context.Context, id string, p Profile) error
	DeleteProfile(ctx context.Context, id string) error
	GetAddresses(ctx context.Context, profileID string) ([]Address, error)
	GetAddress(ctx context.Context, profileID string, addressID string) (Address, error)
	PostAddress(ctx context.Context, profileID string, a Address) error
	DeleteAddress(ctx context.Context, profileID string, addressID string) error
}

$ go run ./cmd/profilesvc/main.go -http.addr :8080
$ curl -d '{"id":"1234","Name":"Go Kit"}' -H "Content-Type: application/json" -X POST $ http://localhost:8080/profiles/
$ curl localhost:8080/profiles/1234
ts=2020-04-06T02:30:46.441838639Z caller=middlewares.go:36 method=GetProfile id=1234 took=1.354µs err=null
{"profile":{"id":"1234","name":"Go Kit"}}
```
/cmd/profilesvc/main.go
```
func main() {
  var (
          httpAddr = flag.String("http.addr", ":8080", "HTTP listen address")
  )
  flag.Parse()

  var logger log.Logger
  {
          logger = log.NewLogfmtLogger(os.Stderr)
          logger = log.With(logger, "ts", log.DefaultTimestampUTC)
          logger = log.With(logger, "caller", log.DefaultCaller)
  }

  var s profilesvc.Service
  {
          s = profilesvc.NewInmemService()
          s = profilesvc.LoggingMiddleware(logger)(s)
  }

  var h http.Handler
  {
          h = profilesvc.MakeHTTPHandler(s, log.With(logger, "component", "HTTP"))
  }

  errs := make(chan error)
  go func() {
          c := make(chan os.Signal)
          signal.Notify(c, syscall.SIGINT, syscall.SIGTERM)
          errs <- fmt.Errorf("%s", <-c)
  }()

  go func() {
          logger.Log("transport", "HTTP", "addr", *httpAddr)
          errs <- http.ListenAndServe(*httpAddr, h)
  }()

  logger.Log("exit", <-errs)
}

```




















