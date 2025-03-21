# Vue.js + Vite における CORS 回避設定ドキュメント

## はじめに
本ドキュメントでは、Vue.js (Vite) を使用したフロントエンドからバックエンドサーバーへAPI通信を行う際に発生する 同一オリジンポリシー違反（CORSエラー） を、ViteのProxy機能を用いて回避する方法について解説します。

## 概要
Vue.js でフロントエンドを開発し、バックエンドAPIが異なるポートやドメインで動いている場合、CORS (Cross-Origin Resource Sharing) エラーが発生します。

Viteの 開発用Proxy機能 を活用することで、ローカル開発時のCORS問題を回避し、スムーズにAPI通信ができるように設定します。

## 設定方法
### 1. vite.config.ts に Proxy 設定を追加
```typescript
import { fileURLToPath, URL } from 'url'
import { defineConfig } from 'vite'
import Vue from '@vitejs/plugin-vue'
import VueJsx from '@vitejs/plugin-vue-jsx'
import AutoImport from 'unplugin-auto-import/vite'
import { viteMockServe } from 'vite-plugin-mock'
import Components from 'unplugin-vue-components/vite'
import VueMacros from 'unplugin-vue-macros/vite'

export default ({ mode }) => {
  return defineConfig({
    server: {
      proxy: {
        // バックエンドAPIが http://localhost:8000/api/v1 で動いている場合
        '/api/v1': {
          target: 'http://localhost:8000',
          changeOrigin: true,
        },
      },
    },
    // その他の設定は省略
  })
}

```

### 2. Vue側でのAPIリクエスト例
```typescript
<template>
  <div>
    <button @click="getUsers">Get Users</button>
  </div>
</template>

<script setup>
const getUsers = async () => {
  const response = await fetch('/api/v1/users') // そのまま /api/v1/users にアクセス
  const data = await response.json()
  console.log(data)
}
</script>
```


### Proxy設定項目の解説
| 設定項目 | 説明 |
| --- | --- |
| target | バックエンドAPIのURL (例: http://localhost:8000) |
| changeOrigin | true にすることで、リクエストヘッダーの Origin をターゲットURLに変更しCORSを回避 |

### 注意点
- この設定は開発環境専用
本番環境では、APIサーバーと同じドメインで動かすか、バックエンド側でCORSヘッダーを正しく設定する必要があります。

- APIバージョン管理
バックエンドが /api/v1 で提供している場合は、フロントエンドもそのまま /api/v1 を使う方が自然です。


## まとめ
- Viteの server.proxy 設定でCORS回避可能
- 本番環境は別途CORS設定が必要