# SpringBoot ResourceLoader 
- Executable jar 로 패키징후 실행시 classpath에 존재하는 파일을 resourceLoader로 읽을시 예외 발생
>  class path resource [mailTemplates/complete.html] cannot be resolved to absolute file path because it does not reside in the file system

- 원인
> - getFile() 의 경우 문제가 발생함.
> - war 파일이나 IDE로 application run as로 실행하였다면 실제 resource 파일인 file:// 프로토콜을 쓰기 때문에 File객체를 생성해 줄수 있지만, executable jar로 실행 했다면 FileNotFoundException이 발생 하게 됩니다.

- 해결방법
https://sonegy.wordpress.com/2015/07/23/spring-boot-executable-jar%EC%97%90%EC%84%9C-file-resource-%EC%B2%98%EB%A6%AC/
```java
ClassPathResource classPathResource = new ClassPathResource(filePath);
```
