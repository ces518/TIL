# JPA Multiple join
- QueryDSL join + EntityGraph 
- 1:1:N 의 관계에서 N+1 쿼리 발생시 다음과 같이 해결..
- 너무 간만에 써서 삽질을 많이한듯..
```java
// Article
@NamedEntityGraph(
    name = "articleWithFile",
    attributeNodes = {
        @NamedAttributeNode(value = "file", subgraph = "file")
    },
    subgraphs = {
        @NamedSubgraph(name = "file", attributeNodes = @NamedAttributeNode("fileDetails"))
    }
)
@Entity
@Getter @Setter
public class Article {
    @Id
    private Long id;

    private String title;

    private String contents;

    @OneToOne
    @JoinColumn(name = "file_id")
    private File file;
}

// File
@Entity
@Getter @Setter
public class File {
    @Id
    private Long id;

    @OneToMany(mappedBy = "file", fetch = FetchType.EAGER)
    private Set<FileDetail> fileDetails;
}

// FileDetail
@Entity
@IdClass(FileDetailId.class)
@Getter @Setter
public class FileDetail {
    @Id
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "file_id")
    private File file;

    @Id
    private Integer oder;
}

// FileDetailId
@Getter @EqualsAndHashCode
public class FileDetailId implements Serializable {
    private Integer file;
    private Integer oder;
}

public List<Article> findAll (String searchType, String searchTxt) {
    EntityGraph<?> entityGraph = entityManager.getEntityGraph("articleWithFile");
    return queryFactory.selectFrom(article)
            .innerJoin(article.file, QFile.file)
            .leftJoin(QFile.file.fileDetails, QFileDetail.fileDetail)
            .distinct()
            .where(likeTitle(searchType, searchTxt),
                    likeContent(searchType, searchTxt))
            .setHint("javax.persistence.fetchgraph", entityGraph)
            .fetch();
}

private BooleanExpression likeTitle (String searchType, String searchTxt) {
    if (!"1".equals(searchType)) {
        return null;
    }
    return relic.title.like("%" + searchTxt + "%");
}

private BooleanExpression likeContent (String searchType, String searchTxt) {
    if (!"2".equals(searchType)) {
        return null;
    }
    return relic.content.like("%" + searchTxt + "%");
}
```
