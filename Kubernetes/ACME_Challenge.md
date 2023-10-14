ACME CA 서버가 클라이언트가 인증서를 요청하는 도메인을 소유하고 있는지 확인하려면 클라이언트는 "챌린지"를 완료해야 한다. 
- 이는 클라이언트가 소유하지 않은 도메인에 대한 인증서를 요청할 수 없고 결과적으로 다른 사람의 사이트를 사칭하는 것을 방지하기 위함
- 2가지 종류의 Challenge 챌린지가 있다.
	- HTTP 01 Challenge
	- DNS 01 Challenge

### HTTP 01 Challenge
가장 많이 사용하는 Challenge 유형
사용자 webserver의 root dir에 token을 포함한 file을 배치하라는 요청

1. Let's Encrypt는 ACME Client에 token을 제공
2. ACME Client는 `http;//<MY_DOMAIN>/.well-known/acme-challenge/<TOKEN>` 의 웹 서버에 파일(token + key의 fingerprint)을 저장
3. ACME Client가 Let's Encrypt에 파일이 준비되었음을 알리면 파일 검색을 시도 (유효성 검사)
4. 유효성 검사가 웹 서버로 부터 올바른 응답을 받으면 성공으로 간주하고 계속해서 인증서 발급

=> 웹 서버가 해당 파일을 서빙할 수 있는지 확인하여 도메인을 소유하고 있는지 확인하는 것

- cons
	- port 80이 차단되어 있으면 작동 X
	- internet에서 접근 가능해야함 (k8s)
	- wildcard 인증서는 발급 불가
	- 웹 서버가 여러 개인 경우 모든 서버에서 파일을 사용할 수 있는지 확인해야 함

### DNS 01 Challenge
HTTP 01 Challenge의 경우 webserver에 종속되기 때문에 커버하지 못하는 경우도 존재

- Let's Encrypt가 ACME Client에 token을 제공하면 해당 토큰과 키에서 파생된 TXT 레코드를 생성 (`_acme-challenge.<MY_DOMAIN>`)
- Let's Encrypt는 해당 레코드에 대해 DNS 쿼리
- 일치하는 항목을 찾으면 인증서 발급을 진행

=> 도메인의 DNS 설정을 변경 및 제어할 수 있는지 확인하는 것

- pros
	- wildcard 인증서 발급 가능
	- webserver가 internet에서 접근 불가능해도 상관 X (DNS Provider만 확인하기 때문에)
	- 웹 서버 개수와 무관
- cons
	- DNS API가 제공되어야만 사용 가능

### Cert-Manager Webhook based DNS 01 Challenge
- https://github.com/cert-manager/webhook-example
	- custom webhook을 통해 TXT record 생성을 포함한 DNS01 Challenge solving logic을 custom하게 작성할 수 있음
