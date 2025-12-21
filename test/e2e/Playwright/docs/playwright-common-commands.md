# Playwright よく使うコマンド集

目的: Playwright の実行・デバッグ・メンテで開発効率を上げるためのコマンドを用途別に整理する。

前提:
- Node.js プロジェクトで以下のパッケージを使用している想定
  ```txt
  @playwright/test
  ```
- 以下のコマンドを利用（グローバルインストール不要）
  ```sh
  npx playwright
  ```

## 基本実行

- 全テスト実行
  ```sh
  npx playwright test
  ```
  まずはこれで全体が動くか確認する。
- 特定のファイルを実行
  ```sh
  npx playwright test tests/e2e/login.spec.ts
  ```
  変更の影響範囲が限定的なときに有効。
- タイトル/grep で絞り込み
  ```sh
  npx playwright test -g "login succeeds"
  ```
  目的のケースだけ素早く回せる。
- プロジェクト指定（複数ブラウザ構成時）
  ```sh
  npx playwright test --project=chromium
  ```
  不要なブラウザ実行を避けて時間短縮。

## 失敗時の調査

- 失敗のみ再実行
  ```sh
  npx playwright test --last-failed
  ```
  連続試行時の待ち時間を削減。
- トレースを開く
  ```sh
  npx playwright show-trace trace.zip
  ```
  テスト中の操作やスクリーンショットを時系列で確認できる。
- 失敗時のみトレースを取得
  ```sh
  npx playwright test --trace=retain-on-failure
  ```
  成功時は軽く、失敗時は詳しく取れる。
- 失敗時のみ動画・スクショ
  ```sh
  npx playwright test --video=retain-on-failure --screenshot=only-on-failure
  ```
  CI のログ確認を短縮。

## デバッグ実行

- UI モードで実行
  ```sh
  npx playwright test --ui
  ```
  テスト探索・再実行・トレース閲覧が GUI でできる。
- ヘッドフル実行
  ```sh
  npx playwright test --headed
  ```
  ブラウザを目視しながら挙動確認。
- 低速モード（操作を遅くする）
  ```sh
  npx playwright test --headed --slow-mo=500
  ```
  操作が速すぎて追えないときに有効。
- デバッガ起動
  ```sh
  PWDEBUG=1 npx playwright test
  npx playwright test --debug
  ```
  Playwright Inspector でステップ実行できる。--debug は PWDEBUG=1 と同様。

## 生成・補助

- コードジェネレーター
  ```sh
  npx playwright codegen http://localhost:3000
  ```
  画面操作からテスト雛形を作れる。
- セレクタ確認
  ```sh
  npx playwright codegen --inspect http://localhost:3000
  ```
  locator の候補や安定性を確認できる。
- スクリーンショット取得（手動）
  ```sh
  npx playwright screenshot http://localhost:3000 out.png
  ```
  画面状態を手早く共有したいときに使う。

## レポート確認

- HTML レポートを開く
  ```sh
  npx playwright show-report
  ```
  直近の実行結果を視覚的に確認。
- 出力ディレクトリ指定
  ```sh
  npx playwright test --output=./test-results
  ```
  生成物の保存先を明示して管理しやすくする。

## 便利な実行オプション

- 並列数を固定
  ```sh
  npx playwright test --workers=4
  ```
  CI のリソースに合わせて最適化。
- タイムアウトを延長
  ```sh
  npx playwright test --timeout=60000
  ```
  重い E2E のみに適用する想定。
- テスト対象のベース URL 指定
  ```sh
  BASE_URL=https://staging.example.com npx playwright test
  ```
  環境切り替えをコマンドで制御。
- 環境変数で実行対象を絞り込み
  ```sh
  TEST_MATCH="A*.ts" npx playwright test
  ```
  設定側で process.env.TEST_MATCH を参照する場合に使える。未対応の場合は設定追加が必要。
- 既定の対象は testDir 配下のテストファイル
  testMatch を明示しない場合、Playwright のデフォルトパターンに一致するファイルのみ実行される。
  デフォルトパターンは `**/*.@(spec|test).(js|ts|mjs|mts|cjs|cts|jsx|tsx)`。
  公式: https://playwright.dev/docs/test-configuration#testmatch
- コンフィグ指定
  ```sh
  npx playwright test -c playwright.staging.config.ts
  ```
  環境別の設定を切り替えるときに使う。

## CI 向け

- JUnit レポーター
  ```sh
  npx playwright test --reporter=junit
  ```
  CI のテスト可視化に使える。
- 失敗時だけ再試行
  ```sh
  npx playwright test --retries=2
  ```
  一時的なフレークを吸収。
