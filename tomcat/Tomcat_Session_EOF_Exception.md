# Tomcat 실행시 세션관련 EOF 발생시 에러

1. context.xml 파일 열어 세션유지설정을 꺼준다

```xml
<Manager className="org.apache.catalina.session.PersistentManager" saveOnRestart="false"/>
```

2. work 디렉토리 하위에 있는 session.ser 을 삭제한다.


* 톰캣은 persist session 을 저장 할 수 있다
persist session 은 톰캣을 shutdown restart 시 생성되며 start는 삭제된다고한다.
이러한 작업을 실패했을떄 java.io.EOFException 이 발생하는것이다.
