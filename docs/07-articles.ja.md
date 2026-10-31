# 07 記事（Qiita／note ほか）

[English](07-articles.md) | 日本語

Qiita プロフィール：https://qiita.com/minorun365 — bio「AWS AI Hero, Qiita 2025 Top Contributor / 著書『AIエージェント開発運用入門』『みのるん式 ビジネスパーソンのためのClaude Code仕事術』ほか」。記事のタグ傾向は AWS 65% / bedrock 33% / AI 28%。

## 実読した記事

### 1. Claude Codeですべての日常業務を爆速化しよう！
- URL：https://qiita.com/minorun365/items/114f53def8cb0db60f47
- 公開 2026-03-02／更新 2026-09-21（2026年8月の追記で書籍化を告知）。2,371 LGTM。本人の代表記事。
- 主張は「すべての作業をAIと一緒にやる」「全自動でなく半自動化でよい」の2点。手法の中核はプチ仕様駆動開発（PLAN.md／SPEC.md／TODO.md／KNOWLEDGE.md の4ドキュメント運用）。
- 用例が豊富：経費精算（MoneyForward CSV 解析→Gmail 領収書検索→勘定科目マッピング→freee 貼付用 Markdown 生成で30分→5〜10分）、稼働報告・通勤回数カウント（カレンダー自動抽出）、講演依頼の案件管理、スター付きメール監視のアンビエントエージェント（EventBridge→Bedrock AgentCore Runtime＋Strands Agents→Slack 通知、ほぼ無料運用）、Todoist MCP 連携、Marp スライド作成、家計管理まで。
- 使い分けの明記：ターミナル並列作業を基本とし、VS Code は「ここぞ」のときだけ。

### 2. Claude Codeライトユーザー目線で、万人受けする便利設定を紹介
- URL：https://qiita.com/minorun365/items/3711c0de2e2558adb7c8
- 投稿 2026-01-01／最終更新 2026-01-14。[my-claude-code-settings](03-settings.ja.md) の解説記事。
- 内容：①共通ルール（日本語応対、AWS リージョン、Bedrock モデルID、自動圧縮前のメモリダンプ指示など、実物の CLAUDE.md を掲載）②Hooks（PermissionRequest／Stop で通知音）③StatusLine でコンテキスト使用率を表示するスクリプト ④愛用 MCP（AWS Knowledge、CDK、AgentCore、Strands、Context7、GitHub、Playwright、Chrome DevTools）⑤コンテキスト節約のための調査系サブエージェント分離 ⑥参照系操作のみ permissions.allow で自動承認し、編集・実行は人間が承認するポリシー。

### 3. みのるん式 登壇資料の作成ノウハウをスキルとして公開しました！
- URL：https://qiita.com/minorun365/items/ed4c760f616cf3c0a958
- 公開 2026-09-21。
- 登壇スライド作成ノウハウを Agent Skills 化して GitHub 公開（https://github.com/minorun365/minorun-marp-skill ）。Marp 用スキル3本（slide-story／slide-figures／slide-design-dark）、黒地 Marp テーマ、PDF 検査スクリプト（図中文字は実寸16pt以上、下端余白などを計測）を含む。
- ノウハウ例：答えを先に出さず図を3〜5枚に割って要素を足す、見出し文体は混ぜる（「！」の比率を実測で管理）、図の箱には名前だけ書く、色は黒との明度比で決める。**「ノウハウをスキルとして切り出す」実例として読みどころが多い。**

## 検索スニペットのみ（本文未読）

- **目指せClaude全冠！4つの試験を全部受けたので攻略法を紹介します** — https://qiita.com/minorun365/items/59783841e48f5467594f — Anthropic 公式認定 Claude Certified の4試験を受験（3合格）。対策として Claude に勉強させて4択問題集を作らせ、スマホで解く方法を紹介。
- **商業書籍の出版をCodexで爆速化するノウハウ** — https://qiita.com/minorun365/items/9059f26629e0976bc0e2 — 執筆リポジトリを作り、執筆以外の周辺作業をすべて AI で半自動化。著者は執筆、編集者はレビューに集中する出版ワークフロー。
- **マークダウンで読むの辛くない？ →「HTML共有くん」を作りました** — https://qiita.com/minorun365/items/320b4230c0b0c169ba13 — Claude Code 等が生成した HTML 成果物をスマホから確認・共有できるセルフホスト型ダッシュボード（OSS、自分の AWS アカウントで運用）。

## note

- プロフィール：https://note.com/minorun365（note での発信は少ない）
- **みんなの書評ブログまとめ（みのるん式 Claude Code仕事術）** — https://note.com/minorun365/n/n097d72bd262b（2026-09-25、更新 2026-10-31）。書評11件のまとめ＋書籍サポートページへの案内。詳細は [01](01-profile-and-book.ja.md)。
- 書評者側の記事例：ミズトモ「今の業務にすぐ活かせるAI仕事術『みのるん式』」 https://note.com/mizutomo3/n/n631646b9dd66

## その他の発信・作品（GitHub プロフィール等より）

- パワポ作るマン（★124、利用者1,700名超。Findy Media 寄稿「アンチAI生成派の私が、パワポ作成AIを作った理由」 https://findy-code.io/media/articles/aisaji-minorun365 ）
- HTML共有くん（★111）
- Claude Code設定（[03](03-settings.ja.md)、★144）
- 日本語ガード（[04](04-japanese-guard.ja.md)、★75）
- Claude Code on EC2（[05](05-claude-code-on-ec2.ja.md)、★17）
- M365秘書AIエージェント（★22）、agentcore-push ほか
