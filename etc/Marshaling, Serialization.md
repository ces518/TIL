# Marshalling, Serialization


#### Marshalling 이란 ?

- 객체의 메모리 구조를 저장이나 전송을 위해서 적당한 자료형태로 변형하는 것을 의미한다.
- Marshalling 은 보통 서로 다른 컴퓨터 혹은 서로 다른 프로그램 간에 데이터가 이동되어야 할 경우 사용된다.
- 시리얼라이즈와 비슷한 경우는 객체가 원격의 다른 객체와 통신할 때 serialize 된 객체를 사용할 경우이다.
- Marshalling 을 수행함으로써 복잡한 통신, 사용자 정의/복잡한 구조의 객체들을 사용하는대신, 단순한 primitive 들을 사용할 수 있다. Marshalling 의 반대말은 Unmarshalling 이라고 한다.



### Serialization 이란 ?
- 객체의 상태를 저장하기 위해서 객체를 byte stream 으로 변환하는 것을 의미한다. 그 반대는 deserialization 이라고 한다.



### Marshalling vs Serialization 
- Marshalling 과 Serialization 은 원격 프로시저를 호출하는 것에서는 약간 유사하지만, 의도를 따지면 의미적으로 틀리다. Marshalling 을 하게 되면, 원격 프로시저를 호출하는 것에서 함수의 parameter 값들 return 값들을 전달할 수 있다.
- 보통 Marshalling 은 여기저기에서 parameter 들을 얻는 반면, Serialization은 구조화된 데이터 를 byte stream 과 같은 primitive 형식 혹은 그 반대로 복사를 하는 것을 의미힌다. 이러한 의미에서 Serialization은 marshalling 의 pass-by-value 를 구현하는 것의 일종으로 볼 수 있다.
- Marshalling 은 추가적인 메타 데이터 (코드 베이스) 를 가질 수 있다는 것에서 Serialization 과 구별된다.



출처: http://starblood.tistory.com/entry/Marshalling-vs-Serialization-마샬링-과-시리얼라이즈-의-차이 [Drink and Be happy]
