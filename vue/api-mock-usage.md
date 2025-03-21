# Vite プロジェクトにおける API Mock の切り替え仕様

## 目的
開発中、同一オリジンポリシー回避のため プロキシ設定 を行っている場合、mocks/mock.ts のモックレスポンスが返されてしまう課題を解決する。
環境変数 `VITE_APP_USE_MOCK` を使用し、モックサーバー使用有無を制御できるようにする。

## 必要ファイル
### .env ファイル
```env
# モックサーバーを使う場合は true に設定
VITE_APP_USE_MOCK=true
```
>**ポイント**
> - true → モックAPIを有効化
> - false または未定義 → 実際の外部APIに通信

### vite.config.ts 設定
```typescript
import { defineConfig, loadEnv } from 'vite'
import vue from '@vitejs/plugin-vue'
import { viteMockServe } from 'vite-plugin-mock'

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd())

  return {
    plugins: [
      viteMockServe({
        supportTs: false,
        mockPath: 'mocks',
        localEnabled: env.VITE_APP_USE_MOCK === 'true',
      }),
    ],
    // その他の設定は省略
  }
})
```
>**ポイント**
> - loadEnv(mode, process.cwd()) で .env の変数を読み込む。
> - localEnabled: env.VITE_APP_USE_MOCK === 'true' でモックの有効/無効を切り替え。
> - プロキシ設定はそのまま動作。