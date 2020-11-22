# JWT (Json Web Token)

## JWT란 ?
- JWT 는 클라이언트와 서버, 서비스와 서비스 사이 통신시 권한 인가를 위해 사용하는 토큰
- URL 에 대해 안전한 문자열로 구성되어 있어 HTTP 어디든 위치할수 있다.

### JWT의 구조
- 헤더 (Header), 페이로드(Payload), 서명(Signature) 세부분을 점으로 구분하는 구조 
```
HEADER.PAYLOAD.SIGNATURE
```

#### HEADER
- JWT 를 검증하는데 필요한 정보를 가진 JSON 객체는 Base64 URL-Safe 인코딩된 문자열이다. 
- 헤더는, JWT를 **어떻게 검증(Verify)** 할것인가 에 대한 내용을 가지고 있다.

```
{
    "alg": "ES256",
    "kid": "Key ID"
}
```
> alg 는 서명시 사용 알고리즘, kid 는 서명시 사용하는 키를 식별하는 값이다.

#### PAYLOAD
- JWT의 본문이다.
- 페이로드에 있는 속성들을 클레임 셋 이라고 부른다.
- 클레임 셋은, JWT 에 대한 내용 이나 클라이언트와 서버간 주고받을 값들로 구성된다.

```
{
    "name": "ncucu",
    "age": 27
}
```

#### SIGNATURE
- 점을 구분자로 해서 헤더와 페이로드를 합친 문자열을 서명한 값이다.
- 서명은, 헤더의 alg 에 정의된 알고리즘과 비밀키를 이용해 생성하고, Base64 URL-Safe 로 인코딩한다.

#### JWT
- 점을 구분자로 해서 헤더, 페이로드, 서명을 모두 합치면 JWT 가 완성된다.
- 완성된 JWT는 헤더의 alg, kid 속성과 공개키를 이용해 검증할 수 있다.
- 서명 검증이 성공하면 JWT의 내용들을 신뢰할 수 있게 되고, 페이로드의 값으로 원하는 처리를 할 수 있다.

## 더 알아보기

#### JWT, JWS, JWE, JWK, JWA ?
- JWT를 이용한 서명, 암호화에 대한 명세는 JWT 하위, JWS(Json Web Signature), JWE(Json Web Encryption)에 되어 있다.
- 이해를 쉽게하기 위해 비유하면 JWT는 추상 클래스, JWS, JWE 는 이를 구현한 콘크리트 클래스이다.
- JWK(Json Web Key)는 JSON 형식으로 암호화 키를 표현한것, JWA(Json Web Algorithm)은 JWS, JWE, JWK 에 사용하는 알고리즘에 대한 명세

#### JWS, Compact Serialization
- 대부분 사용하는 JWT는 JWS 이고, 넓은 의미로 JWT 라고 표현한다.
- JWT의 구조인 헤더.페이로드.서명 구조는 JWS 의 직렬화 방법 중 하나인 Compact Serialization 형식으로 직렬화 한 것이다.

> JWT 는 JWS 를 사용하고, JWS Compact Serialization 으로 직렬화한 문자열이다.


#### Base64 URL-Safe
- Base64 URL-Safe 는 Base64 인코딩에서 + 는 - 로, / 는 _ 로 대체된 인코딩 방법이다.
- JWT 설계의도대로 어디에서도 사용 가능한 범용성을 가지게 되었다.

#### Header & Payload
- JWT 헤더는 Base64 인코딩 이전 항상 UTF-8 인코딩 문자열이여야 한다.
- 헤더는 반드시 JSON 이여야하고, JSON 의 기본 인코딩은 UTF-8 이기 때문이다.
- 정식 명칙은 JOSE(JSON Object Signing and Encryption) HEADER
- 페이로드는 헤더와는 다르게 Base64 URL-Safe 인코딩만 되어있다면 OK

#### Stateless
- JWT 는 JWT에 필욯나 모든 정보를 담을 수 있다.
- JWT 생성시 JWT에 대한 검증 혹은 인가시 필요한 값을 넣으면 되기 때문에 JWT에 대한 상태를 관리하지 않아도 된다.

#### 공개키 암호 방식에서 서명과 암호화
- JWT는 기본적으로 공개 키 암호방식 (PKC, Public Key Cryptography)을 사용한다.
- 비대칭 암호 방식을 사용해 공개키/비밀키를 생성하고 이 키를 활용하여 통신한다.
- 서명은 데이터 해싱 값을 비밀키로 서명하고, 다시 공개키로 서명을 검증한다.
    - 발행한곳 이외에서 복호화 가능
- 암호화는 공개키로 데이터를 암호화하고 비밀키로 데이터를 복호화 한다.
    - 인증 서버에서만 복호화 가능

## 사용 사례

#### JWT 와 API Key
- 애플 푸시 메시지 발송 API 인 APNs Provider API 는 2016년부터 인증을 위해 JWT 를 지원했다.
- JWT 를 API Key로 사용하면서, APNs 는 빠르게 인증을 할 수 있게 되었다.

#### JWT 와 MSA
1. Access Token in MSA
- 접근 권한에 대한 제어가 필요한 서비스는 사용지 인증 및 권한 인가가 필요하다.
- 보통 액세스 토큰은 권한을 가리키는 임의의 문자열로 구성되어 있는데, 권한을 참조한다는 의미로 참조 토큰 이라 부른다.
- 모놀리틱 아키텍쳐에서는 참조 토큰을 액세스 토큰으로 사용해도 문제가 없다.
- MSA 환경에서는 수많은 서비스간 API 호출이 일어나기 때문에 많은 서비스들이 권한 서비스와 통신 해야하고 이로인해 권한 서비스에 부하가 증가한다.

2. JWT as Access Token in MSA
- 참조 토큰 대신 JWT를 액세스 토큰으로 사용할 수 있다.
- JWT는 자체적으로 필요한 정보를 모두 담을수 있기 때문에 값 토큰 이라고 한다.
- JWT 액세스 토큰은 MSA 환경의 인증과 접근 제어에 적합하다.
- 모든 서비스는 JWT에 포함된 값을 기준으로 권한을 확인할 수 있다.
- 서비스와 권한 서비스의 통신은 JWT 서명을 인증하기위한 공개키 조회가 전부이다.
- 단점으로는 사용자 권한 혹은 정보가 변경될 때 마다 토큰으 재발행해야 하고, 토큰의 크기가 커질 수 있다.
- 또한 헤더나 페이로드는 디코딩시 바로 내용 확인이 가능하기 때문에 보안문제로 이어질 수 있다.

3. API Gateway between Access Token and JWT in MSA
- 클라이언트, 권한 서비스, 서비스 사이에 API Gateway를 위치시켜 JWT를 클라이언트에게 숨기면서 서비스간 통신 시 사용한다.
- API Gateway는 클라이언트에게 받은 액세스 토큰을 권한 서비스를 통해 JWT로 받아 액세스토큰 대신 서비스로 넘겨주어 서비스들 간 통신시에는
- JWT 를 사용해 위 문제를 보완한다.

## 참고
- https://meetup.toast.com/posts/239
- https://en.wikipedia.org/wiki/JSON_Web_Signature
- https://jose.readthedocs.io/en/latest/
- https://jwt.io
- https://datatracker.ietf.org/wg/jose/documents/
