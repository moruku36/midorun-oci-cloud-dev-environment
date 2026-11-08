# 構築資材の索引 / Build asset index

更新：2026-11-08。実物の回収記録は [instance-assets/README.ja.md](../instance-assets/README.ja.md)（2026-11-07）。この `scripts/` は索引であり、すべてのスクリプトが収録済みという意味ではない。

Updated 2026-11-08. This directory is an index, not a complete set of rebuild scripts. Recovered files live in [instance-assets/](../instance-assets/README.ja.md). OCI-specific cost-guard assets remain pending.

- OCIの設計・構築記録：[09](../docs/09-oci-cloud-dev-environment.ja.md)
- 構築時の詰まり所：[10](../docs/10-oci-pitfalls.ja.md)
- Mac miniへの移植計画：[11](../docs/11-mac-mini-dev-environment-plan.ja.md)（未構築）

## 収録済み（2026-11-07回収）

| 資材 | 元の配置・用途 | 状態と注意 |
|---|---|---|
| [claude-session.service](../instance-assets/systemd/claude-session.service) | ユーザーsystemd、tmuxセッション生成 | 回収済み。空tmuxを作るだけで、Claude起動や異常終了後の再起動は行わない |
| [oss-agent-collect.service](../instance-assets/systemd/oss-agent-collect.service) | OSS Contributor Agentの単発収集 | 回収済み。EnvironmentFileの秘密は収録しない |
| [oss-agent-collect.timer](../instance-assets/systemd/oss-agent-collect.timer) | 日次06:30の収集タイマー | 回収済み。ゲストのtimezoneに依存。Persistent=trueの追い付き実行に注意 |
| [Claude設定](../instance-assets/claude/settings.json) | Stop／PreToolUse、Remote Control、Sonnet | 回収済み。別環境ではユーザー固有パスを調整し、既存設定を上書きしない |
| [japanese-guard.py](../instance-assets/claude/hooks/japanese-guard.py) | 日本語ガードの回収コピー | 回収済み。原典は[上流OSS](https://github.com/minorun365/claude-code-japanese-guard)。更新時は差分を確認 |

## 未回収・外部参照

| 資材 | 用途 | 状態 |
|---|---|---|
| cost-guard.py | OCI Usage APIを読み、月額750円超過で通知＋稼働中インスタンス停止 | 未回収。インスタンス内のみ存在するという記録 |
| cost-guard.conf.example | OCIガードの設定を秘密・環境固有値なしで記録する予定 | 未収録。実設定のOCID等をそのまま公開しない |
| cost-guard.service / cost-guard.timer | 30分間隔のOCIガード | 未回収。Mac向けには移植しない |
| [oss-contributor-agent](https://github.com/moruku36/oss-contributor-agent) | 収集・トリアージ本体 | 別repoを参照。本索引は現時点の稼働・認証・通知の受入れを示さない |

未回収資材を取得するためのOCI起動や接続は今回行っていない。回収時はその時点の許可を確認し、秘密を除いた内容だけを収録する。回収済みファイルの存在を、現行インスタンスの正常稼働や完全な再構築可能性の証明にしない。

## 公開・移植時のルール

- APIキー、トークン、パスワード、Claude資格情報、OSS Agentのenv、実行時DB・レポートは含めない。
- メール、実IP、OCID、私的パス等の環境固有値を新規資料に出さない。必要ならプレースホルダ化する。
- 回収物は元環境の記録として保存する。移植用変更は別ファイル／変更履歴で追跡する。
- unitの配置、enable、タイマー実行、ログイン、ネットワーク設定変更は別の実施作業。文書のmergeだけで実行しない。
