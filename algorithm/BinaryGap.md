# BinaryGap

`문제`
- N의 2진수에서 1로 둘러쌓인 연속적인 0의 최대 숫자를 구하라
  - 10000100101 에서 1과 1사이의 0개의 갯수의 최대수


`풀이`
- 1을 만나면 시퀀스를 초기화, 0을 만나면 시퀀스를 증가시킨다.
  - 시퀀스를 초기화 할때, 현재 최대 시퀀스값과 비교하여 최대 값을 유지한다.

```java
// N의 2진수에서 1로 둘러쌓인 연속적인 0의 최대 수
// 1을 만나기 전까지 시퀀스를 증가시킴
// 1을 만나면 시퀀스를 초기화 시킨다.
int N = 1041;

String binaryString = Integer.toBinaryString(N);
int length = binaryString.length();
int maxSequence = 0;
int sequence = 0;

for (int i = 0; i < length; i++) {
    if ('1' == binaryString.charAt(i)) {
        maxSequence = Math.max(maxSequence, sequence);
        sequence = 0;
    } else {
        sequence ++;
    }
}
System.out.println("maxSequence = " + maxSequence);
```
 
