# 더 자바 코드를 조작하는 방법 - Reflection API 클래스 정보 수정 또는 실행

#### Reflection API 를 활용해 인스턴스 생성하기

`기본생성자를 가져와 인스턴스 생성`
- 기본생성자로 인스턴스를 생성하는 방법은 getConstructor 메소드에서 인자를 null로 지정해주어 생성자를 가져온다.
- 가져온 생성자의 newInstance 메소드를 통해 인스턴스를 생성해준다.
```java
// 기본생성자를 가져와 인스턴스 생성
Constructor<Book> constructor = bookClass.getConstructor(null);
Book book1 = constructor.newInstance();
```

`파라메터가 존재하는 생성자를 가져와 인스턴스 생성`
- 파라메터가 존재하는 생성자로 인스턴스를 생성하는 방법은 getConstructor 메소드에서 인자를 Class type 인스턴스를 지정해주어 생성자를 가져온다.
- 기본 생성자와 마찬가지로 newInstance 메소드를 통해 인스턴스르 생성해준다.
```java
// 파라메터가 존재하는 생성자를 가져와 인스턴스를 생성
Constructor<Book> constructor1 = bookClass.getConstructor(String.class);
Book hello = constructor1.newInstance("Hello");
```

#### Reflection API 를 활용해 필드 정보 참조하기

`static 필드 정보 참조하기`
- getDeclaredField메소드를 사용해 필드 메타정보를 가져온다.
- 가져온 field의 get, set 메소드를 활용해 필드의 값을 가져오거나, 수정할 수 있다.
- 이때 해당 필드가 static 필드가 아니라면, 첫번째 인자로 특정 인스턴스를 넘겨주어야한다.
```java
Field a = Book.class.getDeclaredField("A");
// 값을 가져올때 해당 필드가 특정한 인스턴스에 해당하는 필드라면, 인스턴스를 넘겨줄 수 있다.
// 하지만 지금은 static한 필드이기 때문에 null을 넘겨주면 가져올 수 있다.
a.get(null);
// 마찬가지로 첫번째인자는 특정 인스턴스, 두번째는 값을 지정해 변경할 수 있다.
a.set(null, "BBBBB");
```

`instance 필드 정보 참조하기`
- static 필드와 마찬가지로 getDeclaredField메소드를 사용해 필드 메타정보를 가져온다.
- 가져온 field의 get, set 메소드를 활용해 필드의 값을 가져오거나, 수정할수 있으며 첫번째 인자로 인스턴스를 넘겨주어야 한다.
```java
Field b = Book.class.getDeclaredField("B");
// B는 특정 인스턴스에 해당하는 필드이기 때문에 book 인스턴스를 넘겨주어야 한다.
// private 변수이기 때문에 접근지시자를 무시하고 가져오도록 한다.
b.setAccessible(true);
b.get(book1);
b.set(book1, "CXCCCCCC");
```

#### Reflection API 를 활용해 메소드 정보 참조하기

`매개변수가 없는 메소드 정보 참조하기`
- getDeclaredMethod 메소드를 활용해 메소드 정보를 참조할 수 있다.
- invoke 메소드를 사용해여 메소드를 실행할 수 있다.
- 특정 인스턴스에 해당하는 메소드라면 해당 인스턴스를 넘겨 주어야 한다.
```java
Method c = Book.class.getDeclaredMethod("c");
// 특정 인스턴스에 해당하는 메소드라면 인스턴스를 넘겨주어야한다.
// private 메소드이기 떄문에 접근지시자를 무시하고 실행하도록 한다.
c.setAccessible(true);
c.invoke(book1);
```

`매개변수가 있는 메소드 정보 참조하기`
- 이전과 마찬가지로 getDeclaredMethod 메소드를 활용해 메소드 정보를 참조할 수 있다.
    - 메소드 정보를 참조할때 해당 메소드의 파라미터의 type은 기본형 타입과 Wrapper 타입을 구분하기 떄문에 주의해야한다.
- invoke 메소드를 사용하여 메소드를 실행할 수 있다
```java
// 메소드를 가져올때 Primitive type, Wrapper type을 구분하므로 주의할것
Method d = Book.class.getDeclaredMethod("d", int.class, int.class);
int invoke = (int) d.invoke(book1, 1, 2);
```
