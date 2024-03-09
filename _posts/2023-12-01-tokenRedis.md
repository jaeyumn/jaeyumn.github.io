---
title: "[Spring Security] JWT + Redis 서버 도입"
categories:
  - develop
---

## 개발 환경

- Java 11
- SpringBoot 2.7.17
- jjwt 0.11.5
- redis

<br>

## Redis 도입 배경

우선 `Refresh Token`을 사용하는 이유에 대해 생각해야 합니다.

`Access Token`은 서버에서 보관하지 않기 때문에 `stateless`합니다. 즉, 서버는 발급한 토큰에 대한 제어권을 가지고 있지 않습니다. 따라서 누군가가 토큰을 탈취한다면 사용자 계정을 마음대로 사용할 수 있게 됩니다.

이러한 문제는 `Access Token`의 유효기간을 짧게 설정한다면 완벽하진 않더라도 어느 정도 보안이 되긴 합니다. 하지만 토큰의 만료 기간이 짧아질 수록 사용자는 새로운 토큰을 얻기 위해 다시 로그인해야 합니다. 보안적으로는 이점이 있지만 사용자에게 불편을 줄 수 있죠.

`Refresh Token`은 이러한 불편을 해결하기 위한 것입니다. `Access Token`보다 만료 기간을 길게 설정하여, `Refresh Token`이 유효하다면 사용자가 다시 로그인할 필요 없이 서버에서 새로운 `Access Token`을 발급하는 과정을 자동으로 처리할 수 있습니다.

그렇다면 이 `Refresh Token`은 어디에 저장될까요??

바로 **서버**에 저장되어야 합니다. 사용자가 보낸 `Access Token`이 유효하지 않다면 `Refresh Token`의 만료 기간을 확인하고 다시 새로운 토큰을 발급해줘야 하기 때문입니다.

이 `Refresh Token`을 저장하는 저장소를 `Redis`로 선택한 이유는 다음과 같습니다.

1. `Redis`는 `In-Memory`방식의 데이터베이스입니다. 따라서 `MySQL`이나 `Oracle`같이 보조기억장치가 아닌 RAM에 데이터를 저장합니다. 데이터를 보조기억장치가 아닌 RAM에서 바로 불러오기 때문에 속도 측면에서 이점을 갖습니다.
2. `Key-Value` 형태의 데이터베이스이기 때문에 데이터를 저장하고 조회하는 비용이 적습니다.
3. `In-Memory` 데이터베이스는 휘발성이므로 `Refresh Token`을 관리하기에 적합합니다.
4. `cache`처럼 데이터 만료 시간을 설정할 수 있습니다. 따라서 기간이 만료된 토큰이 불필요하게 남아있지 않도록 할 수 있습니다.

<br>

## Redis 적용

### Redis 저장소

```java
@RequiredArgsConstructor
@Component
public class RefreshTokenRepository {

    private final RedisRepository redisRepository;

    public void add(String refreshToken, String memberId, Instant expiry) {
        redisRepository.set(refreshToken, memberId, Duration.between(Instant.now(), expiry));
    }

    public Optional<String> findMemberIdByRtk(String rtk) {
        String memberId = redisRepository.get(rtk);
        if (memberId == null) {
            return Optional.empty();
        }
        return Optional.of(memberId);
    }

    public void prohibitAccessToken(String accessToken, String refreshToken, String message, Instant expiry) {
        redisRepository.remove(refreshToken);
        redisRepository.set(accessToken, message, Duration.between(Instant.now(), expiry));
    }
}

```

- add() : 발급된 리프레시 토큰과 사용자 식별값, 만료 시간을 저장합니다.
- findMemberIdByRtk() : 리프레시 토큰 값으로 사용자 식별값을 조회합니다. 액세스 토큰이 만료되었을 때, 새로 발급하기 위한 용도로 사용됩니다.
- prohibitAccessToken() : 리프레시 토큰을 저장소에서 제거하고 액세스 토큰의 정보와 만료 기간을 새로 저장합니다. 로그아웃 시 액세스 토큰이 유효하더라도 사용할 수 없도록 설정하여 보안성을 높였습니다.

<br>

### AccessToken 재발급

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

재발급 순서는 다음과 같습니다.

1. 인자로 전달받은 리프레시 토큰이 유효한지 확인합니다.
2. 리프레시 토큰 값으로 `memberId`를 조회합니다.
3. 조회한 `memberId`를 가지고 액세스 토큰을 발급합니다. 여기서 `memberId`는 토큰의 `subject` 값으로 지정했습니다.
4. 리프레시 토큰도 재발급하여 발급된 두 토큰 값을 반환합니다.

<br>

### 로그아웃 API 호출 시

```java
@Override
    public void prohibitTokens(String accessToken, String refreshToken) {
        String BEARER_TYPE = "Bearer ";
        String accessTokenValue = accessToken.substring(BEARER_TYPE.length());
        Instant expirationOfAccessToken = parse(accessTokenValue).getExpiredAt();

        refreshTokenRepository.prohibitAccessToken(accessTokenValue, refreshToken, "logout", expirationOfAccessToken);
    }
```

- 저장소에서 인자로 받은 리프레시 토큰을 제거합니다.
- 인자로 받은 액세스 토큰의 만료 기간과 동일한 데이터를 `Redis`에 저장합니다. 여기서 저장된 데이터는 사용자가 요청을 보냈을 때 만약 동일한 액세스 토큰이 `Redis`에 저장되어 있다면 요청을 받지 않습니다. 일종의 블랙리스트라고 생각하면 이해가 쉬울 것 같습니다.
