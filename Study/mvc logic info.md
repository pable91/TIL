# LegacyHandlerMapping
기존 프로젝트는 클라이언트 요청 url에 대해 처리를 담당하는 각각의 Controller들을 가지고 있었다(이하 Legacy Handler Mapping)

```
 void initMapping() {
        mappings.put("/", new HomeController());
        mappings.put("/users/form", new ForwardController("/user/form.jsp"));
        mappings.put("/users/loginForm", new ForwardController("/user/login.jsp"));
        mappings.put("/users", new ListUserController());
        mappings.put("/users/login", new LoginController());
        mappings.put("/users/profile", new ProfileController());
        mappings.put("/users/logout", new LogoutController());
        mappings.put("/users/create", new CreateUserController());
        mappings.put("/users/updateForm", new UpdateFormUserController());
        mappings.put("/users/update", new UpdateUserController());

        logger.info("Initialized Request Mapping!");
        mappings.keySet().forEach(path -> {
            logger.info("Path : {}, Controller : {}", path, mappings.get(path).getClass());
        });
    }
```

이 방법은 컨트롤러가 추가될 때마다 매번 (요청 URL) 과 (컨트롤러)를 수동으로 추가하는 작업이 필요하다.
또한 추가하는 번거로움 동시에 Controller 구현체의 갯수도 계속 많아지기 때문에 유지보수성도 좋지않다.

# LegacyHandlerMapping -> AnnotationHandlerMapping
이러한 단점을 보완하기 위해 새로운 기능이 추가될 때마다 매번 컨트롤러를 추가하는 것이 아니라, Controller 하나에 연관있는 메서드들을 추가해서 사용하는것이 좋다. (이하 Annotation Handler Mapping)
우리가 흔히 SpringFramework에서 사용하는 방법처럼...

```
    @RequestMapping("/users")
    public ModelAndView list(HttpServletRequest request, HttpServletResponse response) {
        logger.debug("users findUserId");
        return new ModelAndView(new JspView("/users/list.jsp"));
    }
   
    @RequestMapping(value="/users/show", method=RequestMethod.GET)
    public ModelAndView show(HttpServletRequest request, HttpServletResponse response) {
        logger.debug("users findUserId");
        return new ModelAndView(new JspView("/users/show.jsp"));
    }
   
    @RequestMapping(value="/users", method=RequestMethod.POST)
    public ModelAndView create(HttpServletRequest request, HttpServletResponse response) {
        logger.debug("users create");
        return new ModelAndView(new JspView("redirect:/users"));
    }
```

위의 방법대로 바꾸려면 @Controller, @RequestMapping 등의 어노테이션을 활용해야한다.
즉, 레거시 매핑 프레임워크를 어노테이션 매핑 프레임워크로 바꾸는 것이다.

바꾸는 과정을 내가 작성한 코드를 토대로 설명해보고자 한다.
상용 Spring Framework와는 살짝 다를 수도 있으니 유의하자.

# 1단계
@Controller 어노테이션이 붙은 Controller 클래스들을 자바 리플렉션을 활용해서 찾는다.
```
public ControllerScanner() throws InstantiationException, IllegalAccessException {
        Reflections reflections = new Reflections("next.controller");
        Set<Class<?>> classes = reflections.getTypesAnnotatedWith(Controller.class);
        for (Class<?> clazz : classes) {
            Object instance = clazz.newInstance();
            mapByClass.put(clazz, instance);
        }
    }
```

next.controller 하위 패키지에서 @Controller 어노테이션이 붙은 클래스들을 모두 가져온다.
그리고 순회하면서 클래스 정보를 key, 클래스 객체를 value로 해서 Map에 저장한다.

# 2단계
Controller 클래스 안에서 @RequestMapping 어노테이션이 붙어있는 메소드들을 전부 찾는다.


```
private void initHandlerExecutions(Class<?> controllerClass, Object controllerObj) {
        Set<Method> methods = ReflectionUtils.getAllMethods(controllerClass, ReflectionUtils.withAnnotation(RequestMapping.class));

        for(Method method : methods) {
            HandlerKey handlerKey = createHandlerKey(method);
            
            HandlerExecution handlerExecution = new HandlerExecution(controllerObj, method);
            handlerExecutions.put(handlerKey, handlerExecution);
        }
    }
```

```
private HandlerKey createHandlerKey(Method method) {
        Class<?> declaringClass = method.getDeclaringClass();
        Controller controllerAnnotation = declaringClass.getAnnotation(Controller.class);
        String controllerPath = controllerAnnotation.value();

        RequestMapping rm = method.getAnnotation(RequestMapping.class);

        return new HandlerKey(controllerPath + rm.value(), rm.method());
    }
```

찾은 메소드들에서 (url정보와 http method정보를 더해서 key)로 만들고,
(controller 클래스 정보와 method 클래스 정보를 더해서 value)로 만들어서 Map으로 저장한다.
결국에는 사용자 요청이 오면 Map에서 요청정보에 해당하는 데이터를 찾아서 실행 시킬 것이다.

여기까지가 초기화 단계이다. 이제 실제로 사용자 요청이 들어왔을때 어떻게 동작하는지 살펴보자.

# 3단계
사용자 요청 정보가 DispatcherServler(=Front Controller)에 들어오면 LegacyMapping과 AnnotationMapping를 모두 순회하면서
요청정보(url, http method)에 해당하는 value값을 찾을 것이다. 여기서 value는 handler라고 한다. 이는 getHandler라는 메소드에서 담당한다.

```
private Object getHandler(HttpServletRequest request) {
        for (HandlerMapping mapping : mappingList) {
            Object handler = mapping.getHandler(request);
            if (handler != null) {
                return handler;
            }
        }

        return null;
    }
```

# 4단계
요청에 해당하는 handler를 찾은 후 어탭터 패턴을 사용해서 현재 handler의 실행 방식을 결정할 것이다.

```
...
HandlerAdapter adapter = adapterList.stream()
                .filter(ad -> ad.support(handler))
                .findFirst()
                .orElseThrow(() -> {
                    throw new NotFoundHandlerException("handler 를 찾을 수 없습니다");
                });
...
```

핸들러 어댑터를 좀 더 설명하자면 LegacyMapping과 AnnotationMapping의 실행방법은 다를 것이다.
LegacyMapping은 각각 요청에 맞는 컨트롤러를 호출하고, AnnotationMapping은 각각 요청에 맞는 메소드 정보를 찾아서 리플렉션을 활용해 호출할 것이다.
즉, 기본적으로 실행하는 메커니즘 자체가 다르다.
그래서 실행하는 각각의 메커니즘을 Adaptor들에 구현하였고, AdaptorList를 순회하면서 현재 handler가 실행가능한 Adaptor를 찾아서 실행시키는 것이다.

여기서 문득 들었던 생각은 Adaptor를 각각 만들지 않고 LegacyMapping과 AnnotationMapping 클래스에서 Adaptor들이 하는 작업을 모두 넣었으면 어땠을까?
가장 먼저 들었던 생각은 책임을 너무 집중시켜서 Mapping 클래스 자체의 역할이 많아지는것이 단점인것 같다. 
결국엔 어댑터 패턴을 사용해서 "실행부분"을 추상화 시켜 Adaptor 클래스를 만들었고, 책임을 분리시킨 것이 어댑터 패턴의 가장 큰 역할이자 장점이 아닐까 싶다.

여기까지 이제 완료하면 AnnotationMapping 기반의 프레임워크는 완성되었다.

# 5단계
어노테이션 기반의 프레임워크는 완료되었지만 좀 더 리팩토링 할 부분이 있다.
지금까지 작업한 내용에서는 HttpServletRequest, HttpServletResponse 라는 파라미터를 고정으로 사용했어야했다. 그래야지 getParameter나 getSession으로 내가 원하는 값을 호출할 수 있기때문이었다. 하지만 매번 HttpServletRequest에서 필요한 데이터를 꺼내오는건 번거롭다. 그냥 내가 원하는 파라미터를 메소드에 선언하면 알아서 값이 바인딩 되었으면 좋겠다.

그리고 실제 스프링 프레임워크처럼 @RequestBody, @PathVariable, @RequestParam도 사용하면 그에 맞는 기능이 동작했으면 좋겠다. 밑에 코드는 HttpServletRequest, HttpServletResponse를 무조건 써야하는 기존 코드다.

```
    @RequestMapping(value = "/users/create", method = RequestMethod.POST)
    public ModelAndView createUser(HttpServletRequest req, HttpServletResponse resp) {
        User user = new User(req.getParameter("userId"), req.getParameter("password"), req.getParameter("name"), req.getParameter("email"));
        log.debug("User : {}", user);
        DataBase.addUser(user);
        return new ModelAndView(new RedirectView("redirect:/"));
    }

    @RequestMapping(value = "/users/login", method = RequestMethod.POST)
    public ModelAndView login(HttpServletRequest req, HttpServletResponse resp) {
        String userId = req.getParameter("userId");
        String password = req.getParameter("password");
        User user = DataBase.findUserById(userId);
        if (user == null) {
            ModelAndView mv = new ModelAndView(new ForwardView("/user/login.jsp"));
            mv.addObject("loginFailed", true);
            return mv;
        }

        if (user.matchPassword(password)) {
            HttpSession session = req.getSession();
            session.setAttribute(UserSessionUtils.USER_SESSION_KEY, user);
            return new ModelAndView(new RedirectView("redirect:/"));
        } else {
            ModelAndView mv = new ModelAndView(new ForwardView("/user/login.jsp"));
            mv.addObject("loginFailed", true);
            return mv;
        }
    }
    
    @RequestMapping(value = "/users/logout", method = RequestMethod.GET)
    public ModelAndView logout(HttpSession session) {
        session.removeAttribute(UserSessionUtils.USER_SESSION_KEY);
        return new ModelAndView(new RedirectView("redirect:/"));
    }
    
    @RequestMapping(value = "/users/update", method = RequestMethod.POST)
    public ModelAndView update(HttpServletRequest req, HttpServletResponse resp)  {
        User user = DataBase.findUserById(req.getParameter("userId"));
        if (!UserSessionUtils.isSameUser(req.getSession(), user)) {
            throw new IllegalStateException("다른 사용자의 정보를 수정할 수 없습니다.");
        }

        User updateUser = new User(req.getParameter("userId"), req.getParameter("password"), req.getParameter("name"),
                req.getParameter("email"));
        log.debug("Update User : {}", updateUser);
        user.update(updateUser);

        return new ModelAndView(new RedirectView("redirect:/"));
    }
  
```

이런식으로 변경하려고 한다.
```
    @RequestMapping(value = "/users", method = RequestMethod.POST)
    public ModelAndView create_string(String userId, String password) {
        logger.debug("userId: {}, password: {}", userId, password);
        return null;
    }

    @RequestMapping(value = "/users", method = RequestMethod.POST)
    public ModelAndView create_int_long(long id, int age) {
        logger.debug("id: {}, age: {}", id, age);
        return null;
    }

    @RequestMapping(value = "/users", method = RequestMethod.POST)
    public ModelAndView create_javabean(TestUser testUser) {
        logger.debug("testUser: {}", testUser);
        return null;
    }


    @RequestMapping(value = "/users/{id}", method = RequestMethod.GET)
    public ModelAndView show_pathvariable(@PathVariable long id) {
        logger.debug("userId: {}", id);
        return null;
    }
```

이 기능의 핵심 로직을 설명하자면 현재 실행되어야하는 Hanlder가 정해진 후 해당 메소드에 정의된 파라미터들을 순회하면서 어떤 방법으로 실행될 수 있는지 확인한다. 여기서 어떤 방법이란 Resolver를 의미한다.
즉, 각각의 파라미터의 실행여부를 확인하고, 실행까지 할 수 있는 추상화된 클래스가 Resolver다.


# 6단계
```
public ModelAndView handle(HttpServletRequest request, HttpServletResponse response) throws Exception {
        ParameterNameDiscoverer nameDiscoverer = new LocalVariableTableParameterNameDiscoverer();
        String[] parameterNames = nameDiscoverer.getParameterNames(method);
        Parameter[] parameters = method.getParameters();

        Object[] parameterObjs = new Object[parameterNames.length];

        if(parameterObjs.length == 0) {
            return (ModelAndView) method.invoke(controller);
        }

        for (int i = 0; i < parameterObjs.length; ++i) {
            MethodParameter methodParameter = new MethodParameter(method, parameterNames[i], parameters[i]);
            parameterObjs[i] = getParameterObject(request, response, methodParameter);
        }

        return (ModelAndView) method.invoke(controller, parameterObjs);
    }
```

```
private Object getParameterObject(HttpServletRequest request, HttpServletResponse response, MethodParameter methodParameter) throws IOException {
        for (MethodArgumentResolver methodArgumentResolver : list) {
            if(methodArgumentResolver.supportsParameter(methodParameter)) {
                System.out.println("methodArgumentResolver -> {} " + methodArgumentResolver);
                return methodArgumentResolver.resolveArgument(methodParameter, request, response);
            }
        }

        throw new NotFoundResolverException("실행 할 수 있는 Method Resolver 가 없습니다.");
    }
```

- 메소드에 선언된 파라미터를 하나씩 확인한다(getParameterObject)
- 파라미터를 실행시킬 수 있는 resolver를 찾는다.(supportParameter)
- 선택된 resolver마다 각각 다른 실행 로직이 있을텐데, 실행 후 반환한다(resolveArgument) 

나는 Resolver들을 다음과 같이 구현했고, 실제로 spring에서는 내가 구현한것보다 훨씬 더 많은 Resolver들이 있다.
- HttpServlerArgumentResolver
- HttpSessionResolver
- PathVaribableArgumentResolver
- PrimitiveTypeArgumentResolver
- RequestBodyArgumentResolver
- UserObjectTypeArgumentResolver

모든 Resolver 구현체들은 MethodArgumentResolver라는 인터페이스를 상속하고, 구현체에서는 supportParameter와 resolveArgument 메소드를 구현해야한다.






