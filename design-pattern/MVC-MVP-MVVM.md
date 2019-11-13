# MVC ( Model + View + Controller )

// web에서 많이쓰임.
1. Controller 로 요청이 들어온다.
2. Controller 는 Model의 데이터 업데이트 및 조회
3. Model은 해당 데이터를 Controller에게 전달
4. Controller는 view에게 전달.


# MVP ( Model + View + Presenter )

// android에서 많이쓰임.
1.View로 입력이 들어온다.
2.View는 Presenter 에 작업 요청을한다.
3.Presenter는 Model에 필요한 데이터를 요청한다.
4.Model은 Presenter 에게 응답을한다.
5.Presenter 는 View에 응답을한다.
6.View는 Presenter에게 받은데이터를 화면에보여준다.

* 단점 
view와 presenter가 1:1로 강한 의존성을 가진다.

# MVVM ( Model + View + ViewModel )
// MVVM은 두가지 디자인패턴을 이용한다.
Command 패턴과 Data Binding패턴을이용한다.

1.View에 입력하면 Command패턴으로 인해 ViewModel에 명령을한다.
2.ViewModel은 필요한 데이터를 Model에 요청한다.
3.Model은 ViewModel에 필요한 데이터를 응답한다. 
4.ViewModel은 응답받은 데이터를 가공해서 저장한다.
5.View는 ViewModel과 Data Binding 으로 인해 자동 갱신.

