# MySQL Dump Import

#### Dump
- 전체
  - mysqldump -u 사용자명 -p 데이터베이스명 > 저장파일명.sql
 
- 특정 테이블만
  - mysqldump -u 사용자명 -p 데이터베이스명 테이블명 > 저장파일명.sql


- 복구
  - mysql -u 사용자명 -p 데이터베이스명 < 파일명.sql
  
  

- https://kssong.tistory.com/2
- https://code-factory.tistory.com/21
