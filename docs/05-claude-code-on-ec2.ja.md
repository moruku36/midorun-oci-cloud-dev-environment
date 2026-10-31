# 05 claude-code-on-ec2：EC2 常駐＋スマホ Remote Control

[English](05-claude-code-on-ec2.md) | 日本語

リポジトリ：https://github.com/minorun365/claude-code-on-ec2（★17、2026-02-25 作成）

AWS EC2 上で Claude Code を常時稼働させ、スマホ・Mac・VSCode から Claude Code の **Remote Control** 機能でどこからでもアクセスできる「パーソナル Claude Code サーバー」を、AWS CDK で一発構築するリポジトリ。

[my-claude-code-settings](03-settings.ja.md) の `remoteControlAtStartup: true` は、この運用と連動している（起動時から Remote Control を有効化）。

## 構成

```
cdk/                 TypeScript の CDK プロジェクト
  lib/claude-on-ec2-stack.ts   (10.7KB、実質の本体)
scripts/
  setup.sh           手動セットアップ版（SSM で接続して実行）
  start-claude.sh    tmux セッション起動（ローカル参考用）
README.md            tmux 操作表・トラブルシューティングまで含む
```

## アーキテクチャ

- **EC2**：`t4g.large`（Graviton2 / ARM64、2vCPU・8GB級）、Amazon Linux 2023（ARM64）、EBS gp3 30GB（暗号化）。デフォルト VPC のパブリックサブネット＋ Elastic IP。
- **常駐の仕組み**：EC2 上で tmux セッション「claude」が生き続け、その中で Claude Code が動く。systemd サービスが起動時に自動で tmux セッションを生成するので、**EC2 を再起動しても Claude Code が自動で立ち上がる**。ログインすると既存セッションへ自動アタッチ。Claude Code を exit すると bash に落ち、別ディレクトリで再起動できるヘルパー関数（`cw <dir>`）も用意されている。
- **スマホからの経路**：Remote Control は HTTPS のアウトバウンド接続で claude.ai 側とつながるため、**EC2 側にポートを開ける必要がない**。スマホでは claude.ai/code を開くとアクティブセッションが一覧表示されるほか、ターミナルの URL や QR コードからも接続できる。有効化は EC2 上の Claude Code の `/config` から。前提として claude.ai の **Pro または Max プラン**が必要。
- **管理接続の経路**：SSH ではなく **SSM Session Manager**。VSCode Remote-SSH も ProxyCommand に SSM を使う形でポート転送なしに接続する。

## セットアップの流れ（CDK の User Data が自動化）

1. ローカルに session-manager-plugin を入れ、`cdk bootstrap` → `cdk deploy`（約5〜10分）。
2. User Data が自動実行：`dnf update`、tmux/git のインストール、Claude Code のネイティブインストール、`.bashrc` への自動アタッチ設定、起動用 entrypoint の配置、systemd サービスの登録・有効化・開始、完了マーカーの作成。
3. SSM で接続 → Claude Code を起動してブラウザ認証 → `/config` で Remote Control を有効化 → スマホから接続。

`scripts/setup.sh` / `start-claude.sh` は同じことを CDK なしで手動で行うための参考スクリプト。

## セキュリティ・データ保護（スタックを読んだ上での所見）

- **インバウンド全閉じ**：セキュリティグループは inbound ルールなし（SSH 22番も開けない）。接続は SSM 経由に寄せている。
- **IAM ロールは広い**：`AmazonSSMManagedInstanceCore` に加えて **AdministratorAccess** が付いている。スタック内のコメントにも「EC2 上から AWS CLI/SDK で操作するため。本番運用時は必要なポリシーに絞ることを推奨」と明記されており、個人用サーバー前提の割り切り。
- **データ保護3点**：EC2 の終了保護、EBS は `deleteOnTermination: false` で terminate 後もデータ保持、AWS Backup で週次スナップショット（30日保持、Backup Vault はスタック削除時も RETAIN）。

## コスト目安（README、us-east-1）

EC2 約$49/月 + EBS 約$2.4 + Backup 約$1〜3 ＝ **合計 約$52〜55/月**。

## 学べること

- 「常時動く自分のエージェント基盤」を、インバウンド全閉じ＋ SSM ＋ Remote Control（アウトバウンドのみ）で安全側に倒して作る構成。
- 再起動耐性（systemd + tmux 自動復帰）とデータ保護（終了保護・EBS 保持・週次バックアップ）まで含めた「個人サーバーの完成形」。
- ただし IAM の AdministratorAccess は個人用途の割り切りであり、業務利用では必ず絞ること。
