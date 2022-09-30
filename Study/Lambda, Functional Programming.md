# 람다
람다식은 "이름없는 익명함수"를 뜻한다.

### 기존 함수 
```
String hello() {
    return "Hello World"; 
}
```

```
int min(int x, int y) {
    return x < y ? x : y;
}
```

### 람다식으로 바꾸면
```
() -> { "Hello World" };
```

```
(x, y) -> x < y ? x : y;
```

람다표현식으로 사용하면 클래스를 작성하고 객체를 생성하지 않아도 메소드를 사용할 수 있다.

### 람다 특징
1. 람다가 등장한 이유는 불필요한 코드를 줄이고, 가독성을 높이기 위함이다.
2. 또한 람다식은 함수형 인터페이스의 변수로 사용이 가능하다. 람다식의 반환값이 함수형 인터페이스이기 때문이다. (이 특징으로 stream api의 매개변수로 사용이 가능하다.)
3. 람다식은 함수형 프로그래밍에서 유용하게 쓰이고. 1급객체이자, 순수 함수다.

---
---
---

# 함수형 인터페이스
- 함수형 인터페이스는 추상 클래스와는 달리 단 1개의 추상 메서드만을 가지고 있는 인터페이스다. 
- 그리고 람다식 반환값은 함수형 인터페이스이기때문에 다음과 사용할 수 있다.
```
함수형 인터페이스변수 = 람다식;
```

### 함수형 인터페이스 사용 예
```
interface Calc { // 함수형 인터페이스의 선언
    public int min(int x, int y);

}

public class Lambda02 {
public static void main(String[] args){
        Calc minNum = (x, y) -> x < y ? x : y; // 추상 메소드의 구현
        System.out.println(minNum.min(3, 4));  // 함수형 인터페이스의 사용
    }
}
```

### @FunctionalInterface
- 함수형 인터페이스에는 @FunctionalInterface 를 붙일 수 있다. 해당 어노테이션이 있으면 메서드가 2개 이상 선언되면 컴파일 오류를 발생시키기때문에
함수형 인터페이스를 정의할때 붙여주면 좋다.

```
@FunctionalInterface
interface Calc { 
    public int min(int x, int y);
}
```

### Java에서 제공하는 함수형 인터페이스
Java에는 자주 사용될 것 같은 함수형 인터페이스가 이미 정의되어 있으며, 총 4가지 함수형 인터페이스를 지원하고 있다.
- Supplier<T>
- Consumer<T>
- Function<T, R>
- Predicate<T>
    
    
https://mangkyu.tistory.com/113   
http://www.tcpschool.com/java/java_lambda_concept   
