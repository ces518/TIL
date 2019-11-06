# 더 자바 코드를 조작하는 방법 - DI Container 만들어보기
- Sprign IoC Container에 비하면 부족하지만 Reflection을 활용해 DI Container를 만들어 보자

#### @Inject

@Inject 애노테이션을 만들어 해당 애노테이션이 존재하는 필드까지 DI가 되도록 구현한다.
- Reflection 을 활용하려면 런타임시까지 애노테이션정보가 존재해야하기 떄문에 Rentention을 RUNTIME으로 지정해준다.
```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Inject {
    
}
```

#### ContainerService

우리가 구현할 DI Container 클래스이다.
- T getObject(Class<T>)
    - 메소드 파라메터로 클래스 인스턴스를 받아 Reflection을 활용해 DI 해준다.
    - 이때 createInstance를 활용하며, 만약 @Inject 애노테이션이 붙은 필드가 있다면 해당 필드의 인스턴스도 DI 해준다.
- T createInstance(Class<T>)
    - 메소드 파라메터로 클래스 인스턴스를 받아 Reflection을 활용해 기본 생성자를 가져온뒤 인스턴스를 반환해준다.
```java
public class ContainerService {

    // Method paramter로 넘기는 타입으로 리턴타입을 받는다.
    public static <T> T getObject(Class<T> classType){
        T instance = createInstance(classType);
        Arrays.stream(classType.getDeclaredFields()).forEach(f -> {
            if (f.getAnnotation(Inject.class) != null) {
                Class<?> type = f.getType();
                Object instance1 = createInstance(type);
                f.setAccessible(true);
                try {
                    f.set(instance, instance1);
                } catch (IllegalAccessException e) {
                    throw new RuntimeException(e);
                }
            }
        });
        return instance;
    }

    // instance를 생성하는 메소드
    private static <T> T createInstance(Class<T> classType) {
        try {
            return classType.getConstructor(null).newInstance();
        } catch (InstantiationException | IllegalAccessException | InvocationTargetException | NoSuchMethodException e) {
            throw new RuntimeException(e);
        }
    }
}
```

#### DI 사용해보기

우리가 구현한 DI Container를 사용해보자.

사용하기에 앞서 간단한 클래스를 생성한다.

BookService클래스가 BookRepository에 의존하고 있는 구조이다.

만약 BookService를 DI Container를 통해 객체를 생성한다면, BookService내의 BookRepository도 인스턴스가 존재해야한다.
```java
public class BookRepository {
}

public class BookService {
    @Inject
    BookRepository bookRepository;
}
```

`테스트 코드`
```java
public class ContainerServiceTest {

    @Test
    public void bookRepository () {
        BookRepository object = ContainerService.getObject(BookRepository.class);
        assertNotNull(object);
    }

    @Test
    public void bookService () {
        BookService object = ContainerService.getObject(BookService.class);
        assertNotNull(object);
        assertNotNull(object.bookRepository);
    }
}
```

#### 정리
- JAVA Reflection을 활용해 Spring 만큼은 아니지만 DI Container를 직접 구현해 보았다.
- 클래스 메타정보를 참조해 할수 있는 일은 무궁무진하다.
- 여러 프레임워크들이나 각 디자인패턴중 에서도 Reflection을 활용하여 견고하고 깔끔한 것들이 많다.
- 하지만 언제나 과한것은 좋지않다. Reflection 기술자체가 성능상 많이 느리기때문에 적절히 사용하는것을 추천한다.
