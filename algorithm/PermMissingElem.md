# PermMissingElem

`문제`
- N개의 서로 다른 정수로 구성된 배열 A가 주어진다.
- 배열에 [1] 범위의 정수가 포함되어 있다. (N + 1) 정확히는 하나의 요소가 빠져있다.
  - 1... N + 1 의 범위
- 빠진 요소를 찾아라.
- A[0] = 2
- A[1] = 3
- A[2] = 1
- A[3] = 5

`풀이`
- 누적합 공식 사용 : (시작숫자 + 끝숫자 ) * 끝숫자 / 2


```java
// N개의 서로 다른 정수로 구성된 배열 A가 주어진다.
// 배열에 [1] 범위의 정수가 포함되어 있다. (N + 1) 정확히는 하나의 요소가 빠져있다.
// 빠진 요소를 찾아라.
// A[0] = 2
// A[1] = 3
// A[2] = 1
// A[3] = 5

int[] A = {2, 3, 1, 5};

// stream 사용
int max = Arrays.stream(A).max().getAsInt();
System.out.println("max = " + (max - 1));

// 누적합 공식
// (시작 숫자 + 끝숫자) * 끝숫자 /2
long lastNum = A.length + 1;
long sum = 0;
for (int n: A) {
    sum += n;
}
int result = (int)((1 + lastNum) * lastNum / 2 - sum);
```
