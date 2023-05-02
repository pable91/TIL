# 이전 상황
- MVC 계층마다 log를 삽입하기로 했다.
- log를 삽입할때 원본코드의 수정이 일어나지 않기 위해 프록시 개념을 사용하였다.
- 인터페이스 기반 프록시 vs 구체클래스 기반 프록시
https://github.com/pable91/TIL/blob/main/AOP1(%ED%94%84%EB%A1%9D%EC%8B%9C).md

# 문제
- 계층마다 프록시 클래스를 각각 구현해줘야해서 비효율적이다. 클래스들을 너무 많이 만들어야한다.
- ex) 프록시 적용 대상 계층이 100개 => 직접 구현해야하는 프록시 클래스 100개

# 동적 프록시
- 동적프록시를 사용하면 자동(=런타임)으로 프록시 클래스를 만들어준다. 개발자가 직접 구현할 필요가 없다.
- ***인터페이스 기반 -> JDK 동적 프록시***
- ***구체클래스 기반 -> CGLIB***

# JDK 동적 프록시
- JDK 동적 프록시는 인터페이스 기반에서 동작하기때문에 인터페이스가 필히 있어야한다.
- JDK 동적 프록시에 적용할 로직은 InvocationHandler 인터페이스를 구현해서 작성해야한다.
``` java
public interface InvocationHandler {
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
}

Object proxy : 프록시 자신
Method method : 호출한 메서드
Object[] args : 메서드를 호출할 때 전달한 인수
```

# JDK 동적 프록시 적용 전
``` java
public class ControllerProxy implements ControllerInterface {

    private ControllerInterface target;
    private LogPrinter logPrinter;

    public ControllerProxy(ControllerInterface controllerInterface, LogPrinter logPrinter) {
        this.target = controllerInterface;
        this.logPrinter = logPrinter;
    }

    @Override
    public String func() {
        logPrinter.print("controller");
        return target.func();
    }
}
```
``` java
public class ServiceProxy implements ServiceInterface {

    private ServiceInterface target;
    private LogPrinter logPrinter;

    public ServiceProxy(ServiceInterface target, LogPrinter logPrinter) {
        this.target = target;
        this.logPrinter = logPrinter;
    }

    @Override
    public String func() {
        logPrinter.print("service");
        return target.func();
    }
}

```

# JDK 동적 프록시 적용 후
- 이제 더이상 Controller, Service, Repository 각각 프록시 클래스를 만들 필요가 없다. 하나만 만들면 모든 계층에 적용된다.
``` java
public class LogPrinterBasicHandler implements InvocationHandler {

    private final Object target;
    private final LogPrinter logPrinter;

    public LogPrinterBasicHandler(Object target, LogPrinter logPrinter) {
        this.target = target;
        this.logPrinter = logPrinter;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String message = method.getDeclaringClass().getSimpleName();
        logPrinter.print(message);

        Object result = method.invoke(target, args);

        return result;
    }
}
```

# 동적 프록시 설정 변화
- Before
``` java
@Configuration
public class InterfaceConfig {

    @Bean
    public ControllerInterface controllerInterface(LogPrinter logPrinter) {
        ControllerImpl testController = new ControllerImpl(serviceInterface(logPrinter));
        ControllerProxy controllerProxy = new ControllerProxy(testController, logPrinter);
        return controllerProxy;
    }

    @Bean
    public ServiceInterface serviceInterface(LogPrinter logPrinter) {
        ServiceImpl testService = new ServiceImpl(repositoryInterface(logPrinter));
        ServiceProxy serviceProxy = new ServiceProxy(testService, logPrinter);
        return serviceProxy;
    }

    @Bean
    public RepositoryInterface repositoryInterface(LogPrinter logPrinter) {
        RepositoryImpl testRepository = new RepositoryImpl();
        RepositoryProxy repositoryProxy = new RepositoryProxy(testRepository, logPrinter);
        return repositoryProxy;
    }
```

- After
``` java
@Configuration
public class DynamicProxyBasicConfig {
    @Bean
    public ControllerInterface controllerInterface(LogPrinter logPrinter) {
        ControllerInterface controllerInterface = new ControllerImpl(serviceInterface(logPrinter));

        ControllerInterface proxy = (ControllerInterface) Proxy.newProxyInstance(
                ControllerInterface.class.getClassLoader(),
                new Class[]{ControllerInterface.class},
                new LogPrinterBasicHandler(controllerInterface, logPrinter)
        );
        return proxy;
    }

    @Bean
    public ServiceInterface serviceInterface(LogPrinter logPrinter) {
        ServiceInterface serviceInterface = new ServiceImpl(repositoryInterface(logPrinter));

        ServiceInterface proxy = (ServiceInterface) Proxy.newProxyInstance(
                ServiceInterface.class.getClassLoader(),
                new Class[]{ServiceInterface.class},
                new LogPrinterBasicHandler(serviceInterface, logPrinter)
        );
        return proxy;
    }

    @Bean
    public RepositoryInterface repositoryInterface(LogPrinter logPrinter) {
        RepositoryInterface repositoryInterface = new RepositoryImpl();

        RepositoryInterface proxy = (RepositoryInterface) Proxy.newProxyInstance(
                RepositoryInterface.class.getClassLoader(),
                new Class[]{RepositoryInterface.class},
                new LogPrinterBasicHandler(repositoryInterface, logPrinter)
        );
        return proxy;
    }
}
```
- Proxy.newProxyInstance라는 java 리플렉션 기술로 ***Proxy***를 생성한다. 그리고 생성할때 LogPrinterBasicHandler에는 모두 다른 인자(repository, service, controller)들이 들어간다. 이를 통해서 동적으로 프록시 생성이 가능해 진다.

# JDK 동적 프록시 사용시 의존관계
![의존관계](https://user-images.githubusercontent.com/22884224/235561822-a2a050b0-1868-4f49-9856-012b5a87216d.png)   
![런타임 의존관계](https://user-images.githubusercontent.com/22884224/235561830-ca291dcc-24bd-4a55-ba0e-ec26918881f2.png)
