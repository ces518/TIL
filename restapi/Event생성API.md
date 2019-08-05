# REST API - Event 생성 API
- Event 생성 API
    - 입력 받아야 할 값
        - name
        - description
        - beginEnrollmentDateTime
        - endEnrollmentDateTime
        - closeEnrollmentDateTime
        - beginEventDateTime
        - endEventDateTime
        - location: 없다면 온라인 모임
        - basePrice
        - maxPrice
        - limitOfEnroll

- basePrice, maxPrice 수치에 따른 각각의 로직
- 0, 100 : 선착순
- 0, 0: 무료
- 100, 0 : 경매
- 100, 200 : 제한가 선착순 등록


- 결과 값
    - id
    - name
    - 그 외값
    - eventStatus.DRAFT : 기본상태
    - offline
    - free
    - _links
        - profile
        - self
        - publish
        - ...
