![Tomcat, Servlet, Spring1](https://user-images.githubusercontent.com/22884224/199673336-d1459e7b-f81d-4388-ad0e-7c47d70036b4.png)


![Tomcat, Servlet, Spring2](https://user-images.githubusercontent.com/22884224/199673502-86a2a8fe-01b8-4f6f-872d-f99b2f2bf888.png)


# WAS, Tomcat, Serlvet, Spring
- WAS와 Spring의 관계에 대한 설명을 구글링해서 찾아보면 그림이 많이 다르다 -_- 그래서 헷갈리는 경우가 많은데 몇개의 아티클을 읽으면서 공통적으로 말하는 내용을 간략하게 정리하려고한다.위 그림 2개도 서로 조금은 다르니 참고용도로만 보면 될것같다.
  - 보통 Web Server와 Web Application Server 개별된 존재로 얘기한다. 맞는말이나 요즘엔 WAS안에 WebServer을 포함하는 식으로 구현된다.
  - WAS 안에는 WAS는 Server Container라고 불릴정도로 Servlet Container라는 핵심기술? 역할?이 존재한다. Server Container는 한마디로 Servlet의 라이프싸이클을 관리하는 저장소다
  - Servlet은 인터페이스이기 때문에 구현체가 존재하는데, Spring MVC가 웹 어플리케이션을 위해 특화된 Servlet 구현체이다.(= DispatcherServlet을 특화된 Servlet 구현체로 봐도 된다)
  - 결과적으로 사용자가 어떤 request를 보내면, WAS안에 있는 WebServer를 거쳐 DispatcherServlet으로 가서 처리 후 리턴하는 방식이다.
  
[출처] https://taes-k.github.io/2020/02/16/servlet-container-spring-container/
[출처] https://jojoldu.tistory.com/28
