# 실전! Querydsl - H2 데이터베이스 설치
- 개발이나 테스트 용도로 가볍고 편리한 DB, 웹 화면 제공

# H2 데이터베이스 설치
- h2database.com

- 먼저 파일모드로 실행을 해주어야 TCP로 접속이 가능하다.
    - jdbc:h2:~/querydsl
    - querydsl.mv.db 파일이 생성되었음을 확인후 TCP모드로 접근
    - jdbc:h2:tcp://localhost/~/querydsl

> H2 DB설치시 Spring Boot H2 DB 버전도 확인해야한다. 버전이 다를경우 정상 동작이 안될수 있음.
