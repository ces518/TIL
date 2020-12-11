# Spring Cloud - Feign Client 사용시 주의점

### FeignClient 란 ?
- Netfix 에서 만든 프로젝트로 spring-cloud 프로젝트중 일부
- Http 통신시 HttpUrlConnection, HttpClient, OkHttp 등 클라이언트를 이용해 직접구현해야 한다.
- Feign Client 는 인터페이스를 선언하면 해당 세부구현은 FeignClient 가 처리해주어 편리한 구현이 가능해짐.
- https://spring.io/projects/spring-cloud-openfeign

### 주의점
- Feign Client 는 별다른 설정이 없다면, 기본 전략으로 Http 통신시 Default Client 를 사용하는데, 이는 JDK native URLConnection 을 이용한다.
- 때문에 기본설정으로는 별도의 ConnectionPool 을 사용할 수 없다.
- 만약 ConnectionPool 설정이 필요하다면 Feign Client에서 지원하는 Apache HttpClient 혹은 OkHttp 를 사용해야한다.
- HttpClient 와 OkHttp 도 서로 장단점이 존재한다.

[OkHttp의 장단점](https://github.com/square/okhttp/issues/3472)
```
  OkHttp has HTTP/2, a built-in response cache, web sockets, and a simpler API. 
  It’s got better defaults and is easier to use efficiently. 
  It’s got a better URL model, a better cookie model, a better headers model and a better call model. OkHttp makes canceling calls easy. 
  OkHttp has carefully managed TLS defaults that are secure and widely compatible. 
  Okhttp works with Retrofit, which is a brilliant API for REST. 
  It also works with Okio, which is a great library for data streams. 
  OkHttp is a small library with one small dependency (Okio) and is less code to learn. 
  OkHttp is more widely deployed, with a billion Android 4.4+ devices using it internally.
```

- 만약 DefaultClient 가 아닌 구현체 (HttpClient, OkHttp) 를 사용한다면 주의할 점이 하나 더 있음.
- HttpClient 기준으로 설명을 하면, CloseableHttpClient를 구현체로 등록해서 사용하게 된다.
- 하지만 FeignClient 가 이를 직접사용하는 것이 아니라 한번 Wrapping 해서 사용 (퍼사드) 한다.
- 주의할점은 Client 구현체를 등록할때 설정한 ConnectionTimeout, ReadTimeout의 옵션은 **무시** 된다.
- HttpClient를 기준으로 ApacheHttpClient 클래스로 Wrapping 해서 사용하는데
- 매 요청시마다 feign config 에 설정된 connectionTimeout, readTimeout 옵션을 사용하므로 주의할것.