# 더 자바, 애플리케이션을 테스트하는 방법

## Junit 확장 모델
- Junit4 에 비해 단순해 졌다.
    - @RunWith(Runner) 를 사용하거나, TestRule, MethodRule 등으로 나누어져 있었다.
- Junit5 에서는 Extension 하나로 통합되었다.

`FindSlowTestExtension`

```java
public class FindSlowTestExtension implements BeforeTestExecutionCallback, AfterTestExecutionCallback {

	// 1초로 제한
	private long THRESHOLD = 1000L;

	public FindSlowTestExtension(long threshold) {
		this.THRESHOLD = threshold;
	}

	@Override
	public void beforeTestExecution(ExtensionContext context) throws Exception {
		// ExtensionContext 내에서는 값을 저장할 수 있는 Store 라는 개념이 존재함.
		String testClassName = context.getRequiredTestClass().getName();
		String textMethodName = context.getRequiredTestMethod().getName();
		ExtensionContext.Store store = context.getStore(ExtensionContext.Namespace.create(testClassName, textMethodName));
		store.put("START_TIME", System.currentTimeMillis());
	}

	@Override
	public void afterTestExecution(ExtensionContext context) throws Exception {
		String testClassName = context.getRequiredTestClass().getName();
		String textMethodName = context.getRequiredTestMethod().getName();
		ExtensionContext.Store store = context.getStore(ExtensionContext.Namespace.create(testClassName, textMethodName));
		long startTime = store.remove("START_TIME", long.class);
		long duration = System.currentTimeMillis() - startTime;
		if (duration > THRESHOLD) {
			System.out.printf("Please consider mark method [%s] with @SlowTest. \n", textMethodName);
		}
	}
}
```
- @SlowTest 애노테이션이 적용되어 있지 않지만, 테스트 시간이 THRESHOLD 보다 느릴경우 경고 메시지를 띄워주는 Extension
- 테스트 실행시간을 측정하기 위해 BeforeTestExecutionCallback, AfterTestExecutionCallback 모두 구현해 주어야 한다.
- ExtensionContext 를 통해 테스트 클래스와 테스트 메소드에 대한 정보를 알 수 있다.
- ExtensionContext 에서는 값을 저장할 수 있는 영역인 Store 라는 개념이 존재한다.
- 테스트 시작 시간을 Store 에 저장해두고, 테스트 종료시 Store 에 하는 시간과 현재 시간을 이용해 테스트 수행시 걸린 시간을 계산하도록 구현

`StudyTest`

```java
@ExtendWith(FindSlowTestExtension.class) // 선언적인 Extension 사용 방법, 이 방법은 Extension 을 커스터마이징 할 수 없다..
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class StudyTest {

	// 프로그래밍으로 등록하는 방법
	@RegisterExtension
	static FindSlowTestExtension findSlowTestExtension = new FindSlowTestExtension(1000L);
}
```
- 생성한 Extension 은 크게 3가지 방법으로 사용할 수 있다.
1. @ExtendWith 를 이용한 선언적인 방법
    - @ExtendWith 를 사용한 방법의 단점은 Extension 인스턴스 생성시 특정 값을 커스텀 하는등 이 제한된다.
2. @RegisterExtension 를 이용한 프로그래밍적으로 사용하는 방법
    - @RegisterExtension 는 코드레벨에서 커스터마이징이 가능하다.
3. ServiceLoader 를 사용한 방법
    - 글로벌 설정이 가능하지만, 모든 테스트에 적용되기 때문에 원치않은 Extension 이 적용될 수 있다.

    
- https://junit.org/junit5/docs/current/user-guide/#extensions