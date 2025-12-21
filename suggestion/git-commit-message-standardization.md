# Gitコミットメッセージ共通化導入案
Commitizen / commitlint を用いた標準化の提案

## はじめに

Git を用いた開発において、**コミットメッセージの品質と一貫性**は保守性・可読性・チーム開発効率に直結する重要な要素です。

しかし実際の現場では、
- 人によって書き方が異なる
- 内容が曖昧で後から意図が分からない
- ルールを決めても守られない

といった課題が頻繁に発生します。

本ドキュメントでは、**ツールを用いてコミットメッセージを共通化・標準化する導入案**を提案します。


## 背景と課題

### 現状のよくあるコミットメッセージ

```text
fix
修正
対応
bug fix
〇〇の件
```

### この状態で起きる問題

- コミット履歴を見ても変更内容が分からない
- レビュー時に差分の意図を読み取る必要がある
- 障害調査や仕様確認に時間がかかる
- 人が増えるほど表記ゆれが加速する

## このまま運用を続けた場合のリスク

- Git の履歴が「資産」ではなく「ノイズ」になる
- 引き継ぎコスト・調査コストの増大
- Changelog 自動生成やリリース管理の自動化が困難
- 精神論に依存した運用になる

## 解決方針（結論）

**人の注意やルール徹底に頼らず、ツールによってコミットメッセージを共通化する**

- 書き方を考えさせない
- フォーマットを自動で守らせる
- 誰が書いても一定の品質を担保する

## 採用する考え方：Conventional Commits

本提案では、  
広く利用されている **Conventional Commits** の考え方をベースとします。

- 公式仕様（日本語）  
  https://www.conventionalcommits.org/ja/v1.0.0/

### 特徴

- コミットの意味が構造化される
- Changelog 生成や CI 連携と相性が良い
- 個別ルールではなく標準仕様

## 導入するライブラリ構成

| ライブラリ | 役割 |
|-----------|------|
| commitizen | 対話形式でコミットメッセージを作成 |
| cz-customizable | 質問内容・フォーマットのカスタマイズ |
| commitlint | コミットメッセージのルールチェック |
| commitlint-config-gitmoji | gitmoji 前提のルール定義 |

## 各ライブラリの概要

### commitizen

```bash
npx cz
```

- 対話形式でコミットメッセージを作成
- 書き方に迷わず、必要情報を漏れなく入力可能


### cz-customizable

- Commitizen の質問内容をプロジェクト向けに定義
- 日本語プロンプト対応
- gitmoji / 独自 type 定義が可能

### commitlint / commitlint-config-gitmoji

- コミットメッセージがルールに沿っているかをチェック
- ルール違反のコミットを防止
- CI / Git フックと連携可能

## 運用イメージ（Before / After）

### Before

```bash
git commit -m "修正"
```

### After

```text
🐛 fix(auth): ログイン失敗時のエラーメッセージを修正
```

## 導入後の全体フロー

```text
開発者
  ↓
Commitizen
  ↓
cz-customizable
  ↓
commitlint
  ↓
Git commit
```

## インストール方法

```bash
npm install --save-dev commitizen cz-customizable commitlint commitlint-config-gitmoji
```

## 初期設定（概要）

```json
{
  "config": {
    "commitizen": {
      "path": "cz-customizable"
    }
  }
}
```

## 導入コストと影響範囲

- 初期設定：数時間程度
- 学習コスト：ほぼ不要
- 既存履歴への影響なし


## 導入スコープ案

- フェーズ1：一部リポジトリで試験導入
- フェーズ2：全体展開・CI 連携


## まとめ

- コミットメッセージのばらつきは仕組みで解決すべき課題
- 標準仕様 + ツール導入で再現性の高い運用が可能
- 低コスト・高効果な改善施策

## 参考文献・関連リンク

- Conventional Commits  
  https://www.conventionalcommits.org/ja/v1.0.0/

- Commitizen  
  https://github.com/commitizen/cz-cli

- cz-customizable  
  https://github.com/leonardoanalista/cz-customizable

- commitlint  
  https://github.com/conventional-changelog/commitlint

- commitlint-config-gitmoji  
  https://github.com/carloscuesta/commitlint-config-gitmoji

- gitmoji  
  https://gitmoji.dev/

- Husky  
  https://github.com/typicode/husky
