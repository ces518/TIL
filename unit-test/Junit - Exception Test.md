# Junit - Exception Test
- 예외에 대한 테스트하는 방법

#### expected
- 가장 쉬우면서도 단순한 방법
    - Junit @Test 애노테이션의 expected 를 활용하는 방법이다.
    - 예외에 대한 타입밖에 확인하지 못한다는 단점이 존재한다.
```java
@Test(expected = UsernameNotFoundException.class)
public void findByUsernameFail () {
    final String username = "random@gmail.com";
    accountService.loadUserByUsername(username);
}
```

#### Try - Catch 
- 가장 익숙(?) 한 방법중 하나이다.
    - try-catch 블록을 활용하여 catch 블록으로 가지않는다면 명시적으로 fail() method를 호출하여 테스트를 실패시킨다.
    - catch 블록에서 예외에 대한 정보를 가져올 수 있기때문에 보다 정확한 테스트가 가능하다.
    - 하지만 코드가 장황해진다는 단점이 있다.
```java
@Test
public void findByUsernameFail () {
    final String username = "random@gmail.com";
    try {
        accountService.loadUserByUsername(username);
        fail("supposed to be failed");
    } catch (UsernameNotFoundException e) {
        assertThat(e instanceof UsernameNotFoundException).isTrue();
        assertThat(e.getMessage()).contains(username);
    }
}
```

#### @Rule, ExpectedException
- Junit에서 제공하는 @Rule과 ExpectedException을 활용하는 방법
    - 코드가 간결해지고, 예외에 대한 정보를 가져올 수 있다. 
```java
@Rule
public ExpectedException expectedException;

@Test
public void findByUsernameFail () {

    final String username = "random@gmail.com";

    expectedException.expect(UsernameNotFoundException.class);
    expectedException.expectMessage(Matchers.containsString(username));

    accountService.loadUserByUsername(username);
}
```
