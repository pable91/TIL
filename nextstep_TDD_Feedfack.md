

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


## 2. 배열보단 리스트

## 3. 플라이웨이트 패턴

## 4. enum
