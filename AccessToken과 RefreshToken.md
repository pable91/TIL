# 현재상황
- 현재 monalia 프로젝트에서는 토큰(jwt) 인증,인가 방식을 사용하고 있다. 그리고 토큰은 AccessToken만 사용하고 있었다.
- 그런데 AccssToken의 만료시간을 설정하다가 "AccessToken의 만료시간을 길게 하면 편한거 아니야?" 라는 생각을 문득 하게되었고, 그러면 Trade-off가 분명 있을거라 생각해서 관련 내용을 구글링하다가 RefreshToken에 대해서 알게 되었다.

# AccessToken(=토큰방식)의 장점
- 인가의 정보를 클라이언트가 들고 있다. 서버가 인가에 대한 정보를 가지고 있을 필요가 없다.(= Stateless) 그렇기 때문에 서버가 확장되거나 서버에 문제가 생겼을 때 인가정보를 들고있는 사용자에게 이슈가 생길 확률이 줄어든다

# AccessToken만 사용했을때의 문제점
### AccessToken을 탈취당할 수 있다.
1. 탈취당할 시간을 줄이기 위해 AccessToken의 만료시간을 줄인다.
2. 하지만 ***만료시간이 짧기때문에 AccessToken을 자주 발급받아야한다.(= 로그인을 자주 해야한다)***
3. 로그인은 가벼운 작업이 아니다 + 사용자에게 번거로운 작업을 요구해야한다.

# RefreshToken?
1. 최초 로그인 때 ***AccessToken과 RefreshToken을 사용자에게 전달한다.(만료시간을 AccessToken은 비교적 짧게, RefreshToken은 비교적 길게)***
2. ***AccessToken이 만료 되면 사용자는 RefreshToken을 서버에게 보내 Refresh 토큰 만료시간 검증 후 갱신한 새로운 AccessToken을 전달받는다.(=로그인을 할 필요가 없다)***
3. 만약 RefreshToken도 만료되면 그때는 로그인을 해야된다.
![refresh-token](https://user-images.githubusercontent.com/22884224/221601387-95b523a2-713c-4418-b811-62a2a12df7f4.png)


# RefreshToken의 장점
- 짧은 만료시간을 가진 AccessToken이 만료 되었을 때, 사용자가 자주 로그인해야하는 문제를 해결해준다.

# RefreshTokendms 어떤 형식으로 저장해야할까?
- 토큰방식
  - jwt 토큰방식이면 토큰 자체에 데이터를 담기때문에 AccessToken 처럼 stateless 방식이다. 이렇게 하면 DB같은곳에 데이터를 관리하지않아도 되지만 AccessToken처럼 탈취당했을때 서버에서 제어할 수 없는 단점이 있다.
- UUID방식?
  - Random String 또는 UUID 같은걸로 Refresh token을 사용하면 데이터를 DB에 저장해야한다. 그렇기 때문에 액세스 비용과 자원이 소요되겠지만 탈취당했을때 서버에서 제어할 수 있다는 장점이 있다. 
    - DB에 Refresh Token을 저장하면 발급 후 만료시간이 지났을때 직접 제거해줘야한다. 하지만 Redis 같은 인메모리 DB를 사용하면 유효시간을 지정할 수 있다는 이점이 있고, 만약 메모리에서 실수로 Refresh Token이 제거되어도 크게 치명적이지 않기때문에 Redis가 괜찮은 방법 일 수 있다.  

# RefreshToken의 한계점
- AccessToken의 만료시간을 짧게했다고 해서, AccessToken의 탈취가능성을 완전히 없앤것은 아니다.
- RefreshToken을 탈취당할 수 있다.

# 탈취할 가능성을 줄이는 여러 방법들
- RTR(Refresh Token Rotation) : Refresh Token을 가지고 갱신을 요청할때마다 새로운 AccessToken과 Refresh Token을 발급한다. 그러면 전에 사용되었던 Refresh Token은 탈취당해도 사용 불가능하다. 그렇지만 Refresh Token을 탈취당하고 다시 재갱신하지 않으면 탈취당한 Refresh 은 유효하다는 맹점이 있다.
- 최초로 접근해서 Refresh Token을 발급받은 IP를 서버에 저장한다. 다음에 이 IP에서 갱신요청이 온것이 아니라면 Token들을 모두 삭제하고 로그아웃 시키는 방법도 있다. 하지만 이방법은 사용자가 여러 IP를 사용한다고 하면, 로그아웃이 자주 일어날 수 있다는 문제가 있다.

# 적용 코드
- Redis를 사용해서 RefreshToken을 관리한다
```
@RedisHash(value = "refreshToken", timeToLive = 60)
@Getter
public class RefreshToken {

    @Id
    private String refreshToken;
    private String accountId;

    public RefreshToken(final String refreshToken, final String accountId) {
        this.refreshToken = refreshToken;
        this.accountId = accountId;
    }
}
```

- Access Token이 만료되어 refreshToken이 서버로 오면, 해당 refresh Token이 (유효한 토큰 and 만료 시간이 지나지않았으면) 새로운 Access Token을 만들어준다.
```
public String recreateAccessToken(final String refreshToken, final User user) {
        Optional<RefreshToken> resultToken = refreshTokenRepository.findById(refreshToken);

        if (resultToken.isPresent() && isMyToken(user, resultToken)) {
            return createAccessToken(user);
        }
        throw new InvalidRefreshTokenException(UserErrorCode.INVALID_REFRESH_TOKEN, UserErrorCode.INVALID_REFRESH_TOKEN.getMessage());
    }
```



# (보너스)어떤 방식으로 탈취하게 되지?
- 새롭게 알게 된 단어들이다. 웹 공격에 대한 용어들이니 간략하게 이해하고 넘어가면 좋을듯.
- XSS 공격 : 공격자가 브라우저에서 악성 스크립트를 삽입하면, 상대방의 브라우저에서 강제로 악성 스크립트가 실행되게 하는 방식(공격대상->클라이언트)
  - ex) 게시판에 악성 스크립트가 포함된 글을 올려서 사용자들이 클릭하면 스크립트가 실행된다.
- CSRF 공격 : 공격자가 스크립트를 수정해서 그 스크립트를 사용자가 실행하면, 서버는 신뢰할만한 사용자가 요청을 보낸것이라 생각하고 악성 스크립트에 대한 응답을 내려준다.(공격대상 -> 서버) 

[출처] https://tecoble.techcourse.co.kr/post/2021-10-20-refresh-token/     
[출처] https://hudi.blog/refresh-token/    
[출처] https://hudi.blog/refresh-token-in-spring-boot-with-redis/    
[출처] https://velog.io/@yukina1418/JWT%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%A1%9C%EA%B7%B8%EC%9D%B8%EB%A1%9C%EA%B7%B8%EC%95%84%EC%9B%83    
[출처] https://www.youtube.com/watch?v=1QiOXWEbqYQ   
[출처] https://blackrose.tistory.com/77
