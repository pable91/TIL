# 브라우저 캐시(HTTP 캐시)

### 브라우저 캐시
- 서버에 요청을 보내고 응답 데이터를 ***브라우저 캐시*** 라는 곳에 저장한다. 그러면 다음에 또 다시 요청을 보낼 때 ***브라우저 캐시안에 데이터가 존재하면 그 데이터를 바로 사용***하고, 만약 데이터가 존재하지 않는다면 서버로 요청을 보낼 것이다.
![cache1](https://user-images.githubusercontent.com/22884224/218252092-2a46f648-7e1a-4ae7-a206-f52fa5cf3cf6.png)
- 그런데 만약 서버에 데이터가 변경되면 ***브라우저측에서는 요청을 다시 보내야할지? 캐시에 있는 데이터를 재사용해야할지? 판단할 수가 없다.*** 그래서 ***만료시간(max-age)*** 라는 개념이 등장한다.
![cache2](https://user-images.githubusercontent.com/22884224/218252106-80e3d4eb-4c95-4dd1-8911-dbba8ee7d5e2.png)


### 만료시간(max-age)
- 브라우저 캐시의 ***만료시간***을 설정해주면 위와 같은 문제는 발생하지 않는다. 이 값은 ***cache-control 이라는 헤더에 설정되서 응답으로 보내진다.*** 설정한 시간이 지나면 다시 서버로 재요청을 하고(이를 검증한다고 표현하기도 한다), 만료되기 전에는 요청을 보내지 않고 캐시의 데이터를 사용한다.  
![cache3](https://user-images.githubusercontent.com/22884224/218252488-d1b58405-6fd1-4ba6-aa24-482ecfe36eca.png)
![cache4](https://user-images.githubusercontent.com/22884224/218252522-35b7b0bf-7a6a-4fa2-b2e6-9bd1889f4c61.png)
- 하지만 여기서도 문제가 있다. 브라우저 캐시 시간이 만료되면 서버로 요청을 보낼텐데, 서버의 데이터가 변경되서 응답이 새로 오면 괜찮지만 서버의 데이터가 변경되지않은경우엔 이미 브라우저 캐시에 담긴 데이터가 또 다시 응답값으로 온다. ***이러면 캐시에 있는 동일한 정보가 다운로드 되기때문에 불필요한 데이터 송수신이 발생할 것이다.***
![cache5](https://user-images.githubusercontent.com/22884224/218252672-ccfc0af6-b776-46eb-b35f-87217d56b025.png)

### E-tag
- 위의 문제는 ***E-tag*** 설정으로 해결할 수 있다. E-tag는 데이터의 고유 해시값이다.
- 처음에 서버로 요청을 보내고 클라이언트로 데이터를 응답받을 때 E-tag값도 같이 응답받는다. 
![cache6](https://user-images.githubusercontent.com/22884224/218253208-c63814b9-b4bd-4c81-9c7e-de127babe5f8.png)

- 캐시 만료시간이 지나면 서버로 재요청을 할때 E-tag 값도 같이 서버로 보내는데 넘어온 E-tag값과 서버에서 생성된 E-tag값을 비교해서, ***값이 같다면 동일한 데이터라는 의미로 데이터를 응답하지않고 값이 다르다면 다른 데이터이기 때문에 클라이언트로 데이터 응답을 수행 할 것이다.***
![cache7](https://user-images.githubusercontent.com/22884224/218253364-98b47c6d-eb2e-4e80-80b8-7a6fc011f9fc.png)
![cache8](https://user-images.githubusercontent.com/22884224/218253368-cf9d0e01-5df6-4ee5-b0f3-c3963ad67b30.png)

- 하지만 결국 이 방법도 맹점이 존재한다. 왜냐하면 결국엔 ***캐시 만료시간이 다 되어야지만 서로의 E-tag값을 확인 가능하기 때문이다.*** 만약에 데이터가 바뀌어야하는 상황에 캐시 만료시간이 엄청 길게 설정되어있다면 그 때까지 변경된 데이터는 응답을 받을 수 없다.

### 파일의 이름
- 위 문제는 ***파일 이름에 버전이나 날짜정보***를 기입해서 업데이트하면 해결 가능하다. 기본적으로 브라우저는 파일의 이름이 변경되면 데이터가 변경되었다고 판단해서 서버에서 수정된 정보를 다운로드 받기 때문이다.
![cache10](https://user-images.githubusercontent.com/22884224/218253961-2e8e0937-98b7-4d6c-bb3f-d4129a72feaa.png)
![cache11](https://user-images.githubusercontent.com/22884224/218253969-885f4eb0-7b34-4bd0-8560-d23b8ea0a933.png)

[출처] https://www.youtube.com/watch?v=NxFJ-mJdVNQ&t=604s
