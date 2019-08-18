# Call - by Value , Call - by - reference


#### Call By Value
- 함수로 인자를 전달할 때 전달될 결과와 대응하는 함수의 변수로 '복사' 되며 복사된 값은
- 지역적으로 사용되는 로컬변수이다.
- 호출된 함수에서 어떤작업을 하더라도 caller는 영향을 받지않음.


# Call By Reference
- 함수로 인자를 전달할때 변수의 묵시적인 레퍼런스를 전달, 그것이 변수의값은 아님.
- 변수자체에 대한 레퍼런스를 전달한다. C언어는 포인터로 구현할 수 있다.



```
C       :  call by value   ( 포인터 그 자체도 매개변수로 넘어갈때 value  복사됨, clone copy 됨 ) 

C++   : call by value    (  "&  레퍼런스"  일 경우 call by reference , 즉 clone copy 하지 않음 )

Java  : call by value   ( 레퍼런스 그 자체도 매개변수로 넘어갈때 value  복사됨, , clone copy 됨 ) 

C#     : call by value    ( "ref"  키워드 붙으면  call by reference , 즉 clone copy 하지 않음 )
```
