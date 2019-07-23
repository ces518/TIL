# Spring Http HEAD, OPTIONS 요청 처리
- Spring MVC에서 자동으로 처리하는 Http Method
    - HEAD
    - OPTIONS

- HEAD
    - GET 요청과 동일하지만, 응답본문을 받아오지않고, 응답 헤더만 가져온다.

- OPTIONS
    - 사용할수 있는 HTTP Method제공 
    - 서버 또는 특정 리소스가 제공하는 기능을 확인하는 용도
    - 서버는 Allow 응답 헤더에 사용할 수 있는 HTTP Method 목록을 응답 해야한다.

- Spring MVC 를 사용한다면 기본적으로 제공하는 기능이기 때문에 따로 구현을 할 필요가 없다.
