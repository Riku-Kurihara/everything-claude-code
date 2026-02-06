---
name: jpa-patterns
description: Spring Boot における JPA/Hibernate パターン: エンティティ設計、関係、クエリ最適化、トランザクション、監査、インデックス、ページネーション、プーリング。
---

# JPA/Hibernate パターン

Spring Boot のデータモデリング、リポジトリ、パフォーマンスチューニングに使用します。

## エンティティ設計

```java
@Entity
@Table(name = "markets", indexes = {
  @Index(name = "idx_markets_slug", columnList = "slug", unique = true)
})
@EntityListeners(AuditingEntityListener.class)
public class MarketEntity {
  @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  @Column(nullable = false, length = 200)
  private String name;

  @Column(nullable = false, unique = true, length = 120)
  private String slug;

  @Enumerated(EnumType.STRING)
  private MarketStatus status = MarketStatus.ACTIVE;

  @CreatedDate private Instant createdAt;
  @LastModifiedDate private Instant updatedAt;
}
```

監査を有効にします:
```java
@Configuration
@EnableJpaAuditing
class JpaConfig {}
```

## 関係と N+1 防止

```java
@OneToMany(mappedBy = "market", cascade = CascadeType.ALL, orphanRemoval = true)
private List<PositionEntity> positions = new ArrayList<>();
```

- デフォルトで遅延読み込みを使用; 必要に応じてクエリで `JOIN FETCH` を使用
- コレクションで `EAGER` を避ける; 読み取りパスで DTO プロジェクションを使用

```java
@Query("select m from MarketEntity m left join fetch m.positions where m.id = :id")
Optional<MarketEntity> findWithPositions(@Param("id") Long id);
```

## リポジトリパターン

```java
public interface MarketRepository extends JpaRepository<MarketEntity, Long> {
  Optional<MarketEntity> findBySlug(String slug);

  @Query("select m from MarketEntity m where m.status = :status")
  Page<MarketEntity> findByStatus(@Param("status") MarketStatus status, Pageable pageable);
}
```

軽量なクエリにはプロジェクションを使用:
```java
public interface MarketSummary {
  Long getId();
  String getName();
  MarketStatus getStatus();
}
Page<MarketSummary> findAllBy(Pageable pageable);
```

## トランザクション

- サービス メソッドで `@Transactional` でアノテート
- 読み取りパスで `@Transactional(readOnly = true)` を使用して最適化
- 伝播を注意深く選択; 長時間実行されるトランザクションを避ける

```java
@Transactional
public Market updateStatus(Long id, MarketStatus status) {
  MarketEntity entity = repo.findById(id)
      .orElseThrow(() -> new EntityNotFoundException("Market"));
  entity.setStatus(status);
  return Market.from(entity);
}
```

## ページネーション

```java
PageRequest page = PageRequest.of(pageNumber, pageSize, Sort.by("createdAt").descending());
Page<MarketEntity> markets = repo.findByStatus(MarketStatus.ACTIVE, page);
```

カーソルのようなページネーションの場合、JPQL に `id > :lastId` を含め、順序付けします。

## インデックスとパフォーマンス

- 一般的なフィルター (`status`、`slug`、外部キー) にインデックスを追加
- クエリパターンに一致する複合インデックスを使用 (`status, created_at`)
- `select *` を避ける; 必要な列のみをプロジェクト
- `saveAll` で書き込みをバッチし、`hibernate.jdbc.batch_size` を使用

## コネクション プーリング (HikariCP)

推奨プロパティ:
```
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.validation-timeout=5000
```

PostgreSQL LOB 処理の場合、以下を追加:
```
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
```

## キャッシング

- 第 1 レベル キャッシュは EntityManager ごと; トランザクション間でエンティティを保持しない
- 読み取り集約型エンティティの場合、第 2 レベル キャッシュを慎重に検討; 削除戦略を検証

## マイグレーション

- Flyway または Liquibase を使用; 本番環境では Hibernate 自動 DDL に依存しない
- マイグレーションはべき等で加算的に保つ; 計画なしに列をドロップしない

## データアクセステスト

- Testcontainers で `@DataJpaTest` を推奨して本番環境をミラー
- ログを使用して SQL 効率を確認: `logging.level.org.hibernate.SQL=DEBUG` および `logging.level.org.hibernate.orm.jdbc.bind=TRACE` をパラメータ値に設定

**注意**: エンティティをシンプルに保ち、クエリを意図的に、トランザクションを短く。フェッチ戦略とプロジェクションで N+1 を防止し、読み取り/書き込みパスにインデックスを作成します。
