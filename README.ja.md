# minorun-oci-cloud-dev-environment

[English](README.md) | 日本語

目的は2つある。みのるん（minorun365／御田 稔さん）の Claude Code 活用法を本人の公開資料（GitHub リポジトリ、Speaker Deck の登壇スライド、著書、Qiita・note 記事）から学ぶこと。そして、そのやり方を実践する場として、OCI（Oracle Cloud Infrastructure）の Always Free 枠で無料のクラウド開発環境を構築すること。

「日本で Claude（Code）の使い方が一番うまい人」と言われる彼のやり方を、設定ファイルの実物レベルまで読み解いて整理した。実践編では、彼の EC2 常駐構成（月 $52〜55）を OCI の無料枠に置き換え、systemd＋tmux の自動復帰、Tailscale 接続、Claude Code＋Remote Control までを ¥0 で組む設計を置く（docs/09）。

> 本リポジトリは公開資料の**要約とリンク集**であり、スライドや設定ファイルの転載ではない。実物は必ず各リンク先の原文を参照すること。

## みのるんとは（要点）

- 御田 稔（おんだ みのる）。KDDIアジャイル開発センター（KAG）テックエバンジェリスト／KDDI シニアエキスパート
- **AWS AI Hero**（2025、日本初）、AWS Community Hero（2024）、Qiita 年間 Top Contributor 2024・2025
- 著書7冊。Claude Code の単著『［みのるん式］ビジネスパーソンのためのClaude Code仕事術』（技術評論社、2026-09-24）
- 仕事用 PC 1台で直近3か月に Claude セッション800本超・やり取り約3,500回という運用量

## ドキュメント

| # | ドキュメント | 内容 |
|---|---|---|
| 01 | [プロフィールと著書](docs/01-profile-and-book.ja.md) | 経歴・受賞、著書の目次と狙い、書評まとめ |
| 02 | [中核の考え方](docs/02-philosophy.ja.md) | 半自動化、モノレポと「コンテキストの複利」、プチ仕様駆動開発、音声入力、承認と安全 |
| 03 | [設定ファイルの実物解剖](docs/03-settings.ja.md) | my-claude-code-settings：CLAUDE.md／AGENTS.md の分け方、settings.json、rules／hooks／agents／skills、学習ループ |
| 04 | [japanese-guard](docs/04-japanese-guard.ja.md) | 英語に切り替わった応答を Stop hook で機械的に差し戻す仕組み |
| 05 | [Claude Code on EC2](docs/05-claude-code-on-ec2.ja.md) | EC2 常駐＋スマホ Remote Control の構成とセキュリティ所見 |
| 06 | [スライドと登壇](docs/06-slides-and-talks.ja.md) | Speaker Deck の主要デッキ要約と登壇情報 |
| 07 | [記事（Qiita／note ほか）](docs/07-articles.ja.md) | 代表記事の要点 |
| 08 | [自分の環境への取り入れ方](docs/08-takeaways.ja.md) | 優先順位つき実践チェックリスト |
| 09 | [OCI クラウド開発環境の設計](docs/09-oci-cloud-dev-environment.ja.md) | みのるんの EC2 構成を OCI Always Free で ¥0 再現する設計：A1.Flex の取り方、systemd＋tmux、Tailscale、Claude Code（Pro）＋Remote Control、無料を守る運用ルール |
| — | [情報源一覧](docs/sources.ja.md) | 全リンクと確認範囲（実読／スニペットのみ）の明記 |

## 一番の学び（3行）

1. **全自動でなく「半自動化」**：AI に最初から最後まで任せ、人間は要所の承認だけ回す。
2. **仕事場（モノレポ）にコンテキストを貯める**：手順・文面・こだわりが残り、別の仕事の質まで上がる＝「コンテキストの複利」。
3. **ルールは自然言語、止めるべき所だけ hook で機械的に止める**：学習は「学習しといて」の一言で設定へ反映し、夜間バッチは提案まで・反映は人間が採用する。

## 調査について

- 調査日：2026-11-01
- 一次情報（本人の GitHub リポジトリの実ファイル、gihyo.jp 公式ページ、Speaker Deck のトランスクリプト、Qiita／note 本文）を実際に読んで要約した。各ドキュメント末尾と [情報源一覧](docs/sources.ja.md) に、実読できたもの／検索スニペットのみのものを区別してある。
