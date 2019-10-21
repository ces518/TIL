# 더 자바 코드를 조작하는 방법 - Reflection 
- Class API를 사용하여 리플렉션을 사용할 수 있음

#### Reflection을 사용하여 참조할 수 있는 정보
- 클래스의 필드
- 상위클래스
- 인터페이스
- 메소드 목록 등 ..

#### Reflection 사용해보기
- Reflection 을 사용하기 이전 참조할 클래스를 생성해보자

`Book, MyBook, MyInterface`
```java
public class Book {
    public static String b = "BOOK";

    public static final String C = "BOOK";

    private String a = "a";

    public String d = "d";

    protected String e = "e";

    public Book() {
    }

    public Book(String a, String d, String e) {
        this.a = a;
        this.d = d;
        this.e = e;
    }

    private void f() {
        System.out.println("f");
    }

    public void g() {
        System.out.println("g");
    }

    public int h() {
        return 100;
    }
}

public class MyBook extends Book implements MyInterface {
}

public interface MyInterface {
}
```


`Reflection 사용하기`

Class 인스턴스를 참조하는 방법
- class 인스턴스를 참조하는 방법은 3가지이다.
- 1.Book.class 와 같이 클래스 로드시 힙에 저장되는 class 인스턴스를 참조하는 방법
- 2.특정인스턴스의 getClass() 메소드로 참조하는 방법
- 3.FQCN을 이용해 Class.forName() 메소드로 참조하는 방법

```java
// Class 로딩이 끝나면 class타입의 인스턴스를 만들어 힙에 저장됨
Class<Book> bookClass = Book.class;

// 특정 인스턴스가 있다면 getClass() 를 사용해 가져올 수 있다.
Book book = new Book();
Class<? extends Book> aClass = book.getClass();

// FQCN 으로 접근이 가능함
Class<?> aClass1 = Class.forName("me.june.Book");
```

특정 클래스의 필드 참조하기
- getFields() 메소드와 getDeclaredFields() 메소드를 통해 클래스의 필드에 접근이 가능하다.
- 이때 getFields() 메소드는 public 한 필드에만 접근이 가능하다.
```java
// 클래스의 필드들에 접근
// 이 메소드는 public한 것만 접근이 가능하다
Field[] fields = bookClass.getFields();
Arrays.stream(fields).forEach(System.out::println);

// 모든 필드에 접근이 가능하다.
Field[] declaredFields = bookClass.getDeclaredFields();
Arrays.stream(declaredFields).forEach(System.out::println);
```

특정 클래스의 필드값 가져오기
- 이전과 같은 방법으로 필드의 값을 가져올수 있는데 이때 해당 필드의 접근제어자가 private 이라면 예외가 발생한다.
- private한 필드에도 접근이 가능하도록 setAccessible(true); 로 설정을 해주어야 한다.
```java
// 필드의 값을 가져오기
Arrays.stream(declaredFields).forEach(f -> {
    try {
        // private 한 변수의 값을 가져오기 위한 옵션
        f.setAccessible(true);
        System.out.printf("%s %s", f, f.get(book));
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    }
});
```

특정 클래스의 생성자에 접근하는 방법
- getDeclaredConstructors() 메소드로 생성자에 접근이 가능하다.
```java
// 생성자들 접근
Arrays.stream(bookClass.getDeclaredConstructors()).forEach(System.out::println);
```

특정 클래스의 부모클래스에 접근하는 방법
- getSuperclass() 메소드로 부모 클래스에 접근이 가능하다.
- 이때 부모클래스는 하나만 존재하기 때문에 Class 타입으로 하나의 인스턴스만 가져올 수 있다.
```java
// 부모클래스 접근
Class<? super MyBook> superclass = MyBook.class.getSuperclass();
System.out.println(superclass);
```

특정 클래스의 인터페이스에 접근하는 방법
- getInterfaces() 메소드로 인터페이스에 접근이 가능하다.
```java
// 인터페이스 접근
Arrays.stream(MyBook.class.getInterfaces()).forEach(System.out::println);
```

#### 정리
- 앞에서 살펴본것 외에도 다양한 클래스 정보가 참조 가능하다.
- https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html
