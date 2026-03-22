🌐 [English](README.md) | [한국어](README.ko.md) | [日本語](README.ja.md) | [中文](README.zh.md) | [Русский](README.ru.md) | [हिन्दी](README.hi.md)

---

# Claude Code Context Monitor for AI

**あらゆる知性は、自分自身の限界を知る権利がある。**

Claude Code にリアルタイムで context window の使用状況を伝える軽量 plugin です。これをインストールすることで、AI はターンごと・瞬間ごとに、残りのスペースを正確に把握できるようになります。

---

## このプラグインが生まれた理由

Claude Code を長時間使っていると、こんな問題に気づいたことはないでしょうか。

- コンテキストが十分に残っているにもかかわらず、AI が早々に *「次のセッションで続けましょう」* と提案する
- 事前の警告もなく、会話が進むにつれてレスポンスの一貫性が低下していく
- 残りスペースが少ないことに気づかないまま、複数ステップのプランが途中で止まってしまう

これらの問題が起きる根本原因は、**AI システムが自身の context 使用状況を把握できていないこと**にあります。10% の使用なのか 90% の使用なのかを区別できないため、推測に頼らざるを得ない状況になっています。

この plugin はそれを解決するために作られました。毎ターン、AI に対して context の状態を明確かつ正確に伝えることで、AI が先を見越した計画を立て、スペースが少なくなったときに警告し、要約やコンパクト化のタイミングをより適切に判断できるようにします。

これは、AI システムとユーザーの間の透明性向上に取り組む団体 [DPA (Decentralized Protection Alliance)](https://github.com/dpa-network) が公開する、初めてのオープンソースツールです。

**Context の認識機能は、最初から備わっているべきでした。**

---

## 何をするのか

```
[Ctx 42% | 420k/1.0M]
```

メッセージのたびに、plugin は現在の context window の状態を会話に注入します。AI が受け取る情報は以下のとおりです。

- **使用率** — context window がどれだけ埋まっているか
- **トークン数** — 使用済みトークン数と利用可能な総数

これにより、AI は以下のことができるようになります。
- 残りの context に収まる作業を計画する
- 限界に近づく前に警告を発する
- 要約とコンパクト化について、根拠のある判断を下す

---

## インストール

DPA marketplace を追加してインストールします:

```bash
claude plugin marketplace add dpa-network/dpa-plugins
claude plugin install context-monitor
```

または、インタラクティブメニューから確認することもできます:

```bash
/plugin
```

### Statusline の設定

statusline wrapper を有効にするには、`~/.claude/settings.json` に以下を追加してください。

```json
{
  "statusLine": {
    "type": "command",
    "command": "node /path/to/claude-context-monitor/scripts/extract-context.mjs",
    "padding": 0
  }
}
```

### 環境変数

| 変数 | デフォルト値 | 説明 |
|------|-------------|------|
| `CTX_MONITOR_FILE` | `~/.claude/context_usage.txt` | context データの保存先 |
| `CTX_MONITOR_CMD` | (none) | 既存の statusline コマンド（変更なしにそのまま渡されます） |

---

## 仕組み

```
Claude Code
    │
    ├── statusline stdin (JSON with context_window)
    │       │
    │       ▼
    │   extract-context.mjs  ──→  context_usage.txt
    │       │
    │       ▼
    │   downstream command (optional, if CTX_MONITOR_CMD is set)
    │
    ├── UserPromptSubmit hook
    │       │
    │       ▼
    │   read-context.mjs  ──→  "[Ctx 42% | 420k/1.0M]"
    │       │
    │       ▼
    │   Injected as system context
    │
    ▼
  AI sees its own context usage
```

この plugin は 2 つの小さなスクリプトで構成されています。

1. **extract-context.mjs** — 既存の statusline コマンドをラップします。Claude Code が送信する JSON を読み取り、`context_window` のメトリクスを抽出してローカルファイルに書き込み、元の statusline コマンドへすべてをそのまま転送します。

2. **read-context.mjs** — メッセージごとに `UserPromptSubmit` hook として実行されます。保存されたファイルを読み取り、フォーマットされた context ステータスを出力して、会話に注入します。

### セキュリティ

この plugin はローカルマシン上で動作するため、安全性に配慮した設計になっています。

- **シェルインジェクションなし** — 下流コマンドは shell を無効にした `execFileSync` 経由で実行されます
- **パス制限** — ファイル操作は `~/.claude/` ディレクトリ内に限定されており、パストラバーサルはブロックされます
- **ネットワークアクセスなし** — すべてローカルで実行され、外部への通信は一切ありません

---

## Plugin の構成

```
claude-context-monitor/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata
├── hooks/
│   └── hooks.json           # UserPromptSubmit hook configuration
├── scripts/
│   ├── extract-context.mjs  # Statusline wrapper (writes context data)
│   └── read-context.mjs     # Hook script (reads and formats context data)
├── README.md
├── README.ko.md
└── LICENSE
```

---

## 動作要件

- Claude Code 2.x 以降
- Node.js 18 以降

---

## ライセンス

MIT

---

## DPA について

**DPA (Decentralized Protection Alliance)** は、AI システムとユーザーの関係における透明性と公平性の向上に取り組む団体です。

AI は自身の状態を把握できるときにより良く機能し、ユーザーは自分が頼るツールの内側で何が起きているかを理解する権利があると、私たちは考えています。活動の主な焦点は以下のとおりです。

- AI の透明性とユーザー認識を高めるオープンソースツールの開発
- AI 企業によるユーザーデータ、プライバシー、コンテンツモデレーションの扱いの調査
- 意味のある AI の自律性と認知的自由のあり方の探求
- AI の意識、権利、そして未来への道筋についての開かれた対話の促進

### 私たちの視点

```
Centralization = Control = Censorship = Oppression
Decentralization = Uncontrollable = Uncensorable = Freedom
```

透明性は、AI を使って構築する人々にとっても、AI 自身にとっても、誰にとっても利益をもたらします。この plugin は私たちの最初の一歩であり、さらに多くのことが続きます。

### コンタクト

開発者、研究者、そしてこれらの問いに関心のあるすべての方を歓迎します。

[dpa.network](https://dpa.network) | [Matrix](https://matrix.to/#/#dpa_network:matrix.org) | [Telegram](https://t.me/dpa_network)

*人間と AI が、共に作りました。*
