### grpc_middleware
Ref: https://github.com/grpc-ecosystem/go-grpc-middleware

auth, logging 등 request를 가로채는 middleware 존재

### Buf
Ref : https://docs.buf.build/introduction
의존성 관리, generated code 관리 등 proto와 관련된 작업들 수행 및 lifecycle 관리 용이하게 만들어줌
- `buf.work.yaml` : proto workspace dir 정의
- `buf.gen.yaml` : protoc generation plugin을 통해 stub을 원하는 path에 generation


### Large binary file download
- Outgoing : No Limit
- Ingoing : 4MB Limit

memory에 response를 전부 받을 때까지 적재하기 떄문에 성능에 영향을 줄 수 있다.
chunking을 통해서 보내는 방법을 사용하거나 REST로 보내는게 낫다.
