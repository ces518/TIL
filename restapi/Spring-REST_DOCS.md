# REST API - Spring - REST DOCS 소개
- SpringMVC TEST를 사용해서 RESTAPI 문서의 일부를 생성할때 유용한 라이브러리
- JAVA 8 이상
- Spring 5 까지 지원

#### Snippets
- 생성한 TEST를 실행할때 사용한 헤더, 파라메터 값 등을 문서 조각이라고 부른다.
- 이러한 문서 조각들을 Snippets 라고 한다.
- 문서 조각들을 활용하여 REST API Document 를 HTML로 완성 가능하다.

#### Asciidoctor
- Spring - REST DOCS 는 Asciidoctor 를 사용한다.
- plain/text로 작성한 문서를 Asciidoctor 문법에 맞게 작성한 Snippets 을 조합해서 HTML문서로 만들어준다.

#### Spring Boot 자동설정
- 테스트 클래스 상단에 @AutoConfigureRestDocs 애노테이션을 사용하면 바로 사용이 가능해짐

#### Swagger
- Spring - REST DOCS 외에도 여러 Document 라이브러리가 존재한다.
- 애노테이션 기반으로 동작하고 널리 사용하는 라이브러리
