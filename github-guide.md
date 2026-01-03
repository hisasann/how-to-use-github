# GitHub活用ガイド（軽く触ったことがある人向け）

---

## プルリクエスト（PR）

### ドラフトPRの活用

ドラフトPRは「まだ作業中だけど見てほしい」というときに使います。

**作り方：**
PRを作るとき「Create pull request」の横の矢印から「Create draft pull request」を選ぶ

**フィードバックの依頼方法：**
- 右サイドバーの「Reviewers」から人を指定
- コメントで「@tanaka ここの設計どう思いますか？」とメンション
- 特定の行にコメントを残して意見を求める

ドラフト状態だとマージボタンが無効なので、「間違ってマージされる」心配なく見てもらえます。完成したら「Ready for review」ボタンで正式なPRに変更します。

---

### レビュアーの指定とCODEOWNERSファイル

リポジトリ内の特定のファイルやディレクトリに対して、自動的にレビュアーを割り当てる仕組みです。

`.github/CODEOWNERS`に置きます：

```
# フロントエンドのコードはfrontendチームが自動でレビュアーに
/src/frontend/    @hoge/frontend-team

# ドキュメントは久松さんが担当
/docs/            @hisasann

# 全体のデフォルト
*                 @lead-developer
```

PRで該当ファイルが変更されると、指定された人やチームが自動でレビュアーに追加されます。「誰にレビュー頼めばいい？」を仕組み化できるので、チーム開発で便利です。

---

### PRテンプレートの作成

`.github/PULL_REQUEST_TEMPLATE.md`を作成すると、PR作成時に自動でテンプレートが挿入されます。

```markdown
## 概要
<!-- このPRで何を変更したか -->

## 変更の種類
- [ ] バグ修正
- [ ] 新機能
- [ ] リファクタリング
- [ ] ドキュメント更新

## テスト
<!-- どのようにテストしたか -->

## 関連Issue
<!-- Fixes #123 -->

## レビュアーへのメモ
<!-- 特に見てほしいポイント -->
```

書くべきことが明確になり、レビューの質が上がります。チームで統一したフォーマットを使えるのもメリットです。

---

### Squash / Merge commit / Rebase の違いと使い分け

3つのマージ方法があります。

**Merge commit（デフォルト）**
```
main:     A---B-------M
               \     /
feature:        C---D
```
- 全コミット履歴を保持
- マージコミット（M）が作られる
- 履歴が複雑になりがち

**Squash merge**
```
main:     A---B---CD'
               \
feature:        C---D（これらは1つにまとめられる）
```
- featureブランチの全コミットを1つにまとめる
- 履歴がきれいになる
- 細かい作業コミットを隠せる

**Rebase merge**
```
main:     A---B---C'---D'
```
- featureのコミットをmainの先頭に付け替える
- マージコミットが作られない
- 直線的な履歴になる

**使い分けの目安：**
- 機能単位で履歴をまとめたい → Squash
- コミット履歴を全部残したい → Merge commit
- 直線的な履歴が好み → Rebase

---

### コンフリクト解消の手順

PRで「Conflicts」と表示されたら：

**GitHub上で解消する場合（簡単なもの）**

1. PRページで「Resolve conflicts」ボタンを押す
2. エディタが開く。`<<<<<<<`と`>>>>>>>`で囲まれた部分を編集
3. 残したい内容だけにして、マーカーを削除
4. 「Mark as resolved」→「Commit merge」

**ローカルで解消する場合（複雑なもの）**

```bash
git checkout feature-branch
git fetch origin
git merge origin/main
# コンフリクトが発生したファイルを手動で編集
git add .
git commit -m "Resolve conflicts"
git push
```

---

## Issue

### Issueテンプレートでバグ報告・機能要望を整理する

`.github/ISSUE_TEMPLATE/`ディレクトリにテンプレートを作成します。

**バグ報告用：`.github/ISSUE_TEMPLATE/bug_report.md`**

```markdown
---
name: バグ報告
about: 不具合を報告する
labels: bug
---

## バグの概要
<!-- 何が起きたか -->

## 再現手順
1.
2.
3.

## 期待する動作
<!-- 本来どうなるべきか -->

## 実際の動作
<!-- 実際に何が起きたか -->

## 環境
- OS:
- ブラウザ:
- バージョン:
```

**機能要望用：`.github/ISSUE_TEMPLATE/feature_request.md`**

```markdown
---
name: 機能要望
about: 新機能のアイデアを提案する
labels: enhancement
---

## 解決したい課題
<!-- どんな問題を解決したいか -->

## 提案する解決策
<!-- どうすれば解決できるか -->

## 代替案
<!-- 他に考えた方法があれば -->
```

報告者が何を書けばいいか迷わなくなり、必要な情報が揃った状態でIssueが作られます。

---

### PRとIssueの紐付け

PRの説明文やコミットメッセージに特定のキーワードを書くと、マージ時にIssueが自動クローズされます。

**使えるキーワード：**
- `Fixes #123`
- `Closes #123`
- `Resolves #123`

```markdown
## このPRについて
ログイン時のバリデーションを修正しました。

Fixes #42
```

PRがマージされると、Issue #42が自動的にクローズされます。複数のIssueを閉じたい場合は`Fixes #42, Fixes #43`のように書きます。

---

## CI/CD（GitHub Actions）

### 基本的なワークフロー構文

`.github/workflows/`ディレクトリにYAMLファイルを置きます。

```yaml
name: CI  # ワークフローの名前

on:  # トリガー
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:  # 実行するジョブ
  test:
    runs-on: ubuntu-latest  # 実行環境

    steps:  # ジョブ内のステップ
      - uses: actions/checkout@v4  # リポジトリをチェックアウト

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test # <- ここを実行するのがこのワークフローの目的
```

**主要な要素：**
- `on`: いつ実行するか（push、pull_request、scheduleなど）
- `jobs`: 並列実行できる単位
- `steps`: 順番に実行される処理
- `uses`: 既存のアクションを使う
- `run`: シェルコマンドを実行

---

### よくあるユースケース

**テスト自動実行**

```yaml
- name: Run tests
  run: npm test
```

**リント（コード品質チェック）**

```yaml
- name: Lint
  run: npm run lint
```

**デプロイ（mainブランチのみ）**

```yaml
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to production
        run: ./deploy.sh
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```

**定期実行（cronジョブ）**

```yaml
on:
  schedule:
    - cron: '0 9 * * 1'  # 毎週月曜9時（UTC）
```

---

### シークレットの管理方法

APIキーやパスワードをActions内で安全に使う仕組みです。

**設定場所：**
リポジトリの Settings → Secrets and variables → Actions → New repository secret

**使い方（ワークフロー内）：**

```yaml
steps:
  - name: Deploy
    env:
      API_KEY: ${{ secrets.MY_API_KEY }}
    run: ./deploy.sh
```

コードに直接書かず、`secrets.XXX`で参照します。ログにも表示されないようマスクされます。

Organization全体で共有するシークレットも設定できます。

---

### ブランチ保護ルールとの組み合わせ

Settings → Branches → Add branch protection rule で設定します。

**よく使う設定：**
- `Require a pull request before merging` - 直接pushを禁止
- `Require approvals` - レビュー承認必須（人数も指定可能）
- `Require status checks to pass` - CI通過必須
- `Require conversation resolution before merging` - コメント解決必須

**CI通過必須の設定例：**

1. 「Require status checks to pass before merging」にチェック
2. 「Status checks that are required」で対象のジョブを選択（例：`test`）

これでCIが失敗したらマージボタンが押せなくなります。

---

### マーケットプレイスのアクション活用

自分で書かなくても、公開されているアクションを`uses`で使えます。

**便利なアクション例：**

```yaml
# Node.jsセットアップ
- uses: actions/setup-node@v4

# Pythonセットアップ
- uses: actions/setup-python@v5

# キャッシュで高速化
- uses: actions/cache@v4

# Slackに通知
- uses: slackapi/slack-github-action@v1

# AWS認証
- uses: aws-actions/configure-aws-credentials@v4
```

GitHub Marketplace（https://github.com/marketplace?type=actions）で検索できます。スターが多いもの、Verifiedマークがついているものが安心です。

個人的には、 [Gemini Code Assist を使用して GitHub コードを確認する  |  Google for Developers](https://developers.google.com/gemini-code-assist/docs/review-github-code?hl=ja) がとてもおすすめです。

---

## その他の便利機能

### ブランチ保護ルール

Settings → Branches → Add branch protection rule で設定します。

**設定できること：**
- 直接pushを禁止（PRを必須に）
- レビュー承認を必須に（人数も指定可能）
- CI通過を必須に
- 管理者にもルールを適用
- 強制プッシュを禁止

`main`ブランチに設定しておくと、壊れたコードがデプロイされるリスクを減らせます。

---

### GitHub Codespaces

ブラウザ上で完全な開発環境を立ち上げられます。ただ、"." を打つだけで起動します。

**使い方：**
リポジトリページで「Code」→「Codespaces」→「Create codespace on main」

**特徴：**
- VS Codeがブラウザで動く
- ターミナルも使える
- 拡張機能もインストール可能
- ローカル環境構築が不要

**カスタマイズ：**
`.devcontainer/devcontainer.json`で環境を定義できます。

```json
{
  "image": "mcr.microsoft.com/devcontainers/javascript-node:20",
  "postCreateCommand": "npm install",
  "customizations": {
    "vscode": {
      "extensions": ["esbenp.prettier-vscode"]
    }
  }
}
```

新メンバーが「環境構築で1日かかる」問題を解消できます。

---

### Dependabot

依存関係の更新PRを自動で作ってくれます。

**設定方法：**
`.github/dependabot.yml`を作成します。

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    labels:
      - "dependencies"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "monthly"
```

**できること：**
- セキュリティアップデートの自動PR
- 定期的なバージョンアップデートPR
- 対応パッケージマネージャー：npm、pip、Composer、Bundler、Cargo、Dockerなど

古い依存関係の放置によるセキュリティリスクを減らせます。

---

### GitHub Discussions

Issueより緩い議論の場です。チャットというよりは**フォーラム**に近いです。

**Issueとの違い：**

| | Issue | Discussions |
|---|---|---|
| 用途 | バグ報告、タスク管理 | 質問、アイデア、雑談 |
| 状態 | Open/Closed | 回答済みマーク可能 |
| 性質 | 解決すべき課題 | 会話・議論 |

**よくある使い方：**
- Q&Aカテゴリで「〇〇の使い方がわからない」
- Ideasカテゴリで「こんな機能どうですか？」
- Generalで「v2.0リリースしました！」のお知らせ

有効にするにはリポジトリのSettings → General → Features → Discussionsにチェックを入れます。

---

### キーボードショートカット

**どのページでも使える：**
- `?` - ショートカット一覧を表示
- `.` - Web Editor（VS Code）を開く
- `s` または `/` - 検索にフォーカス

**リポジトリページで：**
- `t` - ファイル検索
- `w` - ブランチ切り替え
- `l` - 行番号へジャンプ（ファイル表示中）

**Issue/PRページで：**
- `c` - 新規Issue作成
- `a` - アサイン設定
- `l` - ラベル設定
- `m` - マイルストーン設定

特に`.`でのWeb Editorは、ちょっとした修正をブラウザだけで完結させたいときに便利です。
