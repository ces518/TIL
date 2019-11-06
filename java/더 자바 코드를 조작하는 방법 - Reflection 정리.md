# 더 자바 코드를 조작하는 방법 - Reflection 정리

#### 사용 되는곳
1.스프링
- 의존성
- MVC qbdptj sjadjdhs 객체에 바인딩할때

2.하이버네이트
- @Entity 클래스에 Setter가 없다면 리플렉션을 사용한다.

3.Junit
- Junit자체에서 유틸리티를 만들어 사용한다.
- https://junit.org/junit5/docs/5.0.3/api/org/junit/platform/commons/util/ReflectionUtils.html

#### 주의사항
- 지나친 사용은 성능 이슈를 야기할 수 있음. 반드시 필요한 경우에만 사용할것
- 컴파일시 발생하지 않고 런타임시에만 발생하는 문제를 만들 가능성이 존재한다.
- 접근 지시자를 무시할 수 있다.

#### 참고
- https://docs.oracle.com/javase/tutorial/reflect/index.html
