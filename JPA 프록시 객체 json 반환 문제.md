# BookResponseDto (수정전)
```
@Getter
@AllArgsConstructor
public class BookResponseDto {
    private User user;

    private String name;

    private String desc;

    private Integer cost;

    private String author;

    private Long bookId;

    private BookResponseDto(Book book) {
        this.user = book.getUser();
        this.name = book.getName();
    ...
}
```

```
@Table(name = "book")
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class Book {

    @Id @GeneratedValue
    @Column(name = "book_id", updatable = false)
    private Long id;

    @Column(name = "name", nullable = false)
    private String name;

    @Column(name = "desc", nullable = false)
    private String desc;

    @Column(name = "cost", nullable = false)
    private Integer cost;

    @Column(name = "author", nullable = false)
    private String author;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private User user;
```

- user는 Book 엔티티 내부에 지연로딩으로 설정된 객체이기때문에 user를 가져올때 프록시로 가져오게 된다.  
- 그리고 BookResponseDto에서 user를 반환하려 하니 다음과 같은 에러가 발생한다.
```
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor and no properties discovered to create BeanSerializer
```
- 처음엔 프록시 객체 초기화가 안되서 그런줄 알고 강제 초기화해줬다. (ex, book.getUser().getName()) 하지만 그래도 동일한 문제는 발생한다.

### 문제1
- 해당 문제는 직접적인 원인은 ***프록시를 직렬화(Serialize) 하지 못해*** 발생하는 문제다. 이를 해결하는 방법은 여러가지가 있다.
  1. Hibernate5Module을 사용해서 초기화 하지 않은 프록시객체는 노출하지 않는다(초기화하면 노출 된다)
  ```
  @Bean
  Hibernate5Module hibernate5Module() {
  return new Hibernate5Module();
  }
  ```
  2. @JsonIgnore로 무시하기
  3. Lazy가 아닌 Eager 사용


### 문제2
- 사실 reponse로 엔티티를 반환하는 것부터가 잘못 된 설계다. 엔티티값의 모든값이 외부에 노출되는 것은 피해야한다. 

# BookResponseDto (수정후)
```
@Getter
@AllArgsConstructor
public class BookResponseDto {
    private String userName;

    private String name;

    private String desc;

    private Integer cost;

    private String author;

    private Long bookId;

    private BookResponseDto(Book book) {
        this.userName = book.getUser().getName();
        this.name = book.getName();
```

- 문제2의 원인을 해결하면 문제1은 자연스레 해결된다. 엔티티가 아닌 최소한의 데이터만 노출하여 문제를 해결하였다. 프록시 객체가 아닌 String 변수만 노출시켰다.
- 강의에 나와있는 내용이다. 역시 한번 듣고는 완벽히 이해할 수 없다. 이렇게 프로젝트를 진행하면서 직접 문제를 마주치니 훨씬 더 이해가 잘된다.
