# Docker 
- docker 는 가상화 기술을 가상화가아닌 독립적인 프로세스로 이용가능
  성능적인 측면에서 유용함
  
 > docker ps : docker 컨테이너 상태
 
 > docker imgaes : docker 이미지상태
 
 > docker rm : docker 컨테이너 삭제
 
 > docker rmi : docker 이미지 삭제
 
 > docker run : docker 이미지 실행과 동시에 진입
 
 > -i -t
 
 > -d
 
 > docker start: docker 이미지 실행만함
 
 > docker build -t demo .

## Mysql

1. mysql 도커이미지 조회
- docker search mysql
> 해당 명령어를 입력하면 mysql관련 도커이미지를 도커허브를 통해 검색

2. mysql 도커이미지 다운로드
- docker pull mysql
> 해당 명령어를 입력하면 해당 도커이미지를 로컬에 다운로드한다 
> 태그명을 생략했으므로 , mysql최신버전 이미지가 다운로드된다.

3. 다운로드된 도커이미지 확인
- docker images
> 위 명령어로 다운받은 이미지를 확인한다. TAG항목에 latest가 최신버전을 알림.

4. 도커이미지를 통해 mysql 컨테이너 생성
- docker run -d -p 3306:3306 -e MYSQL_RROT_PASSWORD=password --name mysql mysql
> 해당 명령어를 입력하면 mysql 이미지를 통해 mysql컨테이너를 생성하고 동작한다.

> -d : demon 으로 실행

> -p : post binding (로컬의 포트와 도커 컨테이너의 포트는 다르므로 포트바인딩을 해주어야만 접근이가능하다.)

> -e : 환경변수를 지정한다.

> -name : 컨테이너의 이름을 지정한다.

 5. 터미널에서 mysql컨테이너 접속
 - docker exec -i -t mysql bash
 > 해당 명령어를 입력하면 mysql 컨테이너에 접근이가능하다 

## Dockerfile

1. spring boot application build
- mvn clean package 
> 해당 명령어를 실행하면  spring boot application 이 *.jar 파일로 패키징된다.

2. Dockerfile 생성
- Dockerfile 이라는 파일명으로 Dockerfile을 생성한다.
```
FROM openjdk:11-jdk
ADD target/demo-rest-api-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-jar","app.jar"]
```
> openjdk-11 , maven으로 빌드한 jar를 app.jar로 컨테이너에 추가

> 컨테이너 실행시 java -jar app.jar 명령어를 실행 옵션 

3. Dockerimage build
- docker build -t demo . 
> dockerfile을 dockerimage로 빌드 

4. image 실행
- docker run -d -p 9000:8080 --name demo-appilcation demo 
> 생성한 docker image file을 9000 port로 실행