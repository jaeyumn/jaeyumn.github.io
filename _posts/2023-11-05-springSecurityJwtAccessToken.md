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

<br>

### JWT 생성 흐름
---

1. 클라이언트가 서버 측에 로그인 정보(ex: `username`, `password`)로 인증 요청을 보낸다.
2. 클라이언트로부터 전송된 로그인 정보를 `Security Filter(JwtAuthenticationFilter)`가 받아서 처리한다.
3. `Security Filter`는 수신한 로그인 인증 정보를 `AuthenticationManager`에게 전달해 인증 처리를 위임한다.
4. `AuthenticationManager`는 `UserDetailsService`에게 사용자의 `UserDetails` 조회를 위임하고, DB에서 조회된 사용자 정보를 반환받는다.
5. `AuthenticationManager`는 전달받은 로그인 정보와 `UserDetials` 정보를 비교하여 인증을 수행한다.
6. 인증이 성공하면 `JWT(토큰)`가 생성되고 이 토큰은 클라이언트에게 전달된다.
7. 클라이언트는 이 토큰을 이용해 이후의 요청에서 자신을 인증할 수 있다.

<br>

### 구현
---

### SecurityConfig

```java
@RequiredArgsConstructor
@EnableWebSecurity
@Configuration
public class SecurityConfig {

    private final JwtAuthenticationProvider jwtAuthenticationProvider;

    @Value("${cors.allowed-origins}")
    private List<String> allowedOrigins;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        return http
                .headers().frameOptions().disable()
                .and()
                .httpBasic().disable()
                .formLogin().disable()
                .csrf().disable()
                .cors(withDefaults())
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
//                .antMatchers("/api/admin/**").hasRole(Role.ADMIN.name())
                .antMatchers("/api/members/**").hasRole(Role.MEMBER.name())
                .antMatchers("/api/images/**").hasRole(Role.MEMBER.name())
                .anyRequest().permitAll()
                .and()
                .addFilterBefore(new JwtAuthenticationFilter(jwtAuthenticationProvider), UsernamePasswordAuthenticationFilter.class)
                .build();
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(allowedOrigins);
        configuration.setAllowedMethods(List.of("GET", "POST", "PATCH", "PUT", "DELETE", "OPTION"));
        configuration.setAllowedHeaders(List.of("*"));
        configuration.setExposedHeaders(List.of("*"));

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);

        return source;
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

- `HTTP` 기본 인증 및 폼 기반 로그인을 비활성화
- 세션 관리를 `STATELESS`로 설정하여 세션을 사용 비활성화
- 초기 설정이기 때문에 모든 `CORS` 설정을 열어둠

<br>

### Security Filter

```java
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtAuthenticationProvider jwtAuthenticationProvider;

    private static final String BEARER_TYPE = "Bearer ";

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String authorizationHeader = request.getHeader(HttpHeaders.AUTHORIZATION);
        String accessToken = extractAccessToken(authorizationHeader);

        if (accessToken != null) {
            Authentication authentication = jwtAuthenticationProvider.authenticateAccessToken(accessToken);
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }

        filterChain.doFilter(request, response);
    }

    private String extractAccessToken(String authorizationHeader) {
        if (StringUtils.hasText(authorizationHeader) && authorizationHeader.startsWith(BEARER_TYPE)) {
            return authorizationHeader.substring(BEARER_TYPE.length());
        }
        return null;
    }
}

```

- `header`에서 `AccessToken`을 추출하고 `jwtAuthenticationProvider`에게 전달
- `jwtAuthenticationProvider`를 통해 권한 정보를 받음

<br>

### JwtAuthenticationProvider

```java
@Slf4j
@RequiredArgsConstructor
@Service
public class JwtAuthenticationProvider {

    private final JwtService jwtService;

    protected Authentication authenticateAccessToken(String accessToken) {
        if (!jwtService.validate(accessToken)) {
            log.info("is invalid token");
            return null;
        }

        Payload payload = jwtService.parse(accessToken);
        String id = payload.getId();
        String role = payload.getRole();

        return new UsernamePasswordAuthenticationToken(id, null, Collections.singleton(new SimpleGrantedAuthority(role)));
    }
}

```

- 전달 받은 `AccessToken`을 검증하고 파싱한 후, 권한 정보를 담아서 반환

<br>

### JwtService

```java
@Slf4j
@Service
public class JwtServiceImpl implements JwtService {

    @Value("${jwt.secret-key}")
    private String secretKey;

    @Value("${jwt.access-token.expiration}")
    private Long accessTokenExpiration;

    @Override
    public boolean validate(String token) {
        try {
            Jwts.parserBuilder()
                    .setSigningKey(Keys.hmacShaKeyFor(secretKey.getBytes(StandardCharsets.UTF_8)))
                    .build()
                    .parseClaimsJws(token)
                    .getBody();
        } catch (JwtException e) {
            log.info("{}", e.getMessage());
            return false;
        }

        return true;
    }

    @Override
    public String createMemberAccessToken(String memberId) {
        return buildJwt(Role.MEMBER, memberId);
    }

    @Override
    public String createAdminAccessToken(String adminId) {
        return buildJwt(Role.ADMIN, adminId);
    }

    @Override
    public Payload parse(String token) {
        Claims claims = Jwts.parserBuilder()
                .setSigningKey(Keys.hmacShaKeyFor(secretKey.getBytes(StandardCharsets.UTF_8)))
                .build()
                .parseClaimsJws(token)
                .getBody();

        return new Payload(claims.getSubject(), claims.get("role", String.class));
    }

    private String buildJwt(Role role, String id) {
        Instant now = Instant.now();

        return Jwts.builder()
                .signWith(Keys.hmacShaKeyFor(secretKey.getBytes(StandardCharsets.UTF_8)), SignatureAlgorithm.HS256)
                .claim("role", role.getValue())
                .setSubject(id)
                .setIssuedAt(Date.from(now))
                .setExpiration(Date.from(now.plusSeconds(accessTokenExpiration)))
                .compact();
    }
}
```

- Secret Key와 만료 시간 설정 값은 `application.yml` 파일에서 가져옴

<br>

### 기타 구성
---

- PasswordEncoder

```java
@RequiredArgsConstructor
@Service
public class PasswordEncoderImpl implements PasswordEncoder {

    private final BCryptPasswordEncoder passwordEncoder;

    @Override
    public boolean match(String targetPassword, String encodedPassword) {
        return passwordEncoder.matches(targetPassword, encodedPassword);
    }

    @Override
    public String encode(String password) {
        return passwordEncoder.encode(password);
    }
}
```

<br>

- Service

```java
@RequiredArgsConstructor
@Transactional
@Service
public class MemberService {

    private final MemberRepository memberRepository;
    private final PasswordEncoder passwordEncoder;
    private final TokenService tokenService;

    public void signUp(SignUpRequestDto signUpRequestDto) {
        if (memberRepository.findByEmail(signUpRequestDto.getEmail()).isPresent()) {
            throw new IllegalArgumentException("exists member: " + signUpRequestDto.getEmail());
        }

        Member member = signUpRequestDto.toEntity(passwordEncoder);
        memberRepository.save(member);
    }

    public String signIn(SignInRequestDto signInRequestDto) {
        String requestEmail = signInRequestDto.getEmail();
        Member findMember = memberRepository.findByEmail(requestEmail)
                .orElseThrow(() -> new BadCredentialsException("not found email: " + requestEmail));

        findMember.checkPassword(signInRequestDto.getPassword(), passwordEncoder);

        return tokenService.createMemberAccessToken(findMember.getId().toString());
    }
}
```

<br>

- Controller

```java
@RequiredArgsConstructor
@RequestMapping("/api")
@RestController
public class MemberController {

    private final MemberService memberService;

    private static final String BEARER_TYPE = "Bearer ";

    @PostMapping("/sign-up")
    public ResponseEntity<Void> signUp(@RequestBody @Valid SignUpRequestDto signUpRequestDto) {
        memberService.signUp(signUpRequestDto);

        return ResponseEntity.status(HttpStatus.CREATED)
                .build();
    }

    @PostMapping("/sign-in")
    public ResponseEntity<Void> signIn(@RequestBody @Valid SignInRequestDto signInRequestDto) {
        String token = memberService.signIn(signInRequestDto);

        return ResponseEntity.status(HttpStatus.OK)
                .headers(httpHeaders -> httpHeaders.add(HttpHeaders.AUTHORIZATION, BEARER_TYPE + token))
                .build();
    }
}

```