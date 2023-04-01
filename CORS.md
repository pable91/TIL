# 어떤 경우에 CORS 에러가 발생하는가?
- (CORS 설정이 따로 되어있지않은 상태에서) 프론트에서 ajax를 통해 네이버 지도 api 요청하는경우 또는 내가 백엔드 팀원이 개발한 api서버에 요청을 보내는 경우

# CORS(Cross-Origin Resource Sharing) 에러가 생기는 원인은 무엇인가?
- ***SOP 정책을 위반했기 때문이다***
- ***CORS를 사용하지 않았기 때문이다***

# SOP(Same Origin policy)
- SOP는 동일출처정책이라는 뜻으로 같은 출처(Origin)에서만 리소스를 공유할 수 있다. 출처가 다른 서버에 요청해서 리소스를 가지고 오는게 안된다는 정책이다.
	- 출처란? http://localhost:8080 같이 (프로토콜 + 도메인 + 포트번호) 를 의미한다. 
- 웹은 ***기본으로 SOP 정책***을 사용한다.

# 왜 SOP가 있어야하는가?
- 만약 다른 출처와 통신하는것이 아무런 제약도 없다면, 악의를 가진 사용자가 악성코드를 실행시키게 만들어서 우리의 정보를 탈취할 것이다. (ex, CSRF, XSS)
- 하지만 다른 회사의 서버 API를 이용해야 하는 경우가 많다. 그렇기때문에 ***무작정 막기 보다는 예외 조항을 두고 다른 출처의 리소스 요청을 허용하기로 하였다. 이것이 바로 CORS다.***

# CORS
- CORS는 ***SOP정책을 위반하더라도 다른 출처의 리소스를 허용***하겠다라는 정책이다. 그렇기 때문에 우리는 CORS 에러를 방지하기 위해서 CORS에 예외조항을 설정해줘야한다.
- 즉, 기본적으로 SOP정책이 설정되어있지만, ***CORS 설정***을 통해서 우리는 다른 서버에 리소스를 가지고 올 수 있다.

# 출처 비교는 브라우저가 한다.
- CORS 차단 유무는 출처라는 것을 비교해서 결정된다.
- 흔히 CORS 비교를 하는 주체는 서버라고 착각하는데 그렇지 않다. ***출처 비교와 차단의 메커니즘은 브라우저에 존재한다.*** (심지어 서버는 응답도 잘해준다)   
![555](https://user-images.githubusercontent.com/22884224/222697924-5d899f3f-d91c-4fae-b8b4-93cee58a860c.png)

# CORS 처리 과정
1. 기본적으로 웹은 HTTP 프로토콜을 사용해서 서버에 요청을 보낸다. 요청을 보낼때 Origin에 자신의 출처 값을 설정해서 요청한다.   
![333](https://user-images.githubusercontent.com/22884224/222697395-e98bfb52-b026-48ea-99cc-6b9979a272ff.png)
2. 서버는 클라이언트에 요청에 대한 응답을 할때 Access-Control-Allow-Origin 헤더에 출처를 담아 클라이언트에 응답한다. 이떄 CORS 설정이 되어있다면 요청의 Origin과 값이 같을 것이다.   
![444](https://user-images.githubusercontent.com/22884224/222697438-79d91f5d-5293-4196-bba7-41bec74e2c97.png)
3. 브라우저에서 Origin과 서버가 보내준 Access-Control-Allow-Origin을 비교한다.  
		3-1. ***만약 같다면 다른 출처의 리소스를 가지고 온다(성공)***     
		3-2. ***다르다면 온 응답을 사용하지 않고 버린다((=CORS에러)***     

# CORS의 3가지 요청 방식
### Preflight Request
- 예비 요청과 본 요청을 나누어서 서버로 전송하는 방식이다. 가장 많이 사용된다.
- 예비 요청에는 HTTP 메소드 중 OPTIONS 메소드가 사용되고 이는 본 요청을 보내기 전에 브라우저 스스로 이 요청을 보내는 것이 안전한지 확인하는 것이다. 
- CORS 에러는 예비요청 응답여부와는 상관이 없다. ***응답을 받고 나서 브라우저가 CORS정책위반인지를 판단하기 떄문이다.***
- 어떠한 경우에는 예비 요청없이 본 요청만 보내기도한다.
![111](https://user-images.githubusercontent.com/22884224/222696920-db09da73-f115-4fea-b591-9f29a338081c.png)

### SimpleRequest
- ***예비 요청을 보내지 않고 바로 본 요청을 보내는 걸 의미한다.*** Preflight와 전반적인 로직 자체는 같되, 예비 요청 존재 유무만 다르다.
- 하지만 아무때나 SimpleRequest를 쓸수는 없고 다음의 조건이 만족해야한다.
    - 요청의 메소드는 GET, HEAD, POST 중 하나여야 한다.
    - Accept, Accept-Language, Content-Language, Content-Type, DPR, Downlink, Save-Data, Viewport-Width, Width 헤더일 경우 에만 적용된다.
    - Content-Type 헤더가 application/x-www-form-urlencoded, multipart/form-data, text/plain중 하나여야한다. 아닐 경우 예비 요청으로 동작된다.
- 위처럼 SimpleRequest 조건이 까다롭기때문에 SimpleRequest이 일어나는 상황은 드물다. 대부분의 API 요청은 Preflight로 이루어진다.
![222](https://user-images.githubusercontent.com/22884224/222696925-64edacbf-2b53-44bd-9086-7ee1e5f7c69e.png)


### Credentialed Request
- 클라이언트에서 서버에게 ***인증 정보(Credential)을 실어 요청할때 사용된다.***
- 쿠키 혹은 Authorization 헤더에 설정하는 토큰 값 등을 말한다.
- 이는 좀 다른 형태로 통신하게 된다.
    - 요청쪽에서는 credentials 옵션을 설정해야한다.(same-origin 또는 include)
    - 요청에 credentials 옵션이 설정되면 브라우저는 CORS 정책 위반 여부를 검사하는 롤울 다음 두가지를 추가한다
	    - 응답 헤더에는 반드시 Access-Control-Allow-Credentials: true가 존재해야한다.
	    - Access-Control-Allow-Origin에는 *를 사용할 수 없으며, 명시적인 URL이어야한다.

# 결론
- (백엔드) ***결론은 서버에서 Access-Control-Allow-Origin 헤더에 허용할 출처를 기재해주면 된다. 즉, 백엔드 개발자가 작업해야한다.*** 
- (프론트) Webpack Dev Server로 리버스 프록싱하기? 
- 인증 정보같은 특별한 경우에는 응답 헤더에 Access-Control-Allow-Origin 이외의 것들이 추가 된다.

[출처] https://evan-moon.github.io/2020/05/21/about-cors/      
[출처] https://inpa.tistory.com/entry/WEB-%F0%9F%93%9A-CORS-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95-%F0%9F%91%8F     
[출처] https://www.youtube.com/watch?v=bW31xiNB8Nc&t=507s      
