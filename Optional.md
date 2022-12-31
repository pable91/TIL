Optional이란?
- NullPointerException 을 방지할 수 있도록 도와주는 Wrapper(어떤 값을 감싸주는 클래스) 클래스

- Optional의 내부는 다음과 같이 저장하기 때문에 null이더라도 바로 NPE가 발생하지 않는다.

```
public final class Optional<T> {

  // If non-null, the value; if null, indicates no value is present
  private final T value;
   
  ...
}
```

***Optional은 값을 Wrapping하고 다시 풀고, null일 경우에는 대체하는 함수를 호출하는 오버헤드가 있으므로 무분별하게 잘못 사용하면 시스템 성능이 저하될 수 있다. 그러므로 절대 null이 나올 수 없는 값이라면 Optional을 사용하지 않는것이 좋다. 즉, Optional은 메소드의 결과가 null이 될 수 있으며, null에 의해 오류가 발생할 가능성이 매우 높은때 반환값으로 사용되어야한다.***

[출처] https://mangkyu.tistory.com/70
