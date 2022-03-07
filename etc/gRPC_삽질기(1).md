### ProtoBuf guide (proto3)

[https://developers.google.com/protocol-buffers/docs/proto](https://developers.google.com/protocol-buffers/docs/proto3#oneof)3

### Service Definition

[https://grpc.io/docs/what-is-grpc/core-concepts/#service-definition](https://grpc.io/docs/what-is-grpc/core-concepts/#service-definition)

streaming은 client, server side인지에 따라 `stream` 키워드 위치가 달라짐

```proto
### Client streaming
rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);

### Server streaming
rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);
```

- Server side
    
    Service에 정의돈 method 구현 및 gRPC server 실행
    
    decode incoming request ⇒ excute service method ⇒ encode response
    
- Client side
    
    stub (=각자의 언어로 service와 같은 메서드를 구현하고 있는 객체)에 method 호출 
    
    ⇒ server에 
    

### Generate Plugin

.proto에 맞춰 코드를 생성해주는 플러그인 (e.g. `protoc-gen-go` , `protoc-gen-go-grpc`)

`protoc-gen-go` generate plugin으로 생성되는 목록

- unary method
- server-streaming method
- Client-streaming method
- Bidi-streaming method

### gRPC Web?

gRPC를 브라우저에서 사용 가능하도록 한 js 구현체

Envoy 같은 프록시를 통해서 gRPC service와 통신해야함 (브라우저에서 HTTP/2를 지원하지 않기 때문에?)

### grpc-web 다운

```solidity
brew install protoc-gen-grpc-web
```
