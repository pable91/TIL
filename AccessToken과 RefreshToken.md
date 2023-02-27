# 현재상황
- 현재 monalia 프로젝트에서는 토큰(jwt) 인증,인가 방식을 사용하고 있다. 그리고 토큰은 AccessToken만 사용하고 있었다.
- 그런데 AccssToken의 만료시간을 설정하다가 "AccessToken의 만료시간을 길게 하면 편한거 아니야?" 라는 생각을 문득 하게되었고, 그러면 Trade-off가 분명 있을거라 생각해서 관련 내용을 구글링하다가 RefreshToken에 대해서 알게 되었다.

# AccessToken(=토큰방식)의 장점
- 인가의 정보를 클라이언트가 들고 있다. 서버가 인가에 대한 정보를 가지고 있을 필요가 없다.(= Stateless) 그렇기 때문에 서버가 확장되거나 서버에 문제가 생겼을 때 인가정보를 들고있는 사용자에게 이슈가 생길 확률이 줄어든다

# AccessToken만 사용했을때의 문제점
### AccessToken을 탈취당할 수 있다.
1. 탈취당할 시간을 줄이기 위해 AccessToken의 만료시간을 줄인다.
2. 하지만 만료시간이 짧기때문에 AccessToken을 자주 발급받아야한다.(= 로그인을 자주 해야한다)
3. 로그인은 가벼운 작업이 아니다 + 사용자에게 번거로운 작업을 요구해야한다.

# RefreshToken?
1. 최초의 로그인 때 AccessToken과 RefreshToken을 사용자에게 전달한다.(만료시간을 AccessToken은 비교적 짧게, RefreshToken은 비교적 길게)
2. AccessToken이 만료 되면 사용자는 RefreshToken을 서버에게 보내 Refresh 토큰 만료시간 검증 후 새로운 AccessToken을 전달받는다.(=로그인을 할 필요가 없다)
3. 만약 RefreshToken도 만료되면 그때는 로그인을 해야된다.

# RefreshToken의 장점
- AccessToken만 사용하고 만료시간이 짧을때, 사용자가 자주 로그인해야하는 문제를 해결해준다.

# RefreshTokendms 어떤 형식으로 저장해야할까?
- 토큰방식?
- UUID방식?

# RefreshToken의 한계점
- AccessToken의 만료시간을 짧게했다고 해서, AccessToken의 탈취가능성을 완전히 줄인것은 아니다.
- RefreshToken을 탈취당할 수 있다.

# 어떤 방식으로 탈취하게 되지? 그리고 탈취 자체를 불가능하게 할 수는 없을까?
- XSS 공격 
- CSRF 공격 

[출처] https://tecoble.techcourse.co.kr/post/2021-10-20-refresh-token/     
[출처] https://hudi.blog/refresh-token/    
[출처] https://hudi.blog/refresh-token-in-spring-boot-with-redis/    
[출처] https://velog.io/@yukina1418/JWT%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%A1%9C%EA%B7%B8%EC%9D%B8%EB%A1%9C%EA%B7%B8%EC%95%84%EC%9B%83    
[출처] https://www.youtube.com/watch?v=1QiOXWEbqYQ
