# Ioc/DI - 제어의 역전, 의존성주입

- 의존성이란 ?
  - 스프링의 Ioc(Inversion of Control) 이라고도 하는 Di (Dependency Injection) 을 알아보기전 자바에서의 의존성이 무엇인지 알아보자.
  
 
- 의사 코드 
  - 운전자가 자동차를 생산한다.
  - 자동차는 내부적으로 타이어를 생성한다.
  
- 자바코드
  - new Car();
  - Car 객체 생성자 내부에서 new Tire();

- 단순한 정의
  - 의존성은 new 이다.
  - new 를 실행하는 Car와 Tire 사이에서 Car가 Tire에 의존한다.
  

### 스프링을 사용하지않은 자바코드

```java
interface Tire {
  String getBrand();
}

class KoreaTire implements Tire {
  public String getBrand() {
    return "한국 타이어";
  }
}

class KumhoTire implements Tire {
  public String getBrand() {
    return "금호 타이어";
  }
}

class Car {
  Tire tire;
  
  public Car() {
    tire = new KoreaTire();
  }
  
  public String gitTireBrand() {
    return tire.getBrand():
  }
}
```

- Car의 생성자 부분 , 즉 자동차가 타이어를 생성하는 부분에서 의존관계가 일어나고 있다.

> - 자동차는 타이어에 의존한다.
> - 운전자는 자동차를 사용한다.
> - 운전자가 자동차에 의존한다.


### 스프링 없이 의존성 주입하기

- 생성자를 통한 의존성 주입

- 의사코드
  - 운전자가 타이어를 생성한다.
  - 운전자가 자동차를 생산하면서 타이어를 장착한다.
  
- 자바코드 - 생성자 활용
  - Tire tire = new KoreaTire();
  - Car car = new Car(tire);
  
* 의존성 주입이란 ?
  - 주입이란 외부에서 라는 뜻을 내포하고있는 단어이다.
  - 자동차 내부에서 타이어를 생성하는것이 아니라 , 외부에서 생상된 타이어를 자동차에 장착하는 작업이 주입이다.

```java
interface Tire {
  String getBrand();
}

class KoreaTire implements Tire {
  public String getBrand() {
    return "한국 타이어";
  }
}

class KumhoTire implements Tire {
  public String getBrand() {
    return "금호 타이어";
  }
}

class Car {
  Tire tire;
  
  public Car(tire) {
    this.tire = tire;
  }
  
  public String gitTireBrand() {
    return tire.getBrand():
  }
}

class Main {
  public static void main(String[] args) {
    Tire tire = new KoreaTire();
    Car car = new Car(tire);
  }
}
```

- 변경된 코드를 보면 외부에서 생성한 타이어객체를 자동차 객체를 생성할때 생성자의 인자로 사용하는것을 볼 수있다.
- 자동차와 타이어간의 의존성이 외부 (클라이언트) 에 의해 결정되는것으로 변경된것이다.

* 의존성역전 (DI)의 장점 ? 
  - 기존방식대로라면 Car는 KoreaTire, KumhoTire 에 대해 정확한 정보를 알고있어야만 객체를 생성할 수 있었다.
  - 하지만 의존성 주입을 적용할경우 , Car는 단지 Tire인터페이스를 구현한 어떤 객체가 들어오기만한다면 정상작동이 가능하다.
  - 또한 추가적인 확장에도 용이하다.
  - 새로운 타이어 브랜드가 생겨난다고 해도 , Tire인터페이스를 구현해주기만 하면된다.
  - Car객체내부에서 Tire객체를 사용하는 코드에는 변화가 필요없다.
