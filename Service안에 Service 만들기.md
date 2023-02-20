# Service 코드(수정전)
```
@Service
@Transactional
@RequiredArgsConstructor
public class BookUpdateService {

    private final BookRepository bookRepository;
    private final UserRepository userRepository;

    public BookResponseDto registerBook(final BookRequestDto.Add addBookRequestDto) {
        final User findUser = validate(addBookRequestDto);

        final Book newBook = Book.registerBook(addBookRequestDto, findUser);

        return BookResponseDto.of(bookRepository.save(newBook));
    }

    private User validate(final BookRequestDto.Add addBookRequestDto) {
        final User findUser = userRepository.findById(addBookRequestDto.getUserId()).orElseThrow(() -> {
            throw new UserNotFoundException(addBookRequestDto.getUserId(), UserErrorCode.USER_NOT_FOUND);
        });

        Optional<Book> findBook = bookRepository.findByNameAndUser(addBookRequestDto.getName(), findUser);
        if (findBook.isPresent()) {
            throw new BookAlreadyRegisterException(BookErrorCode.BOOK_ALREADY_REGISTER, addBookRequestDto.getName());
        }

        return findUser;
    }

    public BookResponseDto updateBook(final BookRequestDto.Update updateBookRequestDto) {
        final Book findBook = notFoundBookValidate(updateBookRequestDto.getBookId());
        if (!findBook.isMine(updateBookRequestDto.getUserId())) {
            throw new IsNotMyBookException(BookErrorCode.IS_NOT_MY_BOOK, updateBookRequestDto.getBookId());
        }

        findBook.update(updateBookRequestDto);

        return BookResponseDto.of(findBook);
    }

    public BookResponseDto deleteBook(final Long bookId) {
        final Book findBook = notFoundBookValidate(bookId);

        bookRepository.delete(findBook);

        return BookResponseDto.of(findBook);
    }

    private Book notFoundBookValidate(final Long bookId) {
        return bookRepository.findById(bookId).orElseThrow(() -> {
            throw new NotFoundBookException(BookErrorCode.BOOK_NOT_FOUND, bookId);
        });
    }
}

```

```
@Service
@Transactional
@RequiredArgsConstructor
public class OrderBuyService {

    private final BookRepository bookRepository;
    private final UserRepository userRepository;
    private final OrderRepository orderRepository;

    public OrderResponseDto createOrder(final OrderRequestDto.Buy requestDto) {
        final User findUser = userRepository.findById(requestDto.getUserId())
                .orElseThrow(() -> {
                    throw new UserNotFoundException(requestDto.getUserId(), UserErrorCode.USER_NOT_FOUND);
                });

        final Book findBook = bookRepository.findById(requestDto.getBookId()).orElseThrow(() -> {
            throw new NotFoundBookException(BookErrorCode.BOOK_NOT_FOUND, requestDto.getBookId());
        });

        if (findBook.isMine(requestDto.getUserId())) {
            throw new BuyNotMyBookException(BookErrorCode.BUY_NOT_MY_BOOK, requestDto.getBookId());
        }

        if(findBook.isSold()) {
            throw new BookAlreadySoldException(BookErrorCode.BOOK_ALREADY_SOLD, requestDto.getBookId());
        }

        Order order = Order.createOrder(findBook);
        orderRepository.save(order);

        return OrderResponseDto.of(findBook, findUser.getName());
    }
}
```

### 문제점
- 중복코드가 존재한다. 예를 들어 bookRepository.findById는 여러 Service 코드에 존재한다. 하지만 현재 코드는 내가 직접 작성하기때문에 쓸데없는 중복코드가 발생한다. 그리고 같은 코드를 작성하기때문에 버그가 발생될 가능성도 있다.

### 해결책
- Repository와 1:1 매핑되는 또 하나의 Service 계층을 구현하였다(네이밍은 ~QueryService로 구현하였음)
- 이렇게 하면 기존 여러 Service들에서 ~QueryService의 메소드를 재사용하기때문에 중복코드가 제거된다. 즉, 재사용성이 증가와 코드가 좀 더 깔끔해진다.
- 하지만 Service 계층을 새롭게 추가했기때문에 해당 코드도 테스트 코드 작성을 해야하는 번거로움이 있었다.

# Service 코드(수정후)
```
@Service
@RequiredArgsConstructor
public class BookFindQueryService {

    private final BookRepository bookRepository;

    public Book findById(final Long bookId) {
        return bookRepository.findById(bookId).orElseThrow(() -> {
            throw new NotFoundBookException(BookErrorCode.BOOK_NOT_FOUND, bookId);
        });
    }
}
```

```
@Service
@Transactional
@RequiredArgsConstructor
public class BookUpdateService {

    private final UserFindQueryService userFindQueryService;
    private final BookFindQueryService bookFindQueryService;
    private final BookUpdateQueryService bookUpdateQueryService;

    public BookResponseDto registerBook(final BookRequestDto.Add addBookRequestDto) {
        final User findUser = validate(addBookRequestDto);

        final Book newBook = Book.registerBook(addBookRequestDto, findUser);

        return BookResponseDto.of(bookUpdateQueryService.save(newBook));
    }

    private User validate(final BookRequestDto.Add addBookRequestDto) {
        final User findUser = userFindQueryService.findById(addBookRequestDto.getUserId());

        if (bookFindQueryService.isExist(addBookRequestDto, findUser)) {
            throw new BookAlreadyRegisterException(BookErrorCode.BOOK_ALREADY_REGISTER, addBookRequestDto.getName());
        }
        return findUser;
    }
    
    ...
    ...
    ...
```
