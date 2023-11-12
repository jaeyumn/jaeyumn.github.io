---
title: "[Spring Security] CSR server에 JWT 도입하기"
categories:
    - spring
---

### 개발 환경
---

- Java 11
- SpringBoot 2.7.17
- jjwt 0.11.5
- redis

<br>

### 기본적인 JWT 생성 흐름
---

1. 클라이언트가 서버 측에 로그인 정보(ex: `username`, `password`)로 인증 요청을 보낸다.
2. 클라이언트로부터 전송된 로그인 정보를 `Security Filter(JwtAuthenticationFilter)`가 받아서 처리한다.
3. `Security Filter`는 수신한 로그인 인증 정보를 `AuthenticationManager`에게 전달해 인증 처리를 위임한다.
4. `AuthenticationManager`는 `UserDetailsService`에게 사용자의 `UserDetails` 조회를 위임하고, DB에서 조회된 사용자 정보를 반환받는다.
5. `AuthenticationManager`는 전달받은 로그인 정보와 `UserDetials` 정보를 비교하여 인증을 수행한다.
6. 인증이 성공하면 `JWT(토큰)`가 생성되고 이 토큰은 클라이언트에게 전달된다.
7. 클라이언트는 이 토큰을 이용해 이후의 요청에서 자신을 인증할 수 있다.

<br>

### AccessToken 및 RefreshToken 발급 과정
---

```java
private Token buildAccessToken(Role role, String id) {
    Instant now = Instant.now();
    Instant expiration = now.plusSeconds(jwtProperties.getAccessTokenExpSec());
    String tokenValue = Jwts.builder()
            .signWith(Keys.hmacShaKeyFor(jwtProperties.getSecretKey().getBytes(StandardCharsets.UTF_8)), SignatureAlgorithm.HS256)
            .claim("role", role.getValue())
            .setSubject(id)
            .setIssuedAt(Date.from(now))
            .setExpiration(Date.from(expiration))
            .compact();

    return new Token(tokenValue, now, expiration);
}

private Token buildRefreshToken() {
    Instant now = Instant.now();
    Instant expiration = now.plusSeconds(jwtProperties.getRefreshTokenExpSec());
    String tokenValue = Jwts.builder()
            .signWith(Keys.hmacShaKeyFor(jwtProperties.getSecretKey().getBytes(StandardCharsets.UTF_8)), SignatureAlgorithm.HS256)
            .setIssuedAt(Date.from(now))
            .setExpiration(Date.from(expiration))
            .compact();

    return new Token(tokenValue, now, expiration);
}
```

- 로그인 요청이 정상적으로 수행되면, 토큰 발급을 진행한다.
- `AccessToken`과 `RefreshToken`을 모두 발급한다.
- `AccessToken`을 발급할 때, `role` 정보를 같이 전달해 준다. 현재 설정되어 있는 `role`은 일반 사용자와 관리자로 구분되어 있다.

<br>

```java
ResponseCookie rtkCookie = ResponseCookie.from("rtk", tokens.getRefreshToken().getValue())
            .path("/")
            .httpOnly(true)
            .maxAge(Duration.between(Instant.now(), tokens.getRefreshToken().getExpiredAt()))
            .build();
```

- 토큰이 정상적으로 발급되면, `RefreshToken` 정보를 `Cookie`에 담아서 응답 헤더로 전달한다.

<br>

### AccessToken 재발급 과정
---

```java
@PostMapping("/reissue")
public ResponseEntity<CommonResponse<Void>> reissueToken(HttpServletRequest request) {
    Cookie[] cookies = request.getCookies();
    if (cookies == null) throw new UnauthenticatedMemberException();
    String refreshTokenValue = Arrays.stream(cookies)
            .filter(cookie -> cookie.getName().equals("rtk"))
            .findFirst()
            .map(Cookie::getValue)
            .orElseThrow(UnauthenticatedMemberException::new);

    Tokens tokens = authService.reissue(refreshTokenValue);
    String rtkCookie = tokens.getRefreshToken() != null ? ResponseCookie.from("rtk", tokens.getRefreshToken().getValue())
            .path("/")
            .httpOnly(true)
            .maxAge(Duration.between(Instant.now(), tokens.getRefreshToken().getExpiredAt()).getSeconds())
            .build()
            .toString()
            : null;

    log.info("토큰 재발급 완료");

    return ResponseEntity.status(HttpStatus.OK)
            .headers(httpHeaders -> {
                httpHeaders.add(
                        HttpHeaders.AUTHORIZATION,
                        BEARER_TYPE + tokens.getAccessToken().getValue()
                );
                if (rtkCookie != null) {
                    httpHeaders.add(
                            HttpHeaders.SET_COOKIE,
                            rtkCookie
                    );
                }
            })
            .body(CommonResponse.createSuccess(null));
}
```

- 요청에 `Cookie`가 있는지 확인하고 없으면 인증되지 않은 사용자 예외를 던진다.
- `AccessToken`과 `RefreshToken`을 재발급하고 응답 헤더에 담아서 전달한다.
- `authService.reissue()` 로직은 다음과 같다.

<br>

```java
public Tokens reissue(String refreshToken) {
    return jwtManager.reissueTokens(refreshToken);
}
```

```java
@Override
public Tokens reissueTokens(String refreshTokenValue) {
    if (!validate(refreshTokenValue)) {
        throw new InvalidRefreshTokenException();
    }

    String memberId = refreshTokenRepository.findMemberIdByRtk(refreshTokenValue)
            .orElseThrow(UnauthenticatedMemberException::new);

    Token accessToken = createMemberAccessToken(memberId);
    Payload rtkPayload = parse(refreshTokenValue);
    long remainSec = Duration.between(Instant.now(), rtkPayload.getExpiredAt()).toSeconds();
    Token refreshToken = null;
    if (remainSec <= jwtProperties.getRefreshTokenReissueCondSec()) {
        refreshToken = createMemberRefreshToken(memberId);
    }

    return new Tokens(accessToken, refreshToken);
}
```

1. `refreshTokenValue`에 대한 유효성을 검증한다.
2. `refreshTokenRepository`에서 회원 식별값을 받아온 후, `AccessToken`을 발급한다.
3. `RefreshToken`의 `Payload`를 파싱한 후, 남은 유효 기간이 설정된 유효 기간보다 짧다면 `RefreshToken`을 새로 발급해 준다.
4. 새로 발급된 `AccessToken`과 `RefreshToken`을 반환한다.