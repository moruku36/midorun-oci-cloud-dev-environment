# 構築資材（scripts/）

midorun（OCI クラウド開発環境）の構築に使ったスクリプトと設定を、再現性のために集約するフォルダ。再現手順そのものは [docs/09](../docs/09-oci-cloud-dev-environment.ja.md)（設計と手順）、詰まった所は [docs/10](../docs/10-oci-pitfalls.ja.md) に対応する。

Build assets for the midorun OCI cloud development environment, collected for reproducibility. The step-by-step procedure lives in docs/09; pitfalls in docs/10.

## 収録方針

- メールアドレス、テナンシ／インスタンスの OCID、通知トピックの OCID などの環境固有値は、実値を書かずプレースホルダ化する（設定ファイルは `.example` 形式）。
- API キー・トークン・パスワード類は含めない。
- 他リポジトリに本体があるものはコードを複製せず、リンクで参照する。

## 資材一覧

| 資材 | インスタンス上の配置先 | 用途 | 状態 |
|---|---|---|---|
| cost-guard.py | `/usr/local/bin/cost-guard.py` | インスタンスプリンシパルで Usage API を読み、月額が閾値 ¥750 を超えたらメール通知＋全インスタンス停止 | 回収待ち（インスタンス内のみ存在） |
| cost-guard.conf.example | `/etc/cost-guard.conf` | cost-guard の設定。実値はプレースホルダ化して収録する | 回収待ち（インスタンス内のみ存在） |
| cost-guard.service / cost-guard.timer | `/etc/systemd/system/` | cost-guard を30分間隔で実行する systemd ユニット | 回収待ち（インスタンス内のみ存在） |
| claude-session.service | systemd ユーザーユニット | tmux セッション「claude」を常駐生成し Claude Code を常駐させる（linger 有効が前提） | 回収待ち（インスタンス内のみ存在） |
| japanese-guard フック本体 | `~/.claude/hooks/japanese-guard.py` | 英語応答を機械的に差し戻す Stop フック | 外部参照（下記） |
| japanese-guard の登録設定 | `~/.claude/settings.json` | フックの Stop／PreToolUse 登録と `remoteControlAtStartup: true` | 回収待ち（インスタンス内のみ存在） |

## 外部参照（コードは複製しない）

- **japanese-guard**（フック本体）：みのるん（minorun365）の OSS。導入時はソースを精査し、精査したソースとバイト単位で一致することを確認して配置した。 https://github.com/minorun365/claude-code-japanese-guard （解説は [docs/04](../docs/04-japanese-guard.ja.md)）
- **oss-contributor-agent**（毎日 06:30 の収集ジョブ本体）：別リポジトリに本体がある。 https://github.com/moruku36/oss-contributor-agent

## 回収待ちについて

「回収待ち」の資材は midorun インスタンス内にのみ存在し、ローカルに作成元の控えが残っていない。インスタンスは通常停止しているため、回収には起動が必要である（起動はオーナーの承認のもとで別途行う）。回収でき次第、このフォルダへ収録し、上表の状態を更新する。
