# centos apache 설치
- yum 패키지 매니저를 이용해 손쉽게 설치가 가능하다.
 ```
 yum -y install httpd !- 아파치 설치
 systemctl enable httpd.service            !- 부팅시 자동시작
 systemctl start httpd                     !- 아파치 서버 시작
 ```
 
 - 정상적으로 설치가 되었다면, 80 포트로 접근시 default welcompage에 접근이 가능하다.

- https://opentutorials.org/module/1701/10228
