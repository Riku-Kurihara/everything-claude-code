---
name: java-coding-standards
description: Spring Boot サービス向けの Java コーディング標準: 命名、不変性、Optional の使用、ストリーム、例外、ジェネリクス、プロジェクト レイアウト。
---

# Java コーディング標準

Spring Boot サービスの読みやすく、保守しやすい Java (17+) コードのための標準。

## コア原則

- 賢さより明確さを優先
- デフォルトで不変; 共有可能な可変状態を最小化
- 意味のある例外で迅速に失敗
- 一貫した命名とパッケージ構造

## 命名

```java
// ✅ クラス/レコード: PascalCase
public class MarketService {}
public record Money(BigDecimal amount, Currency currency) {}

// ✅ メソッド/フィールド: camelCase
private final MarketRepository marketRepository;
public Market findBySlug(String slug) {}

// ✅ 定数: UPPER_SNAKE_CASE
private static final int MAX_PAGE_SIZE = 100;
```

## 不変性

```java
// ✅ レコードと final フィールドを優先
public record MarketDto(Long id, String name, MarketStatus status) {}

public class Market {
  private final Long id;
  private final String name;
  // ゲッターのみ、セッターなし
}
```

## Optional の使用

```java
// ✅ find* メソッドから Optional を返す
Optional<Market> market = marketRepository.findBySlug(slug);

// ✅ get() の代わりに map/flatMap を使用
return market
    .map(MarketResponse::from)
    .orElseThrow(() -> new EntityNotFoundException("Market not found"));
```

## ストリームベストプラクティス

```java
// ✅ 変換にはストリームを使用し、パイプラインを短く保つ
List<String> names = markets.stream()
    .map(Market::name)
    .filter(Objects::nonNull)
    .toList();

// ❌ 複雑なネストされたストリームを避ける; 明確性のためにループを優先
```

## 例外

- ドメイン エラーには未チェック例外を使用; 技術的な例外はコンテキストでラップ
- ドメイン固有の例外を作成 (例: `MarketNotFoundException`)
- 広い `catch (Exception ex)` を避ける (中央で再スロー/ログしない限り)

```java
throw new MarketNotFoundException(slug);
```

## ジェネリクスと型安全性

- ロー型を避ける; ジェネリック パラメータを宣言
- 再利用可能なユーティリティに対して、境界付きジェネリックスを優先

```java
public <T extends Identifiable> Map<Long, T> indexById(Collection<T> items) { ... }
```

## プロジェクト構造 (Maven/Gradle)

```
src/main/java/com/example/app/
  config/
  controller/
  service/
  repository/
  domain/
  dto/
  util/
src/main/resources/
  application.yml
src/test/java/... (main をミラー)
```

## フォーマットとスタイル

- 一貫して 2 または 4 スペースを使用 (プロジェクト標準)
- 1 つのパブリック トップレベル型を 1 ファイルに
- メソッドを短く、フォーカスされたままに保つ; ヘルパーを抽出
- メンバー順: 定数、フィールド、コンストラクター、パブリック メソッド、protected、private

## 避けるべきコードのニオイ

- 長いパラメーター リスト → DTO/ビルダーを使用
- 深いネスト → 早期リターン
- マジック ナンバー → 名前付き定数
- 静的な可変状態 → 依存性の注入を優先
- サイレント catch ブロック → ログして実行するか、再スロー

## ログ

```java
private static final Logger log = LoggerFactory.getLogger(MarketService.class);
log.info("fetch_market slug={}", slug);
log.error("failed_fetch_market slug={}", slug, ex);
```

## null 処理

- `@Nullable` は避けられない場合にのみ受け入れる; そうでない場合は `@NonNull` を使用
- 入力では Bean Validation (`@NotNull`, `@NotBlank`) を使用

## テストの期待

- JUnit 5 + AssertJ for fluent assertions
- モック用に Mockito を使用; 可能な限り partial mocks を避ける
- 非決定論的テストを優先; 隠された sleeps はなし

**注意**: コードを意図的、型付き、観察可能なままに保ちます。必要性が証明されない限り、マイクロ最適化より保守性を優先します。
