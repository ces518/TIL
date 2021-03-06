# Builder Pattern 

```java
Member customer = Member.build()
    .name("홍길동")
    .age(30)
    .build();
```


생성자 인자가 많을시 Builder패턴을 고려.



```java
// Effective Java의 Builder Pattern
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters(필수 인자)
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values(선택적 인자는 기본값으로 초기화)
        private int calories      = 0;
        private int fat           = 0;
        private int carbohydrate  = 0;
        private int sodium        = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;    // 이렇게 하면 . 으로 체인을 이어갈 수 있다.
        }
        public Builder fat(int val) {
            fat = val;
            return this;
        }
        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        public Builder sodium(int val) {
            sodium = val;
            return this;
        }
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}


NutritionFacts facts = new NewtiritionFacts.builder()
	.calotires(100)
	.sodium(35)
	.build();
```


최초호출시 NutirionFacts에 기본값을 세팅함
return this를통해 체이닝이 가능하다.
마지막에 build를 해줌으로서 
내부의 builder클래스에있는 값들을 실제 세팅할
클래스의 객체의 멤버로 값을세팅해줌.


이런 스타일의 빌더패턴이라면 

롬복의 @Builder 어노테이션으로 쉽게사용가능...




