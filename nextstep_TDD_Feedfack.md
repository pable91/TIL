

## 1. 전략 패턴

### 기존
``` java
public class Car {

    private static final int BOUNDARY = 4;

    private String distance = "";

    public String move(int value) {
        if (value >= BOUNDARY) {
            this.distance += "-";
            return "-";
        }

        return "";
    }

    public String getDistance() {
        return this.distance;
    }
}
```

### 피드백
``` 
Car boundary 값에 대한 요구사항이 바뀌면, 테스트 코드도 모두 변경될 것 같은데요!
Boundary 에 대한 의존성을 제거하고, 테스트할 수 있도록 리펙토링해보는 것은 어떨까요?
의존성이 제거되면 어떤 이점이 있을까요?
```

### 솔루션
``` 
같은 기능이지만 서로다른 전략을 사용할 수 있는 전략 패턴을 사용하였다.
Car가 Move한다는 동작을 추상화 인터페이스로 정의하고, 해당 인터페이스를 상속 받은 클래스에서 실제 전략들을 구현하였다.
내 코드에서는 "움직인다"를 인터페이스로 정의하고, "특정 랜덤값에 의해 움직인다"라는 내게 필요한 전략을 구현해서 사용했다.
``` 

### 수정 후
``` java
package racingCar.domain;

import racingCar.MovableStrategy;

public class Car {

    private final MovableStrategy movableStrategy;

    private CarName name;
    private Position position;

    public Car(String name, MovableStrategy movableStrategy) {
        this.position = new Position();
        this.name = new CarName(name);
        this.movableStrategy = movableStrategy;
    }

    public int move(int value) {
        if (movableStrategy.canMove(value)) {
            return this.position.forward();
        }

        return 0;
    }

    public int getPosition() {
        return this.position.getPosition();
    }

    public String getName() {
        return name.getName();
    }
}
```

### 결론
- Car라는 클래스 안에서 어떠한 하나의 전략에 의존하지 않아도 된다. 
즉, 요구사항이 변경됐을때 Car내부에 있는 코드를 수정하지 않아도 된다.
- 테스트 코드를 작성하기 편하다. 하지만 아쉽게도 위의 내 코드는 테스트 작성의 이점을 살리지 못했다. 
move라는 메소드에서 매개변수로 값을 받는게 아니라 전략에 대한 인터페이스를 매개변수로 받았다면 테스트 코드작성이 쉽다는 장점을 취했을 것이다. 
[이곳](https://jackjeong.tistory.com/108)처럼 작성하는것이 전략패턴의 테스트코드 장점을 살린것이다.

## 2. == VS equals() 의 차이점

### 기존
``` java
    private static boolean isInvalidExp(String exp) {
        return exp == null || exp == "";
```

### 피드백
```
String 클래스가 제공하는  == 메소드를 활용하는건 어떨까요?
```

### 솔루션
```
String 변수를 생성할때는 두가지 방법이 있다.
    1. 리터럴 초기화 방법
    2. new 연산자를 이용한 방법

리터럴 초기화는 string constant pool에 해당 리터럴이 존재하는지 확인하고 기존에 존재했으면 해당 주소값을 반환해서 공유해서 사용한다.
new 연산자는 생성하면 heap영역에 할당되는데 같은 문자열을 생성해도 각각 다른 heap 메모리에 할당된다.

그리고 '==' 비교는 두개의 주소값을 비교하는 반면 string클래스의 'equals'비교는 값 자체를 비교한다.

그럼 같은 문자열을 리터럴 초기화 방법과 new 연산자 방법을 사용해서 각각 생성 한뒤, 그 두개를 '=='로 비교한다면 어떻게 될까?
당연히 할당된 메모리 위치가 다르기 떄문에 같은 문자열이라 할지라도 비교결과는 false가 될 것이다.
new 연산자로 두개 생성해서 '=='로 비교해도 결과는 false가 될 것이다.

결과적으로 문자열 동등비교는 string 클래스의 'equals'를 사용해서 값 자체를 동등비교해야지 우리가 원하는 결과가 나올 것이다. 

https://coding-factory.tistory.com/536

```

## 3. 테스트 코드는 내부 구현보다 최종 결과를 검증
### 피드백
```
클라이언트가 직접으로 필요로하는 executeCalc()메소드만 공개하고 
compute() 메소는 구현 세부사항이이 때문에 private으로 변경하는게 좋다고 생각해요.

테스트코드에서 내부구현을 검증할 경우 프로덕션 코드의 변경이 될 때 테스트코드도 깨지기 쉽기때문이에요. 이는 바람직하지 않다고 생각합니다.

compute() 메소드에 대한 테스틑 executeCalc() 메소드를 통해서도 충분히 가능하지 않을까요?
```

### 솔루션
```
내부 구현을 검증하는 테스트들은 구현을 조금만 변경해도 기존 테스트가 깨질 수 있다.
내부 구현을 테스트하기보다는 내부구현을 포함한 최종 결과를 테스트 하는게 더 효율적일 수 있다.
```

## 2. 배열보단 리스트

## 3. 플라이웨이트 패턴

## 4. enum
