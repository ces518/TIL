# 더 자바, 애플리케이션을 테스트하는 방법

## Junit 설정
- src/test/resources 디렉터리에 설정파일을 위치
- junit-platform.properties

```properties
## 테스트 인스턴스 라이프 사이클 설정
junit.jupiter.testinstance.lifecycle.default = per_class

## 확장팩 자동 감지 기능, 기본으로 활성화 되어 있지 않다
junit.jupiter.extensions.autodetection.enabled = true

## @Disabled 무시하고 실행
junit.jupiter.conditions.deactivate = org.junit.*DisabledCondition

## 테스트 이름 표기 전략 설정
junit.jupiter.displayname.generator.default = \
    org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores
```

- properties 파일로 적용한 설정은 Global 로 적용된다.
- @DisplayName 과 같이 애노테이션 우선순위가 더 높은 경우 해당 값이 적용되므로 주의