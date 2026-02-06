---
name: springboot-patterns
description: Spring Bootアーキテクチャパターン、REST API設計、レイヤードサービス、データアクセス、キャッシング、非同期処理、ログ出力。Java Spring Bootバックエンド作業用。
---

# Spring Boot開発パターン

スケーラブルで本番グレードのサービスのためのSpring BootアーキテクチャとAPIパターン。

## REST API構造

```java
@RestController
@RequestMapping("/api/markets")
@Validated
class MarketController {
  private final MarketService marketService;

  MarketController(MarketService marketService) {
    this.marketService = marketService;
  }

  @GetMapping
  ResponseEntity<Page<MarketResponse>> list(
      @RequestParam(defaultValue = "0") int page,
      @RequestParam(defaultValue = "20") int size) {
    Page<Market> markets = marketService.list(PageRequest.of(page, size));
    return ResponseEntity.ok(markets.map(MarketResponse::from));
  }

  @PostMapping
  ResponseEntity<MarketResponse> create(@Valid @RequestBody CreateMarketRequest request) {
    Market market = marketService.create(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(MarketResponse.from(market));
  }
}
```

## リポジトリパターン (Spring Data JPA)

```java
public interface MarketRepository extends JpaRepository<MarketEntity, Long> {
  @Query("select m from MarketEntity m where m.status = :status order by m.volume desc")
  List<MarketEntity> findActive(@Param("status") MarketStatus status, Pageable pageable);
}
```

## トランザクション付きサービスレイヤー

```java
@Service
public class MarketService {
  private final MarketRepository repo;

  public MarketService(MarketRepository repo) {
    this.repo = repo;
  }

  @Transactional
  public Market create(CreateMarketRequest request) {
    MarketEntity entity = MarketEntity.from(request);
    MarketEntity saved = repo.save(entity);
    return Market.from(saved);
  }
}
```

## DTOと検証

```java
public record CreateMarketRequest(
    @NotBlank @Size(max = 200) String name,
    @NotBlank @Size(max = 2000) String description,
    @NotNull @FutureOrPresent Instant endDate,
    @NotEmpty List<@NotBlank String> categories) {}

public record MarketResponse(Long id, String name, MarketStatus status) {
  static MarketResponse from(Market market) {
    return new MarketResponse(market.id(), market.name(), market.status());
  }
}
```

## 例外処理

```java
@ControllerAdvice
class GlobalExceptionHandler {
  @ExceptionHandler(MethodArgumentNotValidException.class)
  ResponseEntity<ApiError> handleValidation(MethodArgumentNotValidException ex) {
    String message = ex.getBindingResult().getFieldErrors().stream()
        .map(e -> e.getField() + ": " + e.getDefaultMessage())
        .collect(Collectors.joining(", "));
    return ResponseEntity.badRequest().body(ApiError.validation(message));
  }

  @ExceptionHandler(AccessDeniedException.class)
  ResponseEntity<ApiError> handleAccessDenied() {
    return ResponseEntity.status(HttpStatus.FORBIDDEN).body(ApiError.of("Forbidden"));
  }

  @ExceptionHandler(Exception.class)
  ResponseEntity<ApiError> handleGeneric(Exception ex) {
    // 予期しないエラーをスタックトレース付きでログ
    return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
        .body(ApiError.of("Internal server error"));
  }
}
```

## キャッシング

設定クラスで`@EnableCaching`が必須です。

```java
@Service
public class MarketCacheService {
  private final MarketRepository repo;

  public MarketCacheService(MarketRepository repo) {
    this.repo = repo;
  }

  @Cacheable(value = "market", key = "#id")
  public Market getById(Long id) {
    return repo.findById(id)
        .map(Market::from)
        .orElseThrow(() -> new EntityNotFoundException("Market not found"));
  }

  @CacheEvict(value = "market", key = "#id")
  public void evict(Long id) {}
}
```

## 非同期処理

設定クラスで`@EnableAsync`が必須です。

```java
@Service
public class NotificationService {
  @Async
  public CompletableFuture<Void> sendAsync(Notification notification) {
    // メール/SMS送信
    return CompletableFuture.completedFuture(null);
  }
}
```

## ログ出力 (SLF4J)

```java
@Service
public class ReportService {
  private static final Logger log = LoggerFactory.getLogger(ReportService.class);

  public Report generate(Long marketId) {
    log.info("generate_report marketId={}", marketId);
    try {
      // ロジック
    } catch (Exception ex) {
      log.error("generate_report_failed marketId={}", marketId, ex);
      throw ex;
    }
    return new Report();
  }
}
```

## ミドルウェア / フィルタ

```java
@Component
public class RequestLoggingFilter extends OncePerRequestFilter {
  private static final Logger log = LoggerFactory.getLogger(RequestLoggingFilter.class);

  @Override
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
      FilterChain filterChain) throws ServletException, IOException {
    long start = System.currentTimeMillis();
    try {
      filterChain.doFilter(request, response);
    } finally {
      long duration = System.currentTimeMillis() - start;
      log.info("req method={} uri={} status={} durationMs={}",
          request.getMethod(), request.getRequestURI(), response.getStatus(), duration);
    }
  }
}
```

## ページネーションとソート

```java
PageRequest page = PageRequest.of(pageNumber, pageSize, Sort.by("createdAt").descending());
Page<Market> results = marketService.list(page);
```

## エラーレジリエント外部呼び出し

```java
public <T> T withRetry(Supplier<T> supplier, int maxRetries) {
  int attempts = 0;
  while (true) {
    try {
      return supplier.get();
    } catch (Exception ex) {
      attempts++;
      if (attempts >= maxRetries) {
        throw ex;
      }
      try {
        Thread.sleep((long) Math.pow(2, attempts) * 100L);
      } catch (InterruptedException ie) {
        Thread.currentThread().interrupt();
        throw ex;
      }
    }
  }
}
```

## レート制限 (フィルタ + Bucket4j)

**セキュリティ注記**: `X-Forwarded-For`ヘッダはデフォルトでは信頼されません。これはクライアントがスプーフィングできるためです。転送ヘッダーは以下の場合にのみ使用してください：

1. アプリが信頼されたリバースプロキシ（nginx、AWS ALBなど）の背後にある
2. `ForwardedHeaderFilter`がビーンとして登録されている
3. `server.forward-headers-strategy=NATIVE`または`FRAMEWORK`が`application`プロパティで設定されている
4. プロキシが`X-Forwarded-For`ヘッダーを上書き（追加ではなく）するように設定されている

`ForwardedHeaderFilter`が適切に設定されている場合、`request.getRemoteAddr()`は自動的に転送ヘッダーから正しいクライアントIPを返します。この設定なしでは、`request.getRemoteAddr()`を直接使用してください。これは唯一の信頼できる値であるダイレクト接続IPを返します。

```java
@Component
public class RateLimitFilter extends OncePerRequestFilter {
  private final Map<String, Bucket> buckets = new ConcurrentHashMap<>();

  /*
   * セキュリティ：このフィルタはクライアント識別のためにrequest.getRemoteAddr()を使用
   * レート制限します。
   *
   * アプリケーションがリバースプロキシ（nginx、AWS ALBなど）の背後にある場合、
   * 正確なクライアントIP検出のためにSpringが転送ヘッダーを処理するように設定する必須：
   *
   * 1. application.properties/yamlで server.forward-headers-strategy=NATIVE
   * （クラウドプラットフォーム）またはFRAMEWORKを設定
   * 2. FRAMEWORKストラテジーを使用している場合、ForwardedHeaderFilterを登録：
   *
   *    @Bean
   *    ForwardedHeaderFilter forwardedHeaderFilter() {
   *        return new ForwardedHeaderFilter();
   *    }
   *
   * 3. プロキシがX-Forwarded-Forヘッダー（スプーフィング防止）に追加ではなく上書きするよう
   * 設定
   * 4. server.tomcat.remoteip.trusted-proxiesまたはコンテナ同等物を設定
   *
   * この設定なしでは、request.getRemoteAddr()はプロキシIPを返し、クライアントIPではなく。
   * X-Forwarded-Forを直接読むのはしてはいけません。これは信頼できたプロキシ処理なしで
   * 簡単にスプーフィングできます。
   */
  @Override
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
      FilterChain filterChain) throws ServletException, IOException {
    // getRemoteAddr()を使用します。これはForwardedHeaderFilterが
    // 設定されているときは正しいクライアントIPを返すか、
    // 設定されていないときはダイレクト接続IPです。転送ヘッダー
    // なしで直接信頼できる値を返してください。
    String clientIp = request.getRemoteAddr();

    Bucket bucket = buckets.computeIfAbsent(clientIp,
        k -> Bucket.builder()
            .addLimit(Bandwidth.classic(100, Refill.greedy(100, Duration.ofMinutes(1))))
            .build());

    if (bucket.tryConsume(1)) {
      filterChain.doFilter(request, response);
    } else {
      response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
    }
  }
}
```

## バックグラウンドジョブ

Springの`@Scheduled`を使用するか、キューと統合（例：Kafka、SQS、RabbitMQ）。ハンドラをべき等で観測可能に保ちます。

## 可観測性

- 構造化ログ（Logback JSON エンコーダー経由）
- メトリクス：Micrometer + Prometheus/OTel
- トレース：Micrometer Tracing with OpenTelemetry or Brave backend

## 本番デフォルト

- コンストラクタインジェクションを優先、フィールドインジェクションを回避
- RFC 7807エラー用に`spring.mvc.problemdetails.enabled=true`を有効化 (Spring Boot 3+)
- ワークロード用HikariCPプール サイズ設定、タイムアウト設定
- クエリに`@Transactional(readOnly = true)`を使用
- `@NonNull`と`Optional`経由のnull安全性を強制

**覚えておいてください**：コントローラーは薄く、サービスは焦点を絞り、リポジトリはシンプル、エラーはセントラライズしてください。保守性とテスト可能性のために最適化してください。
