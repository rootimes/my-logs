+++
title = 'Spring Sparrow Security'
date = 2026-03-17T18:48:09+08:00
draft = true
tags = [
    "spring",
]
description = "Spring Framework 入門介紹：Security 篇章，實作登入登出 API，包含 session 與 JWT 驗證，並且實作權杖輪換機制以提升安全性。"
+++

## 關注點

1. 實作登入登出 API，包含 session 與 JWT 驗證

## 使用 spring security

### 加入相關依賴

- **spring-security-web**： Web 安全功能，包含過濾器、攔截器與安全上下文管理等。
- **spring-security-config**：允許以程式化方式定義安全規則與策略。
- **spring-security-core**：提供核心安全功能，如認證、授權與密碼編碼等。
- **nimbus-jose-jwt**：提供 JWT 的生成與驗證功能。

```xml
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-web</artifactId>
            <version>7.0.3</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-config</artifactId>
            <version>7.0.3</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-core</artifactId>
            <version>7.0.3</version>
        </dependency>

        <dependency>
            <groupId>com.nimbusds</groupId>
            <artifactId>nimbus-jose-jwt</artifactId>
            <version>10.8</version>
        </dependency>
```

### 先瞭解發生什麼

![auth-flow](../../../images/spring-sparrow/sparrow-auth.png)

我們登入做複合驗證(Composite Authentication)模式，同時支援傳統 Web 的 Session 機制與雙權杖(Double Tokens: Access Token & Refresh Token)：

#### 登入驗證與權杖發放(Login & Issue Tokens)

1. **提交請求**：使用者透過 `POST /api/auth/login` 提交帳號密碼。
2. **身分驗證**：後端使用 `AuthService` 進行 BCrypt 密碼比對與資料庫查詢。
3. **核發權杖**：

    **Access Token(AT)**：一段短效期的 JWT，用於後續 API 請求授權。
    **Refresh Token(RT)**：一個隨機 UUID，儲存於資料庫中(效期通常為 7 天)，用於核發新的 Access Token。

4. **前端儲存**：前端收到 200 OK 回應後，需將 `accessToken` 與 `refreshToken` 儲存於客戶端，後端 Set-Cookie 將 JSESSIONID 傳回給瀏覽器。

#### 一般授權操作 (JWT Validation)

Spring security 的基礎設定外，還需實作一個 JWT 驗證

1. **攔截請求**：`JwtAuthenticationFilter` (繼承自 `OncePerRequestFilter`) 攔截帶有 `Bearer` 標頭的請求。
2. **解析驗證**：過濾器解析 JWT 的簽章與有效期限。
3. **上下文設定**：驗證成功後，透過 `SecurityContextHolder.getContext().setAuthentication()` 將使用者資訊(email、權限)存入 SecurityContext 中。
4. **抵達 Controller**：請求通過過濾器後，業務邏輯即可從 SecurityContext 獲取當前使用者資訊。

#### 權杖輪換機制 (Refresh Token Rotation)

為了提高安全性，系統實作了「權杖輪換」機制，防止 Refresh Token 被盜用：

1. **AT 過期**：當 Access Token (AT-1) 過期時，API 回傳 401 Unauthorized。
2. **要求刷新**：前端發送 `POST /api/auth/refresh` 並攜帶 `refreshToken` (RT-1)。
3. **安全性輪換**：
    後端驗證 RT-1 是否存在於資料庫且未過期。
    **關鍵步驟**：舊的 RT-1 會立即失效，系統產生全新的 RT-2 覆蓋資料庫中的舊值。
4. **更新權杖**：後端回傳新的 { AT-2, RT-2 }，前端必須同步更新本地儲存的權杖。

#### 登出流程

登出操作需確保伺服器與客戶端狀態同步清除：

1. **提交登出**：使用者點擊登出，發送 `POST /api/auth/logout`。
2. **徹底清除**：
    **Session**：執行 `session.invalidate()`。
    **資料庫**：將該使用者的 `refreshToken` 欄位設為 `NULL`，使所有舊權杖失效。
    **SecurityContext**：執行 `SecurityContextHolder.clearContext()`。
3. **前端清理**：前端收到成功回應後，必須主動刪除本地儲存的所有權杖資訊。

### 核心程式碼

SecurityConfig.java

**SecurityConfig** 是整個 Spring Security 的核心設定中樞。

在這個類別中，我們透過註冊不同的 Bean 來設定安全策略

- AuthenticationManager：負責處理認證請求，後續會在 **AuthService** 中被呼叫以驗證帳號密碼。

- UserDetailsService：自訂使用者資訊來源。這裡實作了 Lambda 表達式，從 **UserRepository** 透過 email 查詢使用者，並將其轉換為 Spring Security 內建的 **User** 物件，供框架進行密碼比對。

- SecurityFilterChain：設定 HTTP 請求的安全過濾規則。包含：

  - CORS 與 CSRF：啟用跨域資源共用 (CORS)。考量到我們同時保留了 Session 機制，CSRF 防護未完全關閉，而是針對特定的無狀態 API(如 /api/auth/login)忽略 CSRF 檢查。

  - Session 策略：設定為 **SessionCreationPolicy.ALWAYS** 以配合複合驗證模式。

  - 例外處理：覆寫了預設的 **AuthenticationEntryPoint**，確保未經授權的請求 (401) 會收到 JSON 格式的回應，
    而非預設的 HTML 登入頁 面，這對前端串接非常重要。

  - 路徑授權：設定哪些 API 需要驗證(如 /api/admin/**)，哪些可以直接放行(如：**/api/auth/login**)。

  - 過濾器順序：透過 **addFilterBefore** 將自訂的 JwtAuthenticationFilter 插入至原生的 **UsernamePasswordAuthenticationFilter** 之前，確保 JWT 驗證能優先執行。

- PasswordEncoder：指定使用 **BCryptPasswordEncoder** 作為密碼雜湊演算法。

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthenticationFilter;
    private final UserRepository userRepository;

    public SecurityConfig(JwtAuthenticationFilter jwtAuthenticationFilter, UserRepository userRepository) {
        this.jwtAuthenticationFilter = jwtAuthenticationFilter;
        this.userRepository = userRepository;
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }

    @Bean
    public UserDetailsService userDetailsService() {
        return email -> userRepository.findByEmail(email)
                .map(user -> new org.springframework.security.core.userdetails.User(
                        user.getEmail(),
                        user.getPassword(),
                        Collections.emptyList()))
                .orElseThrow(() -> new UsernameNotFoundException("User not found: " + email));
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

        CsrfTokenRequestAttributeHandler requestHandler = new CsrfTokenRequestAttributeHandler();

        requestHandler.setCsrfRequestAttributeName(null);

        http.csrf(csrf -> csrf.ignoringRequestMatchers("/api/auth/login", "/api/auth/refresh")
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                .csrfTokenRequestHandler(requestHandler))
                .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.ALWAYS))
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/api/auth/login", "/api/auth/refresh").permitAll()
                        .requestMatchers("/api/auth/logout").authenticated()
                        .requestMatchers("/api/status/**").permitAll()
                        .requestMatchers("/api/admin/**").authenticated()
                        .anyRequest().authenticated())
                .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

JwtAuthenticationFilter.java

此過濾器負責在每個 HTTP 請求抵達 Controller 之前，檢查並驗證 JWT。

繼承 **OncePerRequestFilter** 確保該過濾器在單次請求的生命週期中只會被執行一次。

- Session 優先檢查：為了支援複合驗證，**doFilterInternal** 第一步會先檢查 **SecurityContext** 是否已有已認證的 Session。若有，則直接放行，避免重複驗證。

- 提取與解析 Token：若無 Session，則從 HTTP 標頭 **Authorization** 提取以 **Bearer** 開頭的 Token。

- 建立授權上下文：將提取出的 Token 交由 **JwtUtil** 解析。若解析出合法的 email，代表 Token 有效，便建立一個 **UsernamePasswordAuthenticationToken** 物件，並將其存入 **SecurityContextHolder**。

這樣一來，後續的程式碼（包含 Controller）就能直接取得當前操作者的身分。

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private static final int BEARER_SPLIT_INDEX = 7;

    private final JwtUtil jwtUtil;

    public JwtAuthenticationFilter(JwtUtil jwtUtil) {
        this.jwtUtil = jwtUtil;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws ServletException, IOException {

        if (isSessionAuthenticated()) {
            filterChain.doFilter(request, response);
            return;
        }

        String token = resolveRequest(request);

        if (token == null) {
            filterChain.doFilter(request, response);
            return;
        }

        String email = jwtUtil.getValueFromToken(token, "email");
        if (email != null) {
            UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(
                    email, null, Collections.emptyList());
            authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }

        filterChain.doFilter(request, response);
    }

    private boolean isSessionAuthenticated() {
        return SecurityContextHolder.getContext().getAuthentication() != null &&
                SecurityContextHolder.getContext().getAuthentication().isAuthenticated();
    }

    private String resolveRequest(HttpServletRequest request) {
        String authHeader = request.getHeader("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            return null;
        }

        try {
            return authHeader.substring(BEARER_SPLIT_INDEX);
        } catch (Exception e) {
            throw new RuntimeException("Error extracting JWT from request", e);
        }
    }
}
```

JwtUtil.java

這個元件封裝了 **nimbus-jose-jwt** 套件的操作，專注於 JWT 的生成與驗證邏輯。

- 密鑰管理：透過 **Environment** 從 **application.properties** 讀取 **jwt.secret**。
為確保安全性，採用 HS256 演算法時，密鑰長度至少需要 32 個字元。

- 生成 Token (**generateToken**)：建立 **JWTClaimsSet**，設定 Subject (使用者信箱)、自訂 Claim (**email**)、發行者 (**issuer**) 與過期時間。最後使用 **MACSigner** 進行簽名並序列化為字串。

- 解析與驗證 Token (**getValueFromToken**)：使用 **MACVerifier** 驗證 Token 簽章是否遭到篡改。驗證通過後，進一步檢查 **ExpirationTime** 是否已過期。若一切合法，才回傳指定的 Claim 值。

```java
@Component
@PropertySource("classpath:application.properties")
public class JwtUtil {

    final static int ACCESS_TOKEN_EXPIRATION_MINUTES = 30;

    @Autowired
    private Environment env;

    private final long expirationTime = TimeUnit.MINUTES.toMillis(ACCESS_TOKEN_EXPIRATION_MINUTES);

    public String generateToken(String email) {
        try {
            JWSSigner signer = new MACSigner(getSecret().getBytes());

            JWTClaimsSet claimsSet = new JWTClaimsSet.Builder()
                    .subject(email)
                    .claim("email", email)
                    .issuer("sparrow")
                    .expirationTime(new Date(System.currentTimeMillis() + expirationTime))
                    .build();

            SignedJWT signedJWT = new SignedJWT(new JWSHeader(JWSAlgorithm.HS256), claimsSet);
            signedJWT.sign(signer);

            return signedJWT.serialize();
        } catch (JOSEException e) {
            throw new IllegalStateException("Error generating JWT", e);
        }
    }

    public String getValueFromToken(String token, String claimKey) {
        try {
            SignedJWT signedJWT = SignedJWT.parse(token);

            validateTokenSignature(signedJWT);

            JWTClaimsSet claimsSet = signedJWT.getJWTClaimsSet();
            Date expiration = claimsSet.getExpirationTime();

            validateTokenExpiration(expiration);

            return claimsSet.getStringClaim(claimKey);
        } catch (Exception e) {
            throw new BadCredentialsException("Error extracting claim value from JWT", e);
        }
    }

    private String getSecret() {
        String secret = env.getProperty("jwt.secret");
        if (secret == null || secret.length() < 32) {
            return "default-secret-at-least-32-chars-long!";
        }
        return secret;
    }

    private void validateTokenSignature(SignedJWT signedJWT) throws BadCredentialsException {
        try {
            JWSVerifier verifier = new MACVerifier(getSecret().getBytes());
            if (!signedJWT.verify(verifier)) {
                throw new BadCredentialsException("Invalid JWT signature");
            }
        } catch (JOSEException e) {
            throw new BadCredentialsException("Error validating JWT signature", e);
        }
    }

    private void validateTokenExpiration(Date expiration) throws BadCredentialsException {
        try {
            if (expiration != null && expiration.before(new Date())) {
                throw new BadCredentialsException("JWT token has expired");
            }
        } catch (Exception e) {
            throw new BadCredentialsException("Error validating JWT expiration", e);
        }
    }
}
```

AuthController.java

對外開放的 API 入口，負責接收前端請求並轉交業務層次 (AuthService) 處理：

- 登入 (/login)：接收帳號密碼，先觸發認證邏輯，成功後呼叫 Service 核發包含 Access Token 與 Refresh Token 的回應。

- 刷新權杖 (/refresh)：當 Access Token 過期時，前端會攜帶舊的 Refresh Token 呼叫此端點，換取一組全新的權杖。

- 登出 (/logout)：實作完整的清理機制。包含：

    1. 將原有的 HTTP Session 無效化 (session.invalidate())。

    2. 從 SecurityContext 中取得當前使用者，並清除資料庫中該使用者的 Refresh Token，確保舊權杖無法再被使用。

    3. 清空伺服器端的 SecurityContext。

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    private final AuthService authService;

    public AuthController(AuthService authService) {
        this.authService = authService;
    }

    @PostMapping("/login")
    public AuthResponse login(@RequestBody LoginRequest loginRequest) {
        authService.authenticate(loginRequest.getEmail(), loginRequest.getPassword());
        return authService.createAuthResponse(loginRequest.getEmail());
    }

    @PostMapping("/refresh")
    public AuthResponse refresh(@RequestBody AuthResponse refreshRequest) {
        return authService.refreshToken(refreshRequest.getRefreshToken());
    }

    @PostMapping("/logout")
    public AuthResponse logout(HttpServletRequest request) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            session.invalidate();
        }

        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        if (authentication != null && authentication.isAuthenticated()) {
            authService.revokeUserToken(authentication.getName());
        }

        SecurityContextHolder.clearContext();

        AuthResponse response = new AuthResponse();
        response.setMessage("Logout successful");
        return response;
    }
}
```

AuthService.java

處理實際的認證與權杖派發邏輯，並與資料庫進行互動，方法皆標註 @Transactional 以確保資料一致性：

- 密碼認證 (authenticate)：建構 UsernamePasswordAuthenticationToken 交由 AuthenticationManager 驗證。若密碼錯誤，此處會自動拋出例外。

- 核發權杖 (createAuthResponse)：登入成功後，生成新的 Access Token 以及由 UUID 構成的 Refresh Token，計算過期時間後寫入資料庫並回傳。

- 權杖輪換 (refreshToken)：此為雙權杖機制的核心。

    1. 驗證傳入的 Refresh Token 是否存在於資料庫。

    2. 檢查該 Refresh Token 是否已過期。若過期，強制清空資料庫欄位並要求重新登入。

    3. 若驗證通過，生成全新的 Access Token 與全新的 Refresh Token。

    4. 將新的 Refresh Token 覆蓋寫入資料庫（權杖輪換），這確保了每個 Refresh Token 只能被使用一次，大幅降低權杖被竊取的風險。

- 撤銷權杖 (revokeUserToken)：登出時呼叫，將該使用者的 Refresh Token 與過期時間設為 null。

```java
@Service
public class AuthService {

    @Autowired
    private Environment env;

    private final AuthenticationManager authenticationManager;
    private final JwtUtil jwtUtil;
    private final UserRepository userRepository;

    public AuthService(AuthenticationManager authenticationManager, JwtUtil jwtUtil, UserRepository userRepository) {
        this.authenticationManager = authenticationManager;
        this.jwtUtil = jwtUtil;
        this.userRepository = userRepository;
    }

    public void authenticate(String email, String password) {
        Authentication authenticationToken = new UsernamePasswordAuthenticationToken(email, password);
        Authentication authentication = authenticationManager.authenticate(authenticationToken);
        SecurityContextHolder.getContext().setAuthentication(authentication);
    }

    @Transactional
    public AuthResponse createAuthResponse(String email) {
        User user = userRepository.findByEmail(email)
                .orElseThrow(() -> new RuntimeException("User not found"));

        String accessToken = jwtUtil.generateToken(email);
        String refreshToken = UUID.randomUUID().toString();

        user.setRefreshToken(refreshToken);
        user.setRefreshTokenExpiration(
                LocalDateTime.now().plusDays(env.getProperty("jwt.refresh.token.expiration.days", Integer.class, 7)));

        AuthResponse response = new AuthResponse();
        response.setAccessToken(accessToken);
        response.setRefreshToken(refreshToken);
        response.setMessage("Login successful");
        return response;
    }

    @Transactional
    public AuthResponse refreshToken(String refreshToken) {
        User user = userRepository.findByRefreshToken(refreshToken)
                .orElseThrow(() -> new BadCredentialsException("Invalid refresh token"));

        if (user.getRefreshTokenExpiration().isBefore(LocalDateTime.now())) {
            user.setRefreshToken(null);
            user.setRefreshTokenExpiration(null);
            throw new BadCredentialsException("Refresh token expired");
        }

        String newAccessToken = jwtUtil.generateToken(user.getEmail());

        String newRefreshToken = UUID.randomUUID().toString();
        user.setRefreshToken(newRefreshToken);
        user.setRefreshTokenExpiration(
                LocalDateTime.now().plusDays(env.getProperty("jwt.refresh.token.expiration.days", Integer.class, 7)));

        AuthResponse response = new AuthResponse();
        response.setAccessToken(newAccessToken);
        response.setRefreshToken(newRefreshToken);
        response.setMessage("Token refreshed and rotated");
        return response;
    }

    @Transactional
    public void revokeUserToken(String email) {
        userRepository.findByEmail(email).ifPresent(user -> {
            user.setRefreshToken(null);
            user.setRefreshTokenExpiration(null);
        });
    }
}
```

## 參考資料

- [Spring Security 官方文件](https://docs.spring.io/spring-security/reference/)
- [Nimbus JOSE + JWT 官方文件](https://connect2id.com/products/nimbus-jose-jwt)
