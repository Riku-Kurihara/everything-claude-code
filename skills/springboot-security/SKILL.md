---
name: springboot-security
description: Spring Securityベストプラクティス。認証/認可、検証、CSRF、シークレット、ヘッダー、レート制限、依存関係セキュリティ。JavaSpring Bootサービス用。
---

# Spring Bootセキュリティレビュー

認証追加、入力処理、エンドポイント作成、またはシークレット処理時に使用。

## 認証

- ステートレスJWTまたは失効リスト付き不透明トークンを優先
- セッション用に`httpOnly`、`Secure`、`SameSite=Strict`クッキーを使用
- `OncePerRequestFilter`またはリソースサーバーでトークンを検証

```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {
  private final JwtService jwtService;

  public JwtAuthFilter(JwtService jwtService) {
    this.jwtService = jwtService;
  }

  @Override
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
      FilterChain chain) throws ServletException, IOException {
    String header = request.getHeader(HttpHeaders.AUTHORIZATION);
    if (header != null && header.startsWith("Bearer ")) {
      String token = header.substring(7);
      Authentication auth = jwtService.authenticate(token);
      SecurityContextHolder.getContext().setAuthentication(auth);
    }
    chain.doFilter(request, response);
  }
}
```

## 認可

- メソッドセキュリティを有効化：`@EnableMethodSecurity`
- `@PreAuthorize("hasRole('ADMIN')")`または`@PreAuthorize("@authz.canEdit(#id)")`を使用
- デフォルトで拒否、必須スコープのみを露出

## 入力検証

- コントローラーで`@Valid`を使用してBean検証を適用
- DTO上の制約を適用：`@NotBlank`、`@Email`、`@Size`、カスタム検証
- レンダリング前にHTMLをホワイトリストでサニタイズ

## SQLインジェクション防止

- Spring Dataリポジトリまたはパラメータ化クエリを使用
- ネイティブクエリの場合、`:param`バインディングを使用、文字列連結なし

## CSRF保護

- ブラウザセッションアプリの場合、CSRFを有効に保つ、トークンをフォーム/ヘッダーに含める
- Bearerトークン付き純粋APIの場合、CSRFを無効にしてステートレス認証に依存

```java
http
  .csrf(csrf -> csrf.disable())
  .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
```

## シークレット管理

- ソースにシークレットなし、環境またはvaultから読み込み
- `application.yml`を認証情報から保ち、プレースホルダーを使用
- トークンとDB認証情報を定期的にローテーション

## セキュリティヘッダー

```java
http
  .headers(headers -> headers
    .contentSecurityPolicy(csp -> csp
      .policyDirectives("default-src 'self'"))
    .frameOptions(HeadersConfigurer.FrameOptionsConfig::sameOrigin)
    .xssProtection(Customizer.withDefaults())
    .referrerPolicy(rp -> rp.policy(ReferrerPolicyHeaderWriter.ReferrerPolicy.NO_REFERRER)));
```

## レート制限

- 高コストエンドポイントでBucket4jまたはゲートウェイレベルの制限を適用
- バースト時にログとアラート、429とリトライヒントを返す

## 依存関係セキュリティ

- CI内でOWASP Dependency Check / SnyKを実行
- Spring BootおよびSpring Securityをサポート対象バージョンに保つ
- 既知のCVEでビルドを失敗させる

## ログとPII

- シークレット、トークン、パスワード、または完全なPANデータをログ出力しない
- 機密フィールドをマスク、構造化JSONログを使用

## ファイルアップロード

- サイズ、コンテンツタイプ、拡張子を検証
- Webルート外に保存、必要に応じてスキャン

## リリース前チェックリスト

- [ ] 認証トークンが正しく検証・失効
- [ ] すべての機密パスで認可ガード
- [ ] すべての入力が検証・サニタイズ
- [ ] 文字列連結SQLなし
- [ ] アプリタイプに対するCSF体制が正解
- [ ] シークレットが外部化、コミットなし
- [ ] セキュリティヘッダー設定済み
- [ ] APIにレート制限
- [ ] 依存関係スキャン済み、最新
- [ ] ログに機密データなし

**覚えておいてください**：デフォルトで拒否、入力を検証、最小権限、設定優先でセキュアにしてください。
