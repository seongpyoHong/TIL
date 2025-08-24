보안 아키텍처에 대해서는 다음 3가지 개념을 이해해야 한다.
### Identity
User의 정보
- LDAP와 같은 데이터 저장소나 DB에 저장됨

Identity Provider: 시스템 사용자의 신원을 관리
- Authentication
- 사용자 정보를 Service Provider에 전달

Service Provider: Application 기능 + Authorization
### Authentication
Identity Provider가 담당
- 애플리케이션을 사용하려는 사용자가 실제로 본인이 맞는지 확인하는 과정
### Authorization
Service Provider가 담당
- 그 사용자가 허용된 작업만 수행하도록 제한하는 과정

### Flow
- Service Provider → Identity Provider: 사용자 자격 증명(아이디·비밀번호) 전달
- Identity Provider → Service Provider: 인증 결과와 사용자 정보 반환
- Service Provider: 사용자 역할(Role)/그룹(Group)에 따라 권한 부여


### LDAP 기반 아키텍처
**LDAP Identity Provider**를 사용하는 방식입니다.  
거의 모든 조직이 직원 정보를 사내 LDAP에 저장하므로, 애플리케이션을 LDAP과 직접 연동해 단일 ID(기업 AD)로 관리하는 것이 효율적입니다.

**구조**

- Identity: LDAP 저장소에 저장 (대부분 Active Directory)
- Authentication: LDAP Identity Provider가 처리
- Authorization: 여전히 애플리케이션이 처리 (역할/그룹 기반 접근 제어)

### 단점

1. **클라우드 환경 접근 제한**
    - 온프레미스 환경에서는 문제 없음 (AD와 애플리케이션이 동일 데이터센터)
    - 하지만 AWS, Google Cloud, Azure 등 외부 클라우드에 배포된 애플리케이션은 AD에 직접 접근 불가
    - 기업 방화벽과 보안 정책상 외부에서 AD로의 직접 연결은 허용되지 않음 → 보안 위험
        
2. **자격 증명 전달 문제**
    - 여전히 사용자 ID/PW가 애플리케이션을 거쳐 전달됨
    - 가장 안전한 방법은 자격 증명을 애플리케이션이 아닌 **Identity Provider**에 직접 보내는 것


###  SAML
- **LDAP 장점**: 사용자 정보를 한 곳에 모아 관리
- **LDAP 단점**
    1. 애플리케이션과 같은 데이터 센터에 있어야 함 → 클라우드 환경에 제약
    2. 사용자가 매번 자격 증명 입력 → 보안·편의성 문제
        
- **SAML 해결책**
    - **HTTP 리다이렉트**를 사용해 브라우저를 매개로 데이터 센터 간 인증 처리
    - **SSO(싱글 사인온)** 제공 → 회사 네트워크 로그인 상태를 인식하여 추가 인증 입력 없이 접근 가능

- **요청 흐름**
	- 사용자가 애플리케이션에 최초 요청을 보냅니다. (예: company.com/app)
	- 애플리케이션은 사용자가 로그인되어 있지 않다고 판단하고, SAML Identity Provider로 인증 요청을 보내기 위해 사용자 에이전트에 리다이렉트를 지시합니다.
	- 브라우저가 SAML Identity Provider에 요청을 전달합니다.
	- SAML Identity Provider는 사용자가 이미 네트워크에 로그인되어 있음을 인지하고, 자동으로 인증 처리 후 사용자 정보를 포함한 SAML 응답을 만듭니다.
	- SAML Identity Provider는 브라우저를 통해 다시 애플리케이션으로 리다이렉트하면서 SAML 응답(클레임 포함)을 전달합니다.
- **한계**
	- SAML은 중앙화된 사용자 인증과 SSO에 적합하지만, REST API 보안 처리에는 한계가 있음
	- 마이크로서비스 아키텍처에서 각 서비스별 인증 문제 해결이 어렵고, 토큰 전달 방식도 비효율적
	- 분산된 클라우드 환경과 예약 작업 등 비사용자 호출에도 안전한 인증·인가 체계가 필요
	- OAuth 같은 새로운 프레임워크가 이런 문제를 해결하기 위해 등장함
### SSO
**한 번 로그인하면 여러 시스템이나 애플리케이션을 추가 로그인 없이 바로 이용할 수 있게 해주는 인증 방식**


---
 **OAuth 2.0**은 서드파티 애플리케이션이 사용자의 허락을 받아 제한된 범위 내에서 HTTP 서비스(예: 소셜 미디어 API)에 접근할 수 있도록 만든 권한 프레임워크다.    
- 소셜 미디어 앱(LinkedIn 등)은 OAuth 2.0 권한 서버를 사용해 API 접근을 보호한다.
- 주요 역할:
     - **리소스 소유자**: 서비스 계정을 가진 사용자       
    - **서드파티 애플리케이션**: API에 접근하려는 외부 앱 (예: 학습 앱)
    - **리소스 서버**: API를 제공하는 서비스 (예: LinkedIn API)
    - **권한 서버**: 사용자 승인을 관리하는 서버 (LinkedIn 내부에 위치)
- 서드파티 앱은 권한 서버에서 받은 **액세스 토큰**을 사용해 제한된 시간 동안 API에 접근할 수 있다.
- OAuth 2.0은 기업 환경뿐 아니라 소셜 미디어 환경에도 적합하다.

### 역할
- **Resource Owner (자원 소유자)**: 제가 Shutterfly 애플리케이션을 사용하는 사용자이므로 저(사용자)가 Resource Owner입니다.
- **Resource (자원)**: 구글 포토의 내 사진들입니다. 페이스북 자원은 내 페이스북 계정입니다. 
- **User Agent (사용자 에이전트)**: 여기서는 브라우저이며, OAuth 흐름에서 매우 중요한 역할을 합니다. SAML 다이어그램에서 보았듯 인증 과정에서 HTTP 요청을 리다이렉트하는 역할을 합니다.
- **Resource Servers (자원 서버)**: 두 개가 있습니다. 구글 포토 리소스 서버와 페이스북 리소스 서버로 각각 자신의 리소스를 보호하며, REST API 형태로 동작합니다.
- **Client (클라이언트)**: 서드파티 애플리케이션인 Shutterfly 클라이언트 애플리케이션입니다. 이 애플리케이션이 구글 포토 API를 호출하여 사진을 가져오고, 페이스북 리소스 서버를 호출하여 사용자의 페이스북 페이지에 사진을 게시합니다.
- **Authorization Servers (권한 부여 서버)**: 두 개가 있습니다. 구글 권한 부여 서버와 페이스북 권한 부여 서버입니다. Shutterfly는 두 서버 모두와 상호작용해야 합니다. 이 서버들은 Resource Owner로부터 권한을 얻고, 클라이언트에게 액세스 토큰을 제공합니다.

### Client 등록
- OAuth에서 클라이언트가 권한 부여 서버에 접근하려면 **클라이언트 등록**이 필수
- 등록 시 애플리케이션 이름과 특히 **리다이렉트 URI**를 권한 부여 서버에 등록
- 등록 후 클라이언트 아이디와 시크릿을 받으며, 이 정보로 클라이언트 신원 증명
- 클라이언트 아이디와 시크릿은 안전하게 보관해야 하며, 요청 시 함께 전송
- Shutterfly처럼 여러 권한 부여 서버를 사용할 땐 각각 등록 필요

### Token
- 리소스 서버는 클라이언트의 REST API 호출 시 **베어러 토큰**을 통해 요청 유효성을 확인
- 클라이언트는 권한 부여 서버에서 **액세스 토큰**을 받아서 API 호출 시 HTTP Authorization 헤더에 포함시켜 전달
- 리소스 서버는 토큰의 의미를 알지 못해, 권한 부여 서버에 토큰 검증 요청을 보냄
- 권한 부여 서버는 토큰 정보(유효 기간, 권한 범위 등)를 리소스 서버에 알려주고, 이를 통해 요청을 허용 또는 거부
- 토큰은 불투명한 문자열(opaque token)이며, 권한 부여 서버가 내부에서 매핑 관리
- 토큰 만료 후 재발급 필요
- 매 요청마다 토큰 검증이 필요해 대규모 요청 시 성능 문제가 생길 수 있어 구조화된 토큰이 필요
- 구조화된 토큰(대표적으로 JWT)은 토큰 자체에 정보와 서명이 포함된 형태
- 권한 부여 서버가 비밀 키로 서명하고, 리소스 서버가 공개 키로 검증
- 리소스 서버가 매번 권한 서버에 토큰 검증 요청할 필요가 없어 성능상 유리
- JWT 안에 만료 시간, 권한 범위, 사용자 정보 등 주요 데이터 포함
- 토큰 검증 후 리소스 서버는 내부 권한(Authorization) 판단 가능


### Scope
- **스코프(scope)**는 액세스 토큰이 허용하는 권한의 범위
- 사용자 허락에 따라 토큰에 특정 스코프가 포함됨
- API 호출 시 토큰 내 스코프와 API가 요구하는 스코프가 일치해야 요청 승인
- 스코프 불일치 시 API는 요청 거부

### Endpoint
- OAuth에서 중요한 여러 HTTP 요청 URL을 **엔드포인트(endpoints)**라 부름
- 주요 엔드포인트:
    - Authorization Endpoint (권한 요청)
    - Token Endpoint (토큰 발급)
    - Introspect Endpoint (토큰 검증, Opaque 토큰용)
    - JWKS Endpoint (JWT 토큰 서명 검증용 공개키 제공)
    - UserInfo Endpoint (사용자 정보 제공)

### OAuth Grant Types (인가 방식)
#####  1. **클라이언트(Client)의 정의**
- OAuth에서 클라이언트는 **Access Token을 요청하는 애플리케이션**을 의미합니다. 
- 서버 백엔드에서 동작하는 애플리케이션일 수도 있고,
- 브라우저에서 실행되는 AJAX SPA,
- 모바일 앱(iOS, Android),
- 데스크톱 앱도 가능합니다.
- 그래서 다양한 아키텍처 환경을 고려해야 하고, 그에 맞게 여러 **Grant Type**이 존재합니다.

##### 2. **Authorization Code Grant (권한 코드 인가)**
- 가장 일반적이고 안전한 흐름
- 클라이언트는 `Authorization Code`를 받은 후, 토큰 엔드포인트에 그 코드를 보내서 Access Token을 받음
- **Confidential Client** (서버에 위치하며 Client ID와 Client Secret을 안전하게 보관 가능) 대상
- 액세스 토큰을 직접 프론트엔드로 노출하지 않아 보안상 안전
- **왜 두 단계인가?**
    - 액세스 토큰을 프론트엔드(브라우저)에서 직접 받으면 탈취 위험 있음
    - 대신 단시간만 유효한 `Authorization Code`를 주고, 이를 백엔드에서 액세스 토큰으로 교환
        
- **PKCE (Proof Key for Code Exchange)**
    - 공개 클라이언트(브라우저, 모바일 앱 등)에서는 Client Secret 보관이 불가능해 이를 보완하기 위한 확장
    - 동적으로 생성된 `code_verifier`와 `code_challenge`를 사용하여 인증 코드 교환 시 보안을 강화
    - 점점 표준으로 자리 잡고 있음

### 3. **Implicit Grant (암묵적 인가) — Deprecated**
- 이전에 공개 클라이언트에서 Access Token을 바로 받아서 사용하는 방식
- 보안 문제로 인해 더 이상 권장하지 않고 폐기됨
- Access Token이 프론트엔드에 바로 노출되어 탈취 위험이 큼
- 지금은 Authorization Code Grant + PKCE를 사용해야 함
### 4. **Client Credentials Grant**
- 리소스 오너(사용자)가 없는 경우 사용
- 예: 서버에서 실행되는 스케줄링 작업(크론 잡)이 주기적으로 API 호출할 때
- 클라이언트가 Client ID, Client Secret을 가지고 직접 토큰을 받아 사용
- 가장 단순한 형태의 흐름

### 5. **Resource Owner Password Credentials Grant — Deprecated**
- 사용자가 직접 클라이언트 애플리케이션에 사용자명/비밀번호를 제공하는 방식
- 보안상 문제로 제일 권장하지 않음
- 오직 클라이언트와 Resource Server가 같은 조직에 속하고 신뢰 관계가 있을 때만 제한적으로 사용
- 일반적으로는 Authorization Code Grant로 대체
---

### 6. **Refresh Token**
- Access Token은 만료 시간 있음 (예: 구글 1시간)
- 사용자가 긴 시간 동안 앱을 사용할 때 재로그인 없이 계속 접근 가능하도록 Refresh Token 발급
- Refresh Token은 Access Token보다 만료 기간이 훨씬 김(구글은 사실상 만료 없음)
- 만료된 Access Token을 Refresh Token으로 새 토큰 발급 요청 가능
---

### 7. **토큰 폐기 (Revocation)**
- 사용자가 로그아웃할 때 Access Token과 Refresh Token을 무효화 해야 함
- Authorization Server가 **Revocation Endpoint**를 제공
- 클라이언트는 로그아웃 시 해당 엔드포인트를 호출해 토큰 폐기 요청
- 서버마다 동작이 조금씩 다르니 문서 확인 필요
---

### 8. **실제 프로그래밍은?**
- 많은 HTTP 메시지를 직접 다룰 필요 없이
- 다양한 언어와 프레임워크가 OAuth 라이브러리를 제공해 대부분 자동 처리
- 하지만 내부 동작 원리를 이해하는 것이 문제 해결과 디버깅에 중요

| 사용 사례                             | 권장 Grant 타입                    |
| --------------------------------- | ------------------------------ |
| AJAX (싱글 페이지 앱)                   | Authorization Code + PKCE      |
| 전통적인 웹 애플리케이션 (JSF, Spring MVC 등) | Authorization Code (가능하면 PKCE) |
| 모바일 애플리케이션 (iOS, Android)         | Authorization Code + PKCE      |
| 스케줄러나 백엔드 작업(사용자 없음)              | Client Credentials             |
| 데스크탑 GUI / CLI 애플리케이션             | Authorization Code + PKCE      |


요약하자면, 지금까지 배운 OAuth 2.0은 주로 **리소스 API 접근을 위한 액세스 토큰**을 클라이언트가 받는 흐름이었어요. 이 토큰은 권한(스코프)만 담고 있고, 사용자 정보는 거의 포함하지 않죠. 그래서 액세스 토큰 자체에는 사용자에 대한 상세 정보가 없고, ‘sub’(주체, 사용자 식별자) 정도만 있을 뿐입니다.


## OpenID Connect (OIDC)란?

- OAuth 2.0에 **사용자 인증과 사용자 정보 제공** 기능을 추가한 표준 프로토콜입니다.
- 액세스 토큰과 함께 **ID 토큰(id_token)** 을 반환하는데, 이 토큰에는 사용자의 정보(이메일, 이름, 프로필 등)가 포함되어 있어요.
- 사용자 인증 및 프로필 정보를 클라이언트에게 전달하는 데 특화되어 있습니다.
---
## OIDC 동작 방식 (OAuth 2.0과 연동)

1. 클라이언트가 인가 요청 시, OAuth 스코프에  
    `openid`, `profile`, `email`, `address`, `phone` 같은 OpenID Connect 전용 스코프를 포함시킵니다.
    
2. 인가 코드 교환 시, Authorization 서버는 액세스 토큰과 함께  
    **ID 토큰(id_token)** 을 클라이언트에 반환합니다.
    
3. 클라이언트는 ID 토큰을 해석하여 사용자 정보를 얻을 수 있고,  
    추가로 사용자 정보가 필요하면 **UserInfo Endpoint** 를 액세스 토큰으로 호출해 사용자 정보를 받아옵니다.
## 주요 OpenID Connect 스코프

| 스코프     | 설명                      |
| ------- | ----------------------- |
| openid  | OpenID Connect 사용 의사 표시 |
| profile | 사용자 기본 프로필 정보 요청        |
| email   | 이메일 주소 및 이메일 인증 여부 요청   |
| address | 주소 정보 요청                |
| phone   | 전화번호 요청                 |
