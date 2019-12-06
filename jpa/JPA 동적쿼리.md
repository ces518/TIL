# JPA 동적쿼리

Criteria + Mybatis 방식
```java
private Predicate createDynamicQuery (CriteriaBuilder cb, Root<Relic> root, String searchType, String searchTxt) {
    List<Predicate> dynamicQuery = new ArrayList<>();
    dynamicQuery.add(cb.equal(root.get("delSts"), "N"));
    if (searchType == null) {
        return cb.and(dynamicQuery.toArray(new Predicate[dynamicQuery.size()]));
    }
    switch (searchType) {
        case "1":
            // 유물명 검색
            dynamicQuery.add(cb.like(root.get("title"), "%" + searchTxt + "%"));
            break;
        case "2":
            // 유물 내용 검색
            dynamicQuery.add(cb.like(root.get("content"), "%" + searchTxt + "%"));
            break;
    }
    return cb.and(dynamicQuery.toArray(new Predicate[dynamicQuery.size()]));
    }
```

Querydsl + BooleanExpression 방식
```java
/**
    * 제목 검색
    * @param searchType
    * @param searchTxt
    * @return
    */
private BooleanExpression likeTitle (String searchType, String searchTxt) {
    if (!"1".equals(searchType)) {
        return null;
    }
    return QRelic.relic.title.like("%" + searchTxt + "%");
}

/**
    * 내용 검색
    * @param searchType
    * @param searchTxt
    * @return
    */
private BooleanExpression likeContent (String searchType, String searchTxt) {
    if (!"2".equals(searchType)) {
        return null;
    }
    return QRelic.relic.content.like("%" + searchTxt + "%");
}
```
