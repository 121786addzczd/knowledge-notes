# PlaywrightとCypressのE2Eテストツール比較

## 1. 比較表

| 項目 | Playwright | Cypress |
| --- | --- | --- |
| 推奨記法 | Page Object Model (POM) 推奨 | トランザクションスクリプト形式を推奨 |
| 要素取得方法 | locator() をPOM内で定義し、テストコードから呼び出す | cy.get() で直感的に直接取得 |
| 学習コスト | やや高め (POM設計の理解が必要) | 比較的低め (公式もシンプルな直書きスタイルを推奨) |
| テストコードの可読性 | POMでロジックを隠蔽し整理 | 1ファイル内でテストの流れが完結し読みやすい |
| API通信やヘッダー操作 | request()やcontext設定が柔軟 | cy.request() や cy.intercept() で対応可能 |
| セッション管理 |  storageState でログインセッション再利用がしやすい | 基本的に毎回UIログイン or Cookie/LocalStorage操作 |
| 開発体験 | マルチブラウザ対応 (Chromium, Firefox, WebKit) | Chromium中心。リアルタイムなデバッグがしやすい |
| ドキュメント | 詳細な設定や高度なカスタマイズが前提 | シンプルな例が豊富で直感的に始めやすい |

---

## 2. Cypress選定理由

### **トランザクションスクリプト形式でシンプルにテストが書ける**
Cypressは、**cy.get() → クリック → 入力 → 結果検証** という流れを **そのまま1ファイル内で記述**する形式を推奨しています。

- **公式の推奨もシンプルな記法**
- **特別なPOM設計やロジック隠蔽の必要なし**

### **直感的な要素取得**
要素取得も **cy.get('#selector')** といった形式で、**テストの流れが読みやすい & 修正しやすい**。

#### Cypress ログイン処理例
直感的・トランザクションスクリプトスタイル
```javascript
describe('Login Test', () => {
  it('ログイン成功', () => {
    cy.visit('/login')

    // 直感的に要素を取得して入力・クリック
    cy.get('#username').type('admin')
    cy.get('#password').type('password')
    cy.get('#login-button').click()

    // 結果確認
    cy.url().should('include', '/dashboard')
    cy.contains('Welcome, admin')
  })
})
```

#### Playwright ログイン処理例
**Page Object Model設計 + locator中心**
Page Object
```javascript
// pageObjects/LoginPage.ts
import { Page } from '@playwright/test'

export class LoginPage {
  constructor(private page: Page) {}

  async goto() {
    await this.page.goto('/login')
  }

  async login(username: string, password: string) {
    await this.page.locator('#username').fill(username)
    await this.page.locator('#password').fill(password)
    await this.page.locator('#login-button').click()
  }
}
```

テストファイル
```javascript
import { test, expect } from '@playwright/test'
import { LoginPage } from '../pageObjects/LoginPage'

test('ログイン成功', async ({ page }) => {
  const loginPage = new LoginPage(page)

  await loginPage.goto()
  await loginPage.login('admin', 'password')

  await expect(page).toHaveURL(/.*dashboard/)
  await expect(page.locator('text=Welcome, admin')).toBeVisible()
})
```


### **プロジェクト規模が大規模でない**
- **テスト対象が中〜小規模**
- 画面遷移やAPI通信もそこまで複雑ではない
- POMでがっちり設計するほどの保守コストは不要

### **E2Eテスト経験が豊富なメンバーがいない**
- Cypressは **導入から実行までが簡単**
- ドキュメントや公式サンプルも **直感的 & わかりやすい**
- 学習コスト・キャッチアップが **Playwrightより低い**

---

## 3. 結論

以上の理由から、**E2EテストツールとしてCypressを選定**することは **理にかなっており、現時点の体制・プロジェクト規模を考慮して最適** という判断。

---

## 4. 今後の運用方針

- **Custom Commandを活用し、ログインや共通処理は再利用**
- 基本は **トランザクションスクリプト形式** でテスト記述
- プロジェクト規模拡大・テスト件数増加時は **必要に応じてPage Objectパターンやヘルパー関数の導入も検討**
