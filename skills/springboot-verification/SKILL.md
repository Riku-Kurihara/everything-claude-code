---
name: springboot-verification
description: Spring Boot プロジェクトの検証ループ：ビルド、静的分析、カバレッジ付きテスト、セキュリティスキャン、リリースまたは PR 前の差分レビュー。
---

# Spring Boot 検証ループ

PR の前に、大きな変更後、デプロイ前に実行します。

## フェーズ 1: ビルド

```bash
mvn -T 4 clean verify -DskipTests
# または
./gradlew clean assemble -x test
```

ビルドが失敗した場合は停止して修正してください。

## フェーズ 2: 静的分析

Maven（一般的なプラグイン）：
```bash
mvn -T 4 spotbugs:check pmd:check checkstyle:check
```

Gradle（設定されている場合）：
```bash
./gradlew checkstyleMain pmdMain spotbugsMain
```

## フェーズ 3: テスト + カバレッジ

```bash
mvn -T 4 test
mvn jacoco:report   # 80% 以上のカバレッジを確認
# または
./gradlew test jacocoTestReport
```

レポート：
- テスト総数、成功/失敗
- カバレッジ %（行/ブランチ）

## フェーズ 4: セキュリティスキャン

```bash
# 依存関係の CVE
mvn org.owasp:dependency-check-maven:check
# または
./gradlew dependencyCheckAnalyze

# シークレット（git）
git secrets --scan  # 設定されている場合
```

## フェーズ 5: Lint/フォーマット（オプションゲート）

```bash
mvn spotless:apply   # Spotless プラグインを使用している場合
./gradlew spotlessApply
```

## フェーズ 6: 差分レビュー

```bash
git diff --stat
git diff
```

チェックリスト：
- デバッグログが残されていない（`System.out`、保護されていない `log.debug`）
- 意味のあるエラーとHTTPステータス
- トランザクションと検証が必要な場所に存在
- 設定の変更をドキュメント化

## 出力テンプレート

```
検証レポート
============
ビルド:     [PASS/FAIL]
静的:      [PASS/FAIL] (spotbugs/pmd/checkstyle)
テスト:    [PASS/FAIL] (X/Y 成功、Z% カバレッジ)
セキュリティ: [PASS/FAIL] (CVE 検出数: N)
差分:      [X ファイル変更]

全体:      [準備完了 / 準備未完了]

修正する問題:
1. ...
2. ...
```

## 継続モード

- 重要な変更があった場合、または長いセッション中は 30 ～ 60 分ごとにフェーズを再実行
- 短いループを維持：`mvn -T 4 test` + spotbugs で迅速なフィードバック

**重要**: 迅速なフィードバックは遅い驚きに勝ります。ゲートを厳格に保つ—本番システムでは警告を欠陥として扱ってください。
