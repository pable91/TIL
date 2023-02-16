- 내 코드는 클라이언트에서 받은 DTO를 Entity로 변환할때 Service 계층에서 진행한다. 그런데 Service slice테스트 코드를 작성할때 DTO를 직접 세팅해야하니 너무 귀찮게 느껴졌다.
그래서 "굳이 service 계층에서 dto->entity 작업을 해야할까? Controller에서 하고 Serivce에서는 그냥 엔티티만 넘겨받은면 안돼?" 라는 생각을 하게 되었고 구글링을 해보니 나와 같은 고민을 한 사람들이 많이 있었다.

```
  @BeforeEach
    public void init() {
        AddMemberRequestDto addMemberRequestDto = AddMemberRequestDto.builder()
                .name("kim")
                .build();
        user = User.from(1L, addMemberRequestDto.getName());

        addBookRequestDto = BookRequestDto.Add.builder()
                .name("kim")
                .desc("desc")
                .cost(1000)
                .author("author")
                .userId(1L)
                .build();

        updateBookRequestDto = BookRequestDto.Update.builder()
                .userId(1L)
                .bookId(1L)
                .name("update name")
                .desc("update desc")
                .cost(10000000)
                .author("update author")
                .build();
    }

    @Test
    @DisplayName("판매할 책 등록  테스트")
    public void addBookTest() {
        // give
        when(userRepository.findById(any())).thenReturn(Optional.of(user));

        when(bookRepository.save(any(Book.class))).thenReturn(Book.of(addBookRequestDto, user));
//        doReturn(Book.of(addBookRequestDto, newUser)).when(bookRepository)
//                .save(any(Book.class));

        // when
        BookResponseDto bookResponseDto = bookUpdateService.addBookService(addBookRequestDto);

        // then
        Assertions.assertThat(bookResponseDto.getName()).isEqualTo("kim");
        Assertions.assertThat(bookResponseDto.getDesc()).isEqualTo("desc");
        Assertions.assertThat(bookResponseDto.getCost()).isEqualTo(1000);
        Assertions.assertThat(bookResponseDto.getAuthor()).isEqualTo("author");
    }
```

### Controller에서 변환
- 장점
  - Controller에서 변환하고 엔티티를 Service로 넘기면, Service가 dto에 의존하지 않기때문에 해당 Service를 다른 Controller에서도 재사용가능해진다.
- 단점
  - 하지만 DTO와 Entity 변환이라는 비즈니스 로직이 Controller 계층에 포함되고 심지어 dto만으로는 완전한 엔티티를 만들지 못하는 경우 많다.
  - 또한 컨트롤러의 Entity는 영속된 상태가 아닐 수 있기때문에 LazyInitializationException을 발생시킨 위험도 있다.

### Service에서 변환
- 장점
  - Controller에서 비즈니스로직을 없게 할 수 있다. 도메인 캡슐화가 가능하다.
- 단점
  - Dto에 의존하기떄문에 Service의 재사용성이 떨어진다.
  - Service계층이 비대해진다.
  - LazyInitializationException 발생을 예방 가능하다.

[출처] https://stella-ul.tistory.com/163
[출처] https://velog.io/@jsb100800/Spring-boot-DTO-Entity-%EA%B0%84-%EB%B3%80%ED%99%98-%EC%96%B4%EB%8A%90-Layer%EC%97%90%EC%84%9C-%ED%95%98%EB%8A%94%EA%B2%8C-%EC%A2%8B%EC%9D%84%EA%B9%8C
[출처] https://sedangdang.tistory.com/296
