# 더 자바, 애플리케이션을 테스트하는 방법

## Junit 이란 ?
- 자바 개발자가 가장 많이 사용하는 테스트 프레임워크
    - 단위 테스트를 사용하는 자바 개발자 93% 가 Junit 을 사용한다.
- 스프링부트가 2.2.x 로 올라가면서 Junit5 를 의존성으로 제공한다.
- Junit4 와 Junit5는 조금 다르다.
    - Junit4 는 하나의 디펜던시로 들어오고, 각종 라이브러리를 같이 사용하는 구조
    - Junit5 는 하나 자체로 모듈화가 되어있다.

## Junit 의 구조
- Junit Platform
    - 테스트를 실행시켜 주는 런쳐 제공, TestEngine API 를 제공한다.
- Jupiter
    - Junit 5 구현체
- Vintage
    - Junit 4, 3 구현체