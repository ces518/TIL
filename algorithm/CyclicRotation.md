# CyclicRotation

`문제`
- 배열 A와 정수 K가 주어진다.
- 배열 A의 요소를 1번 회전시키면 다음과 같다.
- A [3, 8, 9, 7, 6] -> [6, 3, 8, 9 ,7]
- 배열을 회전시켰을때 각 요소들이 1만큼 우측으로 이동하고, 가장 마지막 요소는 0번에 위치하게 된다.


`풀이`
- K 번 만큼의 이동은 length 범위 내에서의 이동이다.
  - length 의 범위를 벗어날 수 없다.

```java
// 배열 A와 정수 K가 주어진다.
// 배열 A의 요소를 1번 회전시키면 다음과 같다.
// A [3, 8, 9, 7, 6] -> [6, 3, 8, 9, 7]
// 회전을 시키면 가장 마지막요소가 0번으로 위치한다.
// 배열 A를 주어진 정수 K만큼 회전시키는 함수를 완성

// 현재 인덱스의 숫자의 K번 만큼 이동은 length내에서의 이동이다.

int[] a = {3, 8, 9, 7, 6};
int k = 3;
int length = a.length;
int[] result = new int[length];

for (int i = 0; i < length; i++) {
    int index = (k + i) % length;
    result[index] = a[i];
}
System.out.println("Arrays.toString(result) = " + Arrays.toString(result));
```
