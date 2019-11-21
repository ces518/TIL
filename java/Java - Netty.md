# Java - Netty

#### Netty 란 ?

Netty는 프로토콜 서버및 클라이언트와 같은 네트워크 어플리케이션을 빠르고 쉽게개발하는것을 가능케해주는
NIO 서버 프레임워크다.


핵심컴포넌트 

- EventLoop
  - task에서 pipeline 과 scheduling으로 전달해주는 역할 

- PipeLine
  - 이벤트 루프에서 이벤트를 받아 핸들러에 전달하는 역할 
- inbound event 
  - 이벤트 루프가 발생시킨이벤트 를 작성한 inbound event handler 에게 전달

- outbound event
  - 사용자가 요청한 동작을작성한 outbound event handler에게 전달
  - 최종적으로 이벤트루프에 전달되어 I/O가 수행되도록한다.


- 핵심 플로우
> 이벤트루프 - 파이프라인 - 컨텍스트 - 핸들러

