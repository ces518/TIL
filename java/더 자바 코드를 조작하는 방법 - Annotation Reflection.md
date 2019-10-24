# 더 자바 코드를 조작하는 방법 - Annotation Reflection

#### Annotation
애노테이션은 JEE5 부터 추가된 기능이다.

애노테이션은 다양한 목적으로 사용되지면 주로 **메타 데이터**의 비중이 가장 크다.

>메타 데이터(Meta-Data): 데이터를 위한 데이터, 한 데이터에 대한 설명을 의미한다

`MyAnnotation`
```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.FIELD})
@Inherited
public @interface MyAnnotation {

    String name() default "juneyoung";

    int number() default 100;

    String value() default "hello";

}
```

@Retention
- 컴파일러가 애노테이션을 다루는 방법을 기술하며, 어느 시점까지 정보를 유지할것인지 결정한다.
- RetentionPolicy.SOURCE : 컴파일 전까지만 유효. (컴파일 이후에는 사라짐)
- RetentionPolicy.CLASS : 컴파일러가 클래스를 참조할 때까지 유효.
- RetentionPolicy.RUNTIME : 컴파일 이후에도 JVM에 의해 계속 참조가 가능. (리플렉션 사용)
- 기본값은 CLASS

@Target
- 애노테이션을 적용할 위치를 결정한다.
- ElementType.PACKAGE : 패키지 선언
- ElementType.TYPE : 타입 선언
- ElementType.ANNOTATION_TYPE : 어노테이션 타입 선언
- ElementType.CONSTRUCTOR : 생성자 선언
- ElementType.FIELD : 멤버 변수 선언
- ElementType.LOCAL_VARIABLE : 지역 변수 선언
- ElementType.METHOD : 메서드 선언
- ElementType.PARAMETER : 전달인자 선언
- ElementType.TYPE_PARAMETER : 전달인자 타입 선언
- ElementType.TYPE_USE : 타입 선언

@Inherited
- 애노테이션을 상속이 가능하게 한다.

- 애노테이션은 값들을 가질수 있는데 가질수 있는 데이터의 타입은 제한되어있다.
- 기본형 타입과, enum,  Wrapper 타입들만 속성으로 가질수 있다.
- 해당 속성의 기본값도 지정이 가능하다.
```java
String name() default "juneyoung";
```

속성을 지정하면 애노테이션을 사용할때 다음과 같이 속성에 값을 지정할수 있다.
```java
@MyAnnotation(name = "Hello")
```

만약 **값을 받을 속성의 이름을 value 로 지정하면 속성명을 생략**할 수 있다.
```java
@MyAnnotation("Hello")
```

#### Reflection으로 Annotation 정보 참조하기

애노테이션은 기본적으로 주석과 동일한 취급을하기 때문에 바이트코드 로드시 해당 애노테이션의 정보는 제외하고 로드된다.
- Reflection으로 해당 정보를 참조 할 수 없다.

Reflection으로 해당 정보를 참조하고 싶다면 (애노테이션의 정보를 RUNTIME시 까지 유지) @Retention을 사용하여 해당 애노테이션 정보를 언제까지 유지할것인지 지정해 주어야한다.


getAnnoitations()
- 상위 클래스의 상속가능한 애노테이션 정보 까지 참조한다.

getDeclaredAnnotations()
- 타겟 클래스의 애노테이션 정보만 참조한다.
```java
// getAnnotations() : 상위 클래스의 상속가능한 애노테이션정보까지 가져온다.
// getDeclaredAnnotations() : 현재 클래스의 애노테이션 정보만 가져온다.
Arrays.stream(MyBook.class.getAnnotations()).forEach(System.out::println);
Arrays.stream(MyBook.class.getDeclaredAnnotations()).forEach(System.out::println);
```
