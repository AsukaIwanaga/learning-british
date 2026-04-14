# Everything Claude Code (ECC) 解説

> バージョン: 1.10.0 | Anthropic Hackathon 優勝プロジェクト  
> リポジトリ: [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code)

---

## 概要

**Everything Claude Code (ECC)** は、Claude Code をはじめとする AI エージェントハーネスのパフォーマンス最適化システムです。単なる設定ファイル集ではなく、スキル・本能・メモリ最適化・継続的学習・セキュリティスキャン・リサーチファースト開発を備えた完全なシステムです。

10ヶ月以上の実プロダクト開発での日常利用から生まれた、実戦投入済みのエージェント・スキル・フック・ルール・MCP設定のコレクションです。

**対応ハーネス:** Claude Code / Codex / Cursor / OpenCode / Gemini など

**規模:**
- 47 専門エージェント
- 181 スキル
- 79 スラッシュコマンド
- 14 MCP サーバー設定
- 12 言語エコシステム対応

---

## コア原則

| 原則 | 内容 |
|------|------|
| Agent-First | 専門エージェントへのタスク委譲を優先 |
| Test-Driven | 実装前にテストを書く。カバレッジ 80% 以上必須 |
| Security-First | セキュリティは絶対に妥協しない |
| Immutability | オブジェクトは常に新規作成。既存を変異させない |
| Plan Before Execute | 複雑な機能は実装前に計画を立てる |

---

## プロジェクト構造

```
everything-claude-code/
├── agents/        — 47 の専門サブエージェント定義
├── skills/        — 181 のワークフロースキルとドメイン知識
├── commands/      — 79 のスラッシュコマンド（互換性レイヤー）
├── hooks/         — イベントトリガー型自動化
├── rules/         — 常に遵守するガイドライン（言語別）
├── scripts/       — クロスプラットフォーム Node.js ユーティリティ
├── mcp-configs/   — 14 の MCP サーバー設定
├── tests/         — テストスイート
└── ecc2/          — ECC 2.0 alpha（Rust 製コントロールプレーン）
```

> **重要:** `commands/` は後方互換性のためのレガシー層です。新しいワークフローは `skills/` に追加します。

---

## 1. スキル (Skills)

スキルは ECC の主要なワークフロー実行面です。再利用可能なプロンプト・構造・コードマップを含むスコープ付きワークフローバンドルとして機能します。

**配置場所:** `~/.claude/skills/`

**構造の例:**
```
~/.claude/skills/
  pmx-guidelines.md      # プロジェクト固有パターン
  coding-standards.md    # 言語ベストプラクティス
  tdd-workflow/          # マルチファイルスキル（SKILL.md を含む）
  security-review/       # チェックリスト型スキル
```

**フォーマット:** YAML フロントマター + 以下のセクション構成
- When to Use（使うべき場面）
- How It Works（動作原理）
- Examples（使用例）

---

## 2. エージェント (Agents)

専門化されたサブエージェントに作業を委譲することで、コンテキストを節約しながら高品質な出力を得ます。

### 主要エージェント一覧

| エージェント | 役割 | 使うべき場面 |
|-------------|------|-------------|
| `planner` | 実装計画の立案 | 複雑な機能、リファクタリング |
| `architect` | システム設計・スケーラビリティ | アーキテクチャの意思決定 |
| `tdd-guide` | テスト駆動開発 | 新機能、バグ修正 |
| `code-reviewer` | コード品質・保守性 | 書いたコードの確認 |
| `security-reviewer` | 脆弱性検出 | コミット前、セキュリティ重要コード |
| `build-error-resolver` | ビルド・型エラー修正 | ビルド失敗時 |
| `e2e-runner` | Playwright E2E テスト | 重要なユーザーフロー |
| `refactor-cleaner` | デッドコード削除 | コードメンテナンス |
| `loop-operator` | 自律ループ実行・監視 | ループ安全実行 |
| `harness-optimizer` | ハーネス設定チューニング | 信頼性・コスト・スループット |
| `typescript-reviewer` | TypeScript コードレビュー | TS/JS プロジェクト |
| `python-reviewer` | Python コードレビュー | Python プロジェクト |
| `go-reviewer` | Go コードレビュー | Go プロジェクト |
| `rust-reviewer` | Rust コードレビュー | Rust プロジェクト |
| `java-reviewer` | Java コードレビュー | Java/Spring Boot プロジェクト |
| `kotlin-reviewer` | Kotlin コードレビュー | Kotlin/Android/KMP プロジェクト |
| `cpp-reviewer` | C/C++ コードレビュー | C/C++ プロジェクト |
| `database-reviewer` | PostgreSQL/Supabase 専門 | スキーマ設計・クエリ最適化 |
| `pytorch-build-resolver` | PyTorch/CUDA エラー修正 | 学習・ビルド失敗時 |

### エージェントオーケストレーション

オーケストレーターからエージェントを自動起動するルール：

```
複雑な機能リクエスト → planner
コード記述・修正後 → code-reviewer
バグ修正・新機能 → tdd-guide
アーキテクチャ決定 → architect
セキュリティ重要コード → security-reviewer
自律ループ監視 → loop-operator
```

独立した操作は並列実行（複数エージェントを同時起動）。

---

## 3. フック (Hooks)

フックはツール呼び出しやライフサイクルイベントに反応するトリガー型自動化です。

### フックの種類

| 種別 | タイミング | 用途 |
|------|-----------|------|
| `PreToolUse` | ツール実行前 | バリデーション・リマインダー |
| `PostToolUse` | ツール完了後 | フォーマット・フィードバックループ |
| `UserPromptSubmit` | メッセージ送信時 | 入力の前処理 |
| `Stop` | Claude 応答完了時 | セッション後処理 |
| `PreCompact` | コンテキスト圧縮前 | 状態保存 |
| `Notification` | 権限リクエスト時 | 承認制御 |

### フック設定例（実用的なもの）

```json
{
  "PreToolUse": [
    {
      "matcher": "npm|pnpm|yarn|cargo|pytest",
      "hooks": ["tmux セッション確認リマインダー"]
    }
  ],
  "PostToolUse": [
    {
      "matcher": "Edit && .ts/.tsx/.js/.jsx",
      "hooks": ["prettier --write（自動フォーマット）"]
    },
    {
      "matcher": "Edit && .ts/.tsx",
      "hooks": ["tsc --noEmit（型チェック）"]
    }
  ],
  "Stop": [
    {
      "matcher": "*",
      "hooks": ["console.log が残っていないかチェック"]
    }
  ]
}
```

### フックランタイム制御

```bash
# フックプロファイルの切り替え
ECC_HOOK_PROFILE=minimal    # 最小限
ECC_HOOK_PROFILE=standard   # 標準（デフォルト）
ECC_HOOK_PROFILE=strict     # 厳格

# 特定フックを無効化
ECC_DISABLED_HOOKS=hook-name1,hook-name2
```

---

## 4. ルール (Rules)

`.claude/rules/` 以下の `.md` ファイルで定義する「常に遵守すべき」ガイドライン。言語別にディレクトリが分かれています。

```
rules/
  common/         # 共通ルール
  typescript/     # TypeScript 固有
  python/         # Python 固有
  golang/         # Go 固有
  java/           # Java 固有
  kotlin/         # Kotlin 固有
  cpp/            # C++ 固有
  rust/           # Rust 固有
  php/            # PHP 固有
  perl/           # Perl 固有
```

**主要ルール例:**
```
security.md      # ハードコードされた秘密情報禁止、入力バリデーション必須
coding-style.md  # イミュータビリティ、ファイルサイズ上限（800行）
testing.md       # TDD ワークフロー、80% カバレッジ
git-workflow.md  # Conventional Commits 形式
agents.md        # サブエージェントへの委譲ルール
performance.md   # モデル選択、コンテキスト管理
```

---

## 5. コマンド (Commands) — クイックリファレンス

### コアワークフロー

| コマンド | 機能 |
|---------|------|
| `/plan` | 要件整理・リスク評価・実装計画（コード変更前に確認待ち） |
| `/tdd` | テスト駆動開発の強制（RED→GREEN→REFACTOR） |
| `/code-review` | コード品質・セキュリティ・保守性の完全レビュー |
| `/build-fix` | ビルドエラーの自動検出・修正 |
| `/verify` | ビルド→lint→テスト→型チェックの完全検証ループ |
| `/quality-gate` | プロジェクト基準に対する品質ゲートチェック |
| `/e2e` | Playwright E2E テストの生成・実行 |

### 言語別コードレビュー

| コマンド | 対象 |
|---------|------|
| `/python-review` | Python（PEP 8、型ヒント、セキュリティ） |
| `/go-review` | Go（慣用パターン、並行安全性、エラー処理） |
| `/kotlin-review` | Kotlin（null 安全、コルーチン安全性） |
| `/rust-review` | Rust（所有権、ライフタイム、unsafe 使用） |
| `/cpp-review` | C++（メモリ安全、モダンイディオム） |

### セッション管理

| コマンド | 機能 |
|---------|------|
| `/save-session` | セッション状態を保存 |
| `/resume-session` | 直近セッションの復元 |
| `/sessions` | セッション履歴の確認 |

### ハーネス管理

| コマンド | 機能 |
|---------|------|
| `/harness-audit` | ハーネス設定の監査 |
| `/loop-start` | 自律ループの開始 |
| `/loop-status` | ループ状態の確認 |
| `/model-route` | モデルルーティングの設定 |

### 学習・知識管理

| コマンド | 機能 |
|---------|------|
| `/learn` | セッションからパターンを抽出しスキルへ変換 |
| `/skill-create` | git 履歴からスキルを生成 |
| `/instinct-import` | 本能ファイルのインポート |

---

## 6. MCP 連携

ECC には 14 の MCP サーバー設定が含まれます。外部サービスと Claude を直接接続します。

**推奨 MCP 例:**
```json
{
  "github": "@modelcontextprotocol/server-github",
  "supabase": "@supabase/mcp-server-supabase",
  "memory": "@modelcontextprotocol/server-memory",
  "sequential-thinking": "@modelcontextprotocol/server-sequential-thinking",
  "firecrawl": "firecrawl-mcp",
  "vercel": "https://mcp.vercel.com",
  "railway": "@railway/mcp-server"
}
```

**重要:** MCP は多くても **10 個以下** を有効化。200k コンテキストウィンドウが 70k まで減ることがあります。

---

## 7. セキュリティ (AgentShield)

ECC にはセキュリティスキャナー **AgentShield** が内蔵されています。

- **1,282 テスト**・**102 静的解析ルール**
- `/security-scan` スキルで Claude Code から直接実行
- `--opus` フラグで 3 つの Opus エージェントによるレッドチーム/ブルーチーム/監査パイプライン

**コミット前チェックリスト:**
- ハードコードされた秘密情報（APIキー・パスワード・トークン）がないこと
- 全ユーザー入力がバリデーション済み
- SQL インジェクション対策（パラメータ化クエリ）
- XSS 対策（HTML サニタイズ）
- CSRF 保護が有効
- 認証・認可の検証
- 全エンドポイントにレート制限
- エラーメッセージが機密情報を漏洩しないこと

---

## 8. インストール方法

### 基本インストール

```bash
# リポジトリのクローン
git clone https://github.com/affaan-m/everything-claude-code.git
cd everything-claude-code

# 依存関係のインストール
npm install

# macOS/Linux — フルインストール
./install.sh --profile full

# 特定言語のみインストール
./install.sh typescript python golang

# ターゲットハーネスを指定
./install.sh --target cursor typescript
./install.sh --target gemini --profile full
```

```powershell
# Windows PowerShell
.\install.ps1 --profile full
.\install.ps1 typescript python golang
```

### プラグイン経由のインストール（Claude Code）

```bash
/plugin marketplace add https://github.com/affaan-m/everything-claude-code
/plugin install ecc@ecc
```

### 使い始め

```bash
# プラグイン経由（名前空間付き形式）
/ecc:plan "ユーザー認証を追加"

# 手動インストール後（短い形式）
/plan "ユーザー認証を追加"

# 利用可能なコマンドを確認
/plugin list ecc@ecc
```

---

## 9. ダッシュボード GUI

```bash
npm run dashboard
# または
python3 ./ecc_dashboard.py
```

**機能:**
- タブ UI: エージェント / スキル / コマンド / ルール / 設定
- ダーク/ライトテーマ切り替え
- フォントカスタマイズ
- 全コンポーネントの検索・フィルター

---

## 10. パフォーマンス最適化のヒント

### コンテキストウィンドウ管理

- コンテキストウィンドウ残り **20%** 以下での大規模リファクタリングは避ける
- 未使用の MCP・プラグインは無効化する（有効化は 10 個以下が目安）
- アクティブなツールは 80 個以下に保つ

### 並列実行

```bash
# git worktree で並列 Claude インスタンスを実行
git worktree add ../feature-branch feature-branch
# 各 worktree で独立した Claude インスタンスを起動

# セッション永続化には tmux を使用
tmux new -s dev
```

### キーボードショートカット

| ショートカット | 機能 |
|-------------|------|
| `Ctrl+U` | 行全体を削除 |
| `!` | Bash コマンドの即時実行 |
| `@` | ファイル検索 |
| `/` | スラッシュコマンド起動 |
| `Shift+Enter` | 複数行入力 |
| `Tab` | Thinking 表示の切り替え |
| `Esc Esc` | Claude 中断 / コード復元 |

---

## 11. 開発ワークフロー

ECC の推奨開発サイクル：

```
1. Plan（計画）
   └─ planner エージェントで依存関係・リスクを分析
   └─ 実装をフェーズに分割

2. TDD（テスト駆動開発）
   └─ tdd-guide エージェントでテスト先行実装
   └─ RED → GREEN → REFACTOR のサイクル

3. Review（レビュー）
   └─ code-reviewer エージェントで即座にレビュー
   └─ CRITICAL/HIGH 問題を優先対処

4. Commit（コミット）
   └─ Conventional Commits 形式
   └─ 包括的な PR サマリーを作成
```

**Conventional Commits の型:**
`feat` / `fix` / `refactor` / `docs` / `test` / `chore` / `perf` / `ci`

---

## 12. ECC 2.0 (Alpha)

`ecc2/` ディレクトリに Rust 製コントロールプレーンのプロトタイプが含まれています。

利用可能なコマンド:
```bash
ecc2 dashboard   # ダッシュボード起動
ecc2 start       # セッション開始
ecc2 sessions    # セッション一覧
ecc2 status      # ステータス確認
ecc2 stop        # セッション停止
ecc2 resume      # セッション再開
ecc2 daemon      # デーモンモード
```

> 現時点では Alpha 版です。一般リリースではありません。

---

## まとめ

Everything Claude Code は Claude Code を単なる AI アシスタントから **本格的なソフトウェアエンジニアリング自動化システム** に変える包括的なプラグインです。

| 特徴 | 詳細 |
|------|------|
| 47 の専門エージェント | 言語別・役割別に最適化 |
| 181 のスキル | 再利用可能なワークフロー定義 |
| 自動化フック | イベント駆動の品質管理 |
| セキュリティ内蔵 | 1,282 テストの AgentShield |
| クロスプラットフォーム | Windows / macOS / Linux 対応 |
| マルチハーネス | Claude Code / Cursor / Codex / OpenCode |
