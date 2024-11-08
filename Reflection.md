## 리플렉션
- 컴파일한 클래스 정보를 활용해 동적으로 프로그래밍이 가능하도록 지원하는 API
주로 런타임 시점에 동적으로 클래스 정보를 로드해서 사용한다.

## 어떻게 리플렉션이 가능하지?
- 자바에서는 JVM이 실행되면 사용자가 작성한 자바 코드가 컴파일러를 거쳐 바이트 코드로 변환되어 static 영역에 저장된다. Reflection API는 이 정보를 활용한다. 그래서 클래스 이름만 알고 있다면 언제든 static 영역을 뒤져서 정보를 가져올 수 있는 것이다.

## 언제 사용하는가?
- 보통은 애플리케이션 개발에서 사용하는 경우가 드물다. 하지만 프레임워크나 라이브러리에서 많이 사용한다.
프레임워크나 라이브러리에서는 사용자가 어떤 클래스를 만들지 예측할 수 없기 때문에 Reflection을 사용해야한다.
실제로 Spring에서 @Controller 등의 어노테이션을 붙인 클래스를 찾아서 인스턴스 생성 및 사용한다.

## 사용예
다음은 내가 스프링 프레임워크를 만들면서 Reflection을 사용한 부분이다.
    
    
1. 아래코드를 간단히 설명하자면, next.controller패키지에서 @Controller가 붙은 클래스들을 모두 찾아서 각각 클래스들의 인스턴스를 생성해서 저장하는 코드다.
```
       Reflections reflections = new Reflections("next.controller");
        Set<Class<?>> classes = reflections.getTypesAnnotatedWith(Controller.class);
        for (Class<?> clazz : classes) {
            Object instance = clazz.newInstance();
            mapByClass.put(clazz, instance);
        }
```
   
2. 이 코드는 1 코드에서 가져온 클래스에서(@Controller붙어있는) @RequestMapping이 붙은 method들을 찾아서 저장하는 코드다. HandlerKey는 @RequestMapping 속성으로 있는 value와 method를 포함하는 객체다. 
```
        Set<Method> methods = ReflectionUtils.getAllMethods(clazz, ReflectionUtils.withAnnotation(RequestMapping.class));

        for(Method method : methods) {
            RequestMapping rm = method.getAnnotation(RequestMapping.class);
            HandlerKey handlerKey = createHandlerKey(rm);

            Object controller = controllerClass.get(clazz);
            HandlerExecution handlerExecution = new HandlerExecution(controller, method);
            handlerExecutions.put(handlerKey, handlerExecution);
        }
```


- 여기까지 진행되면 우리가 흔히 사용하는 다음과 같은 코드가 실행될 수 있다.
```
@Controller
public class UserController {

	@RequestMapping(value = "/users/create", method = RequestMethod.POST)
    	public ModelAndView createUser(HttpServletRequest req, HttpServletResponse resp) {
        		User user = new User(req.getParameter("userId"), req.getParameter("password"), req.getParameter("name"),
                	req.getParameter("email"));
        		log.debug("User : {}", user);
        		DataBase.addUser(user);
        		return new ModelAndView(new RedirectView("redirect:/"));
    	}
}
```

## 결론 
- 위와 같이 런타임에 해당 어노테이션이 붙은 클래스를 탐색하고 발견한다면, 리플렉션을 통해 해당 클래스의 인스턴스를 생성하고 필요한 필드를 주입하여 Bean Factory에 저장하는 식으로 사용이 된다. 

https://steady-coding.tistory.com/609   
https://tecoble.techcourse.co.kr/post/2020-07-16-reflection-api/
