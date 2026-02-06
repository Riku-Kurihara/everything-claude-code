| 名前 | 説明 |
|------|-------------|
| cloud-infrastructure-security | クラウドプラットフォームへのデプロイメント、インフラストラクチャの設定、IAMポリシー管理、ログ/モニタリング設定、CI/CDパイプライン実装時に使用。ベストプラクティスに沿ったクラウドセキュリティチェックリストを提供。 |

# クラウド＆インフラストラクチャセキュリティスキル

このスキルはクラウドインフラストラクチャ、CI/CDパイプライン、デプロイメント設定がセキュリティベストプラクティスを遵守し、業界標準に準拠していることを保証します。

## アクティベーション時期

- クラウドプラットフォーム（AWS、Vercel、Railway、Cloudflare）へのアプリケーションデプロイメント
- IAMロールと権限の設定
- CI/CDパイプラインの設定
- インフラストラクチャアズコード（Terraform、CloudFormation）の実装
- ログとモニタリングの設定
- クラウド環境でのシークレット管理
- CDNとエッジセキュリティの設定
- ディザスタリカバリとバックアップ戦略の実装

## クラウドセキュリティチェックリスト

### 1. IAM＆アクセス制御

#### 最小権限の原則

```yaml
# 正解：最小限の権限
iam_role:
  permissions:
    - s3:GetObject  # 読み取りアクセスのみ
    - s3:ListBucket
  resources:
    - arn:aws:s3:::my-bucket/*  # 特定のバケットのみ

# 誤り：過度に広い権限
iam_role:
  permissions:
    - s3:*  # すべてのS3アクション
  resources:
    - "*"  # すべてのリソース
```

#### 多要素認証 (MFA)

```bash
# ルート/管理者アカウントのMFAを常に有効化
aws iam enable-mfa-device \
  --user-name admin \
  --serial-number arn:aws:iam::123456789:mfa/admin \
  --authentication-code1 123456 \
  --authentication-code2 789012
```

#### 確認ステップ

- [ ] 本番環境でルートアカウントを使用していない
- [ ] すべての権限あるアカウントでMFAが有効
- [ ] サービスアカウントがロールを使用、長期認証情報ではなく
- [ ] IAMポリシーが最小権限に従っている
- [ ] アクセスレビューが定期的に実施されている
- [ ] 未使用の認証情報がローテーション・削除されている

### 2. シークレット管理

#### クラウドシークレットマネージャー

```typescript
// 正解：クラウドシークレットマネージャーを使用
import { SecretsManager } from '@aws-sdk/client-secrets-manager';

const client = new SecretsManager({ region: 'us-east-1' });
const secret = await client.getSecretValue({ SecretId: 'prod/api-key' });
const apiKey = JSON.parse(secret.SecretString).key;

// 誤り：ハードコードまたは環境変数のみ
const apiKey = process.env.API_KEY; // ローテーションなし、監査なし
```

#### シークレットローテーション

```bash
# データベース認証情報の自動ローテーションを設定
aws secretsmanager rotate-secret \
  --secret-id prod/db-password \
  --rotation-lambda-arn arn:aws:lambda:region:account:function:rotate \
  --rotation-rules AutomaticallyAfterDays=30
```

#### 確認ステップ

- [ ] すべてのシークレットがクラウドシークレットマネージャーに保存（AWS Secrets Manager、Vercel Secrets）
- [ ] データベース認証情報の自動ローテーションが有効
- [ ] APIキーが四半期ごとにローテーション
- [ ] コード、ログ、エラーメッセージにシークレットなし
- [ ] シークレットアクセスの監査ログが有効

### 3. ネットワークセキュリティ

#### VPCとファイアウォール設定

```terraform
# 正解：制限されたセキュリティグループ
resource "aws_security_group" "app" {
  name = "app-sg"

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]  # VPC内のみ
  }

  egress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # HTTPSアウトバウンドのみ
  }
}

# 誤り：インターネットに開かれている
resource "aws_security_group" "bad" {
  ingress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # すべてのポート、すべてのIP！
  }
}
```

#### 確認ステップ

- [ ] データベースが公開アクセス可能ではない
- [ ] SSH/RDPポートがVPN/bastionのみに制限されている
- [ ] セキュリティグループが最小権限に従っている
- [ ] ネットワークACLが設定されている
- [ ] VPCフローログが有効

### 4. ログ＆モニタリング

#### CloudWatch/ログ設定

```typescript
// 正解：包括的なログ出力
import { CloudWatchLogsClient, CreateLogStreamCommand } from '@aws-sdk/client-cloudwatch-logs';

const logSecurityEvent = async (event: SecurityEvent) => {
  await cloudwatch.putLogEvents({
    logGroupName: '/aws/security/events',
    logStreamName: 'authentication',
    logEvents: [{
      timestamp: Date.now(),
      message: JSON.stringify({
        type: event.type,
        userId: event.userId,
        ip: event.ip,
        result: event.result,
        // 機密データをログ出力しない
      })
    }]
  });
};
```

#### 確認ステップ

- [ ] CloudWatch/ログがすべてのサービスで有効
- [ ] 失敗した認証試行がログ記録されている
- [ ] 管理操作が監査されている
- [ ] ログ保有期間が設定されている（コンプライアンス用に90日以上）
- [ ] 異常なアクティビティのアラートが設定されている
- [ ] ログがセントラライズされ、改ざん防止されている

### 5. CI/CDパイプラインセキュリティ

#### セキュアなパイプライン設定

```yaml
# 正解：セキュアなGitHub Actionsワークフロー
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read  # 最小権限

    steps:
      - uses: actions/checkout@v4

      # シークレットスキャン
      - name: Secret scanning
        uses: trufflesecurity/trufflehog@main

      # 依存関係監査
      - name: Audit dependencies
        run: npm audit --audit-level=high

      # 長期トークンではなくOIDCを使用
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/GitHubActionsRole
          aws-region: us-east-1
```

#### サプライチェーンセキュリティ

```json
// package.json - ロックファイルと整合性チェックを使用
{
  "scripts": {
    "install": "npm ci",  // 再現可能なビルドのためにciを使用
    "audit": "npm audit --audit-level=moderate",
    "check": "npm outdated"
  }
}
```

#### 確認ステップ

- [ ] OIDCが長期認証情報の代わりに使用されている
- [ ] パイプラインでシークレットスキャンが実施されている
- [ ] 依存関係の脆弱性スキャンが実施されている
- [ ] コンテナイメージスキャン（該当する場合）
- [ ] ブランチ保護ルールが強制されている
- [ ] マージの前にコードレビューが必須
- [ ] サインされたコミットが強制されている

### 6. Cloudflare＆CDNセキュリティ

#### Cloudflareセキュリティ設定

```typescript
// 正解：セキュリティヘッダー付きCloudflare Workers
export default {
  async fetch(request: Request): Promise<Response> {
    const response = await fetch(request);

    // セキュリティヘッダーを追加
    const headers = new Headers(response.headers);
    headers.set('X-Frame-Options', 'DENY');
    headers.set('X-Content-Type-Options', 'nosniff');
    headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
    headers.set('Permissions-Policy', 'geolocation=(), microphone=()');

    return new Response(response.body, {
      status: response.status,
      headers
    });
  }
};
```

#### WAFルール

```bash
# Cloudflare WAFマネージドルールを有効化
# - OWASPコアルールセット
# - Cloudflareマネージドルールセット
# - レート制限ルール
# - ボット保護
```

#### 確認ステップ

- [ ] WAFがOWASPルールで有効
- [ ] レート制限が設定されている
- [ ] ボット保護がアクティブ
- [ ] DDoS保護が有効
- [ ] セキュリティヘッダーが設定されている
- [ ] SSL/TLSストリクトモードが有効

### 7. バックアップ＆ディザスタリカバリ

#### 自動バックアップ

```terraform
# 正解：自動RDSバックアップ
resource "aws_db_instance" "main" {
  allocated_storage     = 20
  engine               = "postgres"

  backup_retention_period = 30  # 30日保有
  backup_window          = "03:00-04:00"
  maintenance_window     = "mon:04:00-mon:05:00"

  enabled_cloudwatch_logs_exports = ["postgresql"]

  deletion_protection = true  # 誤削除を防止
}
```

#### 確認ステップ

- [ ] 自動日次バックアップが設定されている
- [ ] バックアップ保有期間がコンプライアンス要件を満たしている
- [ ] ポイントインタイムリカバリが有効
- [ ] バックアップテストが四半期ごとに実施されている
- [ ] ディザスタリカバリ計画が文書化されている
- [ ] RPOとRTOが定義され、テストされている

## 本番環境クラウドデプロイ前セキュリティチェックリスト

本番環境へのクラウドデプロイ前：

- [ ] **IAM**: ルートアカウントを使用していない、MFAが有効、最小権限ポリシー
- [ ] **シークレット**: すべてのシークレットがクラウドシークレットマネージャーにあり、ローテーション中
- [ ] **ネットワーク**: セキュリティグループが制限されている、公開データベースなし
- [ ] **ログ**: CloudWatch/ログが保有期間で有効
- [ ] **モニタリング**: 異常なアクティビティのアラート設定済み
- [ ] **CI/CD**: OIDC認証、シークレットスキャン、依存関係監査
- [ ] **CDN/WAF**: CloudflareのWAFがOWASPルールで有効
- [ ] **暗号化**: データが転送中と保存時に暗号化
- [ ] **バックアップ**: 自動バックアップにはテスト済みリカバリ
- [ ] **コンプライアンス**: GDPR/HIPAA要件が満たされている（該当する場合）
- [ ] **ドキュメント**: インフラが文書化、ランブックが作成されている
- [ ] **インシデント対応**: セキュリティインシデント計画が策定されている

## 一般的なクラウドセキュリティ設定ミス

### S3バケット露出

```bash
# 誤り：公開バケット
aws s3api put-bucket-acl --bucket my-bucket --acl public-read

# 正解：特定のアクセス付きプライベートバケット
aws s3api put-bucket-acl --bucket my-bucket --acl private
aws s3api put-bucket-policy --bucket my-bucket --policy file://policy.json
```

### RDS公開アクセス

```terraform
# 誤り
resource "aws_db_instance" "bad" {
  publicly_accessible = true  # これをしてはいけない！
}

# 正解
resource "aws_db_instance" "good" {
  publicly_accessible = false
  vpc_security_group_ids = [aws_security_group.db.id]
}
```

## リソース

- [AWS Security Best Practices](https://aws.amazon.com/security/best-practices/)
- [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services)
- [Cloudflare Security Documentation](https://developers.cloudflare.com/security/)
- [OWASP Cloud Security](https://owasp.org/www-project-cloud-security/)
- [Terraform Security Best Practices](https://www.terraform.io/docs/cloud/guides/recommended-practices/)

**覚えておいてください**：クラウド設定ミスはデータ侵害の主な原因です。1つの露出したS3バケットまたは過度に許可的なIAMポリシーがインフラストラクチャ全体を危険にさらす可能性があります。常に最小権限の原則と多層防御に従ってください。
