# minorun-oci-cloud-dev-environment

[English](README.md) | 日本語

みのるんの Claude Code 活用法を本人の公開資料から学び、OCI（Oracle Cloud Infrastructure）の Always Free 枠で無料のクラウド開発環境を構築する実践リポジトリ。

## 目的

目的は3つある。正確には2.5個で、3つ目は2つ目の動機に当たる。

1. **学ぶこと**：みのるん（minorun365／御田 稔さん）の Claude Code 活用法を、本人の公開資料（GitHub リポジトリ、Speaker Deck の登壇スライド、著書、Qiita・note 記事）から学ぶ。
2. **実践すること**：そのやり方を実践する場として、OCI の Always Free 枠で無料のクラウド開発環境を構築する。
3. **2つ目の動機**：ローカルの Windows 環境に依存しない開発の場を持つこと。作業中の内容が Windows Update の再起動でリセットされるといった環境固有の問題を避けるため、もう1つの開発環境をクラウドに置く。

「日本で Claude（Code）の使い方が一番うまい人」と言われる彼のやり方を、設定ファイルの実物レベルまで読み解いて整理した。実践編では、彼の EC2 常駐構成（月 $52〜55）を OCI の無料枠に置き換え、systemd＋tmux の自動復帰、Tailscale 接続、Claude Code＋Remote Control までを ¥0 で組む設計を置く（docs/09）。

> 本リポジトリは公開資料の**要約とリンク集**であり、スライドや設定ファイルの転載ではない。実物は必ず各リンク先の原文を参照すること。

## みのるんとは（要点）

- 御田 稔（おんだ みのる）。KDDIアジャイル開発センター（KAG）テックエバンジェリスト／KDDI シニアエキスパート
- **AWS AI Hero**（2025、日本初）、AWS Community Hero（2024）、Qiita 年間 Top Contributor 2024・2025
- 著書7冊。Claude Code の単著『［みのるん式］ビジネスパーソンのためのClaude Code仕事術』（技術評論社、2026-09-24）
- 仕事用 PC 1台で直近3か月に Claude セッション800本超・やり取り約3,500回という運用量

## 構築した開発環境の構成（2026-11-03 時点）

みのるんの EC2 常駐構成を OCI で再現し、実際に構築した現在の構成である。設計と実績の詳細は [docs/09](docs/09-oci-cloud-dev-environment.ja.md)、構築で詰まった所は [docs/10](docs/10-oci-pitfalls.ja.md) にある。2026-11-01 の構築当初から、シェイプの縮小（4 OCPU／24GB → 1 OCPU／6GB）と通貨単位の修正（予算・閾値 USD 5 → ¥750）に伴い変更されている。

```mermaid
flowchart TB
    User["moruku（利用者）<br/>SSH 接続と Claude Code Remote Control"]

    subgraph Tenancy["OCI テナンシー（構築先リージョン：大阪 ap-osaka-1）"]
        Budget["OCI Budget「guardrail-monthly」<br/>月 ¥750・実績／予測アラートをメール通知"]
        subgraph Instance["インスタンス「midorun」<br/>VM.Standard.A1.Flex（ARM）1 OCPU / 6 GB<br/>Ubuntu 24.04 Minimal／ブートボリューム 50GB"]
            Claude["Claude Code（Claude Pro、既定モデル Sonnet）<br/>tmux セッションで常駐・Remote Control 有効<br/>japanese-guard フック導入済み"]
            Guard["cost-guard（systemd タイマー・30分間隔）<br/>インスタンスプリンシパルで Usage API を参照<br/>月額が閾値 ¥750 を超えたらメール通知＋全インスタンス停止"]
            Oss["oss-contributor-agent（venv）<br/>systemd タイマー毎日 06:30（インスタンス稼働中のみ）<br/>good first issue を収集し Meta Model API でトリアージ"]
        end
    end

    User -->|SSH：公開IP 129.225.187.56| Instance
    User -->|SSH：Tailscale 100.75.1.65| Instance
    User -.->|Remote Control| Claude
    Budget -.->|メールアラート| User
    Guard -.->|メール通知（Notifications トピック経由）| User
```

| 項目 | 構成 |
|---|---|
| リージョン | 構築先は大阪（ap-osaka-1）。ホームリージョンは東京（ap-tokyo-1） |
| インスタンス | 「midorun」：VM.Standard.A1.Flex（ARM）、1 OCPU／6GB、ブートボリューム 50GB（2026-11-02 にホスト容量不足のため 4 OCPU／24GB から縮小。docs/10 No.14） |
| OS | Ubuntu 24.04 Minimal（Arm、aarch64） |
| 常駐の仕組み | systemd ユーザーユニット＋linger で tmux セッション「claude」を保持し、ログインしなくても Claude Code が常駐する。Remote Control で外から操作できる。Claude Code は Claude Pro の claude.ai ログインで使い、API 課金は発生しない |
| 日本語ガード | japanese-guard フック（みのるんの OSS）を、ソースを精査した上で導入（英語応答を機械的に差し戻す） |
| 既定モデル | Claude Code の既定モデルは Sonnet（オーナー指示で設定） |
| アクセス | SSH（ユーザー ubuntu、鍵認証のみ・パスワード認証は無効）の2経路：公開IP 129.225.187.56／Tailscale 100.75.1.65。あわせて Claude Code の Remote Control |
| 課金ガードレール | 二重化している。①OCI Budget「guardrail-monthly」（月 ¥750、実績と予測のアラートをメール通知）②箱側の自動停止 cost-guard（systemd タイマー30分間隔、インスタンスプリンシパルで Usage API を読み、月額が閾値 ¥750 を超えたらメール通知＋稼働中インスタンスを全停止） |
| その他の常駐ジョブ | oss-contributor-agent の収集ジョブ（venv、systemd タイマー毎日 06:30、インスタンス稼働中のみ動作）。対象リポジトリの good first issue を収集し、Meta Model API でトリアージする |
| 運用方針 | 使う時だけ起動し、終わったら停止する。現在は停止中 |

## 構築資材（scripts/）

構築に使ったスクリプトと設定は [scripts/](scripts/README.md) に集約している。将来、別クラウドで同じ環境を作り直せるようにするためである。メールアドレスや OCID などの環境固有値はプレースホルダ化してある。

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
| 10 | [OCI 困った集](docs/10-oci-pitfalls.ja.md) | AWS／Azure／GCP 経験者が OCI 初構築で詰まる所を随時追記する生きた記録：テナンシー名サインイン、無料枠の半減、用語対応表、在庫不足ほか |
| — | [情報源一覧](docs/sources.ja.md) | 全リンクと確認範囲（実読／スニペットのみ）の明記 |

## 一番の学び（3行）

1. **全自動でなく「半自動化」**：AI に最初から最後まで任せ、人間は要所の承認だけ回す。
2. **仕事場（モノレポ）にコンテキストを貯める**：手順・文面・こだわりが残り、別の仕事の質まで上がる＝「コンテキストの複利」。
3. **ルールは自然言語、止めるべき所だけ hook で機械的に止める**：学習は「学習しといて」の一言で設定へ反映し、夜間バッチは提案まで・反映は人間が採用する。

## 調査について

- 調査日：2026-11-01
- 一次情報（本人の GitHub リポジトリの実ファイル、gihyo.jp 公式ページ、Speaker Deck のトランスクリプト、Qiita／note 本文）を実際に読んで要約した。各ドキュメント末尾と [情報源一覧](docs/sources.ja.md) に、実読できたもの／検索スニペットのみのものを区別してある。
