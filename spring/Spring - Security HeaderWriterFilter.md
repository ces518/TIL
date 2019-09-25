# Spring Security - HeaderWriterFilter
- Security FilterChain에서 3번째로 동작하는 SecurityFilter이다.
- 크게 신경쓸일이 없는 필터이다.
- 특별한 설정이 필요없음.

#### HeaderWriterFilter
- 응답 헤더에 시큐리티 관련 헤더를 추가해주는 역할을 담당한다.
- 기본적으로 5개의 헤더 Writer가 적용된다.
- 1. XContentTypeOptionsHeaderWriter: MIME TYPE SNIFFING ATTACK PROTECITON
    - 브라우저가 컨텐츠 타입을 알아내기위해 MIME타입을 분석하는 경우가 있음.
    - 실행할수 없는 MIME타입임에 불구하고 실행하려고 시도하는 과정에서 보안 관련 문제가 존재함.
        - 뭔가를 실행한다 -> 브라우저가 다운로드를 받는 동작 등
    - X-Content-Type-Options: nosniff 헤더가 존재하면 반드시 Content-Type으로만 랜더링 하게끔한다.
    
- 2. XXssProtectionHeaderWriter: 브라우저에 내장된 XSS 필터 적용
    - XSS어택을 방어해준다.
    - 모든 XSS Attack을 방어해주진 못한다.
    - X-XSS-Protection: 1; mode=block; 헤더가 존재할 경우 활성화하는 옵션이다.
    - Naver의 Lucy-Filter등을 추가 적용하는것을 추천함.

- 3. CacheControlHeadersWriter
    - Cache 사용하지 않도록 설정한다.
    - Cache를 쓰면 성능상 이점을 가져오는데 왜 ?
        - 정적인 리소스를 다룰때만 해당함.
        - 동적인 페이지에는 민감한 정보가 노출될수 있기때문에 브라우저 캐싱될경우 문제가 될 수 있음.
    
- 4. HstsHeaderWriter: HTTPS로만 소통하도록 강제한다.
    - https 인증서 기본 유효기간이 1년이기 때문에 1년을 기본 Default 값으로 설정해 주는 등 헤더 정보를 제공한다.
    - https 설정을 하면 헤더정보가 같이 나간다.

- 5. XFrameOptionsHeaderWriter: clickjacking방어
    - iframe 과 같은것을 활용하여 개인정보 노출을 방지한다. (clickjacking)


#### 참고
- X-Content-Type-Options:
    - https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options
- Cache-Control:
    - https://www.owasp.org/index.php/Testing_for_Browser_cache_weakness_(OTG-AUTHN-006)
- X-XSS-Protection
    - https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection
- Lucy-Filter
    - https://github.com/naver/lucy-xss-filter
- HSTS
    - https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html
- X-Frame-Options
    - https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
https://cyberx.tistory.com/171
