# JPA OneToOne 매핑시 주의

## @OneToOne
- JPA 에서 제공하는 1:1 관계 매핑 애노테이션
- 단방향, 양방향 모두 지원한다.
- **주인을 반대로 지정할 수 없다.**
    - 주인으로 지정하면 **반드시 외래키를 가진다.**
    - ex) (@OneToMany 이고, 연관관계의 주인 으로 지정) 과 같이 매핑이 불가능함
- 외래키를 가지지 않는 반대쪽에서 참조하려면 반드시 양방향 매핑을 사용해야 한다.
- 가장 큰 문제는 LAZY Loading 을 제대로 지원하지 않는다.
    - 정확히는 OneToOne 매핑 관계에서 연관관계의 주인 (외래키를 가지고 있는 곳) 에서만 LAZY 가 정상적으로 동작한다.

## 테스트 코드
- 아래 코드는 검증만을 위한 코드

`엔티티 `

```java
@Getter
@Setter
@Entity
public class Author {
	@Id
	@GeneratedValue
	private Long id;

	@OneToOne(fetch = FetchType.LAZY, mappedBy = "author")
	private Book book;
}

@Getter
@Setter
@Entity
public class Book {
	@Id
	@GeneratedValue
	private Long id;

	@OneToOne(fetch = FetchType.LAZY)
	@JoinColumn(name = "author_id")
	private Author author;
}
```

`테스트`

```java
@SpringBootTest
@Transactional
class JpaOnetooneApplicationTests {

	@Autowired
	AuthorRepository authorRepository;

	@Autowired
	BookRepository bookRepository;

	@Autowired
	EntityManager em;

	@Test
	void test() {
		List<Author> authors = List.of(new Author(), new Author(), new Author());
		List<Author> savedAuthors = authorRepository.saveAll(authors);

		Book book = new Book();
		book.setAuthor(authors.get(0));

		Book book1 = new Book();
		book1.setAuthor(authors.get(1));

		Book book2 = new Book();
		book2.setAuthor(authors.get(2));

		List<Book> books = List.of(book, book1, book2);

		List<Book> savedBooks = bookRepository.saveAll(books);

		em.flush();
		em.clear();

		System.out.println(" =============== find Authors ================== ");
		List<Author> findAuthors = authorRepository.findAll();
		for (Author findAuthor : findAuthors) {
			System.out.println("findAuthor.getId() = " + findAuthor.getId());
			System.out.println("findAuthor.getBook() = " + findAuthor.getBook());
		}

		System.out.println(" =============== find Books ================== ");

		em.flush();
		em.clear();

		List<Book> findBooks = bookRepository.findAll();

		for (Book findBook : findBooks) {
			System.out.println("findBook.getId() = " + findBook.getId());
			System.out.println("findBook.getAuthor() = " + findBook.getAuthor());
		}
	}

}
```

`실행 결과`

```shell
 =============== find Authors ================== 
Hibernate: 
    select
        author0_.id as id1_0_ 
    from
        author author0_
Hibernate: 
    select
        book0_.id as id1_1_0_,
        book0_.author_id as author_i2_1_0_ 
    from
        book book0_ 
    where
        book0_.author_id=?
Hibernate: 
    select
        book0_.id as id1_1_0_,
        book0_.author_id as author_i2_1_0_ 
    from
        book book0_ 
    where
        book0_.author_id=?
Hibernate: 
    select
        book0_.id as id1_1_0_,
        book0_.author_id as author_i2_1_0_ 
    from
        book book0_ 
    where
        book0_.author_id=?
findAuthor.getId() = 1
findAuthor.getBook() = me.june.jpaonetoone.Book@576323ff
findAuthor.getId() = 3
findAuthor.getBook() = me.june.jpaonetoone.Book@71806c64
findAuthor.getId() = 5
findAuthor.getBook() = me.june.jpaonetoone.Book@69ae7632
 =============== find Books ================== 
Hibernate: 
    select
        book0_.id as id1_1_,
        book0_.author_id as author_i2_1_ 
    from
        book book0_
findBook.getId() = 2
Hibernate: 
    select
        author0_.id as id1_0_0_ 
    from
        author author0_ 
    where
        author0_.id=?
findBook.getAuthor() = me.june.jpaonetoone.Author@346a2799
findBook.getId() = 4
Hibernate: 
    select
        author0_.id as id1_0_0_ 
    from
        author author0_ 
    where
        author0_.id=?
findBook.getAuthor() = me.june.jpaonetoone.Author@4a490518
findBook.getId() = 6
Hibernate: 
    select
        author0_.id as id1_0_0_ 
    from
        author author0_ 
    where
        author0_.id=?
findBook.getAuthor() = me.june.jpaonetoone.Author@167bae0b
```
- 연관관계의 주인으로 설정된 Book Entity 에서만 LAZY Loading 이 적용되는걸 확인할 수 있다.

## 해결책 ?
- 양방향 매핑은 피하면 좋으나... 어쩔수 없이 1:1 매핑을 사용해야 한다면 양방향 매핑을 사용해야 한다..
    - 되도록 단방향 관계를 유지하는 것이 좋다.
- 차선책으로 @OneToOne -> @OneToMany 로 지정하여, 연관관계를 반대로 지정 할 수 있게 허용하는 방법도 있다.
    - 이 경우 매핑은 OneToMany 이지만, 실 데이터 매핑은 1:1로 유지되도록 해주어야 한다
