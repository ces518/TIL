# 실전! 스프링 데이터 JPA - 스프링 데이터 JPA 구현체 분석

#### 스프링 데이터 JPA 구현체 분석
- SimpleJpaRepository
    - JpaRepository 의 구현체
```java
/*
 * Copyright 2008-2020 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package org.springframework.data.jpa.repository.support;

import static org.springframework.data.jpa.repository.query.QueryUtils.*;

import java.util.ArrayList;
import java.util.Collection;
import java.util.Collections;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;
import java.util.Optional;

import javax.persistence.EntityManager;
import javax.persistence.LockModeType;
import javax.persistence.NoResultException;
import javax.persistence.Parameter;
import javax.persistence.Query;
import javax.persistence.TypedQuery;
import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.CriteriaQuery;
import javax.persistence.criteria.Order;
import javax.persistence.criteria.ParameterExpression;
import javax.persistence.criteria.Path;
import javax.persistence.criteria.Predicate;
import javax.persistence.criteria.Root;

import org.springframework.dao.EmptyResultDataAccessException;
import org.springframework.data.domain.Example;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageImpl;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.jpa.convert.QueryByExamplePredicateBuilder;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.data.jpa.provider.PersistenceProvider;
import org.springframework.data.jpa.repository.EntityGraph;
import org.springframework.data.jpa.repository.query.EscapeCharacter;
import org.springframework.data.jpa.repository.query.QueryUtils;
import org.springframework.data.jpa.repository.support.QueryHints.NoHints;
import org.springframework.data.repository.support.PageableExecutionUtils;
import org.springframework.data.util.ProxyUtils;
import org.springframework.lang.Nullable;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.util.Assert;

/**
 * Default implementation of the {@link org.springframework.data.repository.CrudRepository} interface. This will offer
 * you a more sophisticated interface than the plain {@link EntityManager} .
 *
 * @author Oliver Gierke
 * @author Eberhard Wolff
 * @author Thomas Darimont
 * @author Mark Paluch
 * @author Christoph Strobl
 * @author Stefan Fussenegger
 * @author Jens Schauder
 * @author David Madden
 * @author Moritz Becker
 * @param <T> the type of the entity to handle
 * @param <ID> the type of the entity's identifier
 */
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {

	private static final String ID_MUST_NOT_BE_NULL = "The given id must not be null!";

	private final JpaEntityInformation<T, ?> entityInformation;
	private final EntityManager em;
	private final PersistenceProvider provider;

	private @Nullable CrudMethodMetadata metadata;
	private EscapeCharacter escapeCharacter = EscapeCharacter.DEFAULT;
    ...
}
```

```java
@Override
public Optional<T> findById(ID id) {

    Assert.notNull(id, ID_MUST_NOT_BE_NULL);

    Class<T> domainType = getDomainClass();

    if (metadata == null) {
        return Optional.ofNullable(em.find(domainType, id));
    }

    LockModeType type = metadata.getLockModeType();

    Map<String, Object> hints = getQueryHints().withFetchGraphs(em).asMap();

    return Optional.ofNullable(type == null ? em.find(domainType, id, hints) : em.find(domainType, id, type, hints));
}
```

> JPA의 내부 기능 들을 활용하여 동작한다. EntityManager를 사용함

@Repository 를 사용하기 때문에 컴포넌트 스캔의 대상이 됨
- 예외를 Spring 예외로 변환해주기 때문에 구현기술을 바꿔도 예외 처리는 동일하게 됨 (PSA)

@Transacional(readOnly = true)
- Spring data JPA는 모든 기능에 트랜잭션을 걸고 시작하게 됨
- 서비스 계층에서 트랜잭션 단위를 설정하면 트랜잭션을 이어서 사용하게 됨
- 이 옵션을 사용하면 flush를 사용하지 않기 때문에 읽기전용 최적화가 됨

> @Transactional 애노테이션을 사용하면 JDBC에서 Connection 객체에 setAutoCommit(false)로 설정함

`S save(S entity)`
- 새로운 엔티티라면 (persist)
- 새로운 엔티티가 아니라면 병합 (merge)

> merge 는 되도록 사용하지 말것. 데이터 변경은 영속성 컨텍스트의 변경 감지를 활용
