# 실전! 스프링 데이터 JPA - 스프링 데이터 JPA 페이징과 정렬

#### 페이징과 정렬 파라미터
- org.springframework.data.domain.Sort: 정렬 기능
- org.springframework.data.domain.Pageagle: 페이징 기능 (Sort 포함)

> 페이징과 정렬을 공통화 시킴

`Pageable`
- Spring data JPA 에서 제공하는 페이징 관련 객체
- Spring data JPA 에서 페이징이 필요할때 해당 메소드 파라메터로 넣어주기만 하면 된다.


`특별한 반환 타입`
- org.springframework.data.domain.Page: 추가 count 쿼리 결과를 포함하는 페이징
- org.springframework.data.domain.domain.Slice: 추가 count 쿼리 없이 다음페이지만 확인 가능 (내부적으로 limit + 1)


```java
// Pageable 은 0 부터 페이지가 시작
// PageRequest는 Pageable의 구현체
PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

Page<Member> page = memberRepository.findByAge(age, pageRequest);
// 반환타입이 Page일 경우 totalCount 쿼리까지 같이 나간다.
// 반환타입 List는 엔티티 목록만 조회한다.
// long totalCount = memberRepository.totalCount(age);
List<Member> members = page.getContent();
long totalCount = page.getTotalElements();

// then
assertThat(members.size()).isEqualTo(3);
assertThat(totalCount).isEqualTo(7);
assertThat(page.getNumber()).isEqualTo(0); // 페이지 번호도 제공한다.
assertThat(page.getTotalPages()).isEqualTo(3); // 총 페이지
assertThat(page.isFirst()).isTrue(); // 첫페이지 유무
assertThat(page.hasNext()).isTrue(); // 다음 페이지 존재 유무

/////////////////////////////////////////

// Slice 를 사용하면 totalCount는 가져오지 않는다.
// Slice는 limit + 1 개를 요청한다.
Slice<Member> slice = memberRepository.findSliceByAge(age, pageRequest);

// > 페이징 방식을 사용하다가 더보기 방식으로 변경하게 된다면 ?
// > 반환타입을 Slice로 수정하기만 하면 된다.
```

`List`
- 단순히 목록만을 제공받고 싶을때 사용한다.

`Page`
- totalCount, totalPage, 첫 페이지 여부, 다음 페이지 존재 여부 등 페이징 관련 처리시 유용한 기능을 제공한다.

`Slice`
- Page와 비슷하지만 차이점은 totalCount 쿼리가 나가지 않는다.
- maxResult + 1 개 만큼 데이터를 요청한다.
    - 더보기 기능을 구현할때 유용함

> 페이징 방식에서 더보기 방식으로 변경하게 된다면 반환타입을 Slice로 수정하기만 하면 된다.

#### 페이징을 기피하는 이유
- totalCount 쿼리때문에 성능이 안나오는 경우가 많다.
    - DB의 모든 데이터를 카운팅 해야한다.

- Spring data JPA 의 페이징 기능을 사용하면 문제점이 하나 존재한다.
    - 엔티티를 조회할때 사용한 join을 totalCount에서도 그대로 사용하기때문에 성능상 문제가 발생한다.
- toalCount 쿼리의 성능에 따라 다음과 같이 별도의 카운팅 쿼리를 지정해주어 최적화를 진행해야 한다.
```java
@Query(value = "select m from Member m left join m.team t", countQuery = "select count(m) from Member m")
```

#### DTO로 반환하기
- Page에서 map 메소드를 제공하기 때문에 DTO로 손쉽게 변환할 수 있다.
- Page 객체는 API에서 그대로 반환해도 좋다.
```java
Page<MemberDto> toDto = page.map(member -> new MemberDto(member.getId(), member.getUsername(), null));
```
