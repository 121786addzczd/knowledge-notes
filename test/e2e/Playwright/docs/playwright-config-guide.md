# Playwright 設定ファイル解説（playwright.config.ts）

このドキュメントは、`test/e2e/Playwright/playwright.config.ts` の内容を解説する。

対象の設定ファイルは、Playwright Test の標準的な構成をベースにしており、複数ブラウザでの実行や CI 向けの挙動を明示している。

## 目的

- 実行対象のディレクトリやテストの並列実行の考え方を理解する
- CI とローカルで挙動が変わるポイントを把握する
- プロジェクト（ブラウザ）ごとの設定の意味を知る

## 設定全体の概要

設定は `defineConfig` によってまとめられ、主に以下の項目で構成される。

- テストの探索対象（testDir）
- 並列実行やリトライなどの実行制御
- レポーター設定
- 共通のブラウザ設定（use）
- ブラウザ別プロジェクト設定（projects）
- （任意）web サーバ起動の設定

## 主要設定の解説

### testDir

```ts
  testDir: './tests',
```

- テストを探索するディレクトリを `./tests` に指定している。
- `testMatch` を指定していないため、Playwright のデフォルトパターンで `tests` 配下のテストが対象になる。

### testMatch

```ts
  testMatch: process.env.TEST_MATCH ? [process.env.TEST_MATCH] : undefined,
```

- 環境変数 `TEST_MATCH` が設定されている場合のみ、そのパターンで実行対象を絞り込む。
- 未設定の場合は `undefined` となり、Playwright のデフォルトパターンが使われる。
- デフォルトのパターンは `**/*.@(spec|test).(js|ts|mjs|mts|cjs|cts|jsx|tsx)` で、`testDir` 配下の該当ファイルが対象になる。

### fullyParallel

```ts
  fullyParallel: true,
```

- 同一ファイル内のテストも並列で実行する設定。
- テストが独立している場合は実行時間の短縮に有効。

### forbidOnly

```ts
  forbidOnly: !!process.env.CI,
```

- CI 上では `test.only` が残っていた場合に失敗させる。
- ローカルでは `false` なので開発時の試行がしやすい。

### retries

```ts
  retries: process.env.CI ? 2 : 0,
```

- CI のみリトライを 2 回に設定。
- ローカルでは即失敗するため、素早くフィードバックを得られる。

### workers

```ts
  workers: process.env.CI ? 1 : undefined,
```

- CI では 1 worker に固定し、安定性を優先。
- ローカルでは Playwright のデフォルト worker 数を利用。

### reporter

```ts
  reporter: 'html',
```

- HTML レポートを出力する。
- `npx playwright show-report` で確認可能。

### use

```ts
  use: {
    trace: 'on-first-retry',
    video: 'on',
    ignoreHTTPSErrors: true,
  },
```

- 全プロジェクト共通の設定。
- 1 回目のリトライ時にトレースを取得して、失敗原因の調査をしやすくする。
- video を on にして、各テストの録画を常に残す。
- ignoreHTTPSErrors を true にして、自己署名証明書などの HTTPS エラーを無視する。

### projects

```ts
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox',  use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit',   use: { ...devices['Desktop Safari'] } },
  ],
```

- Chromium / Firefox / WebKit の 3 ブラウザで同じテストを実行する設定。
- `devices` を使って Desktop 向けの標準設定を適用している。
- さらにモバイルや特定ブラウザ向けの設定例がコメントで残されている。

### webServer（コメントアウト）

```ts
  // webServer: {
  //   command: 'npm run start',
  //   url: 'http://localhost:3000',
  //   reuseExistingServer: !process.env.CI,
  // },
```

- テスト前にローカルサーバを起動するための設定。
- コメントアウトされているため現在は無効。
- E2E テストでアプリを自動起動したい場合に有効化する。

## 参考リンク（公式）

- テスト設定の全体: https://playwright.dev/docs/test-configuration
- testMatch: https://playwright.dev/docs/test-configuration#testmatch
- reporter: https://playwright.dev/docs/test-reporters
- trace viewer: https://playwright.dev/docs/trace-viewer
- webServer: https://playwright.dev/docs/test-configuration#webserver
