# インスタンス資材（2026-11-07 回収）

OCI インスタンス「midorun」（ap-osaka-1、Ubuntu 24.04 ARM）の中にだけ残っていた
セットアップ資材を、2026-11-07 に Tailscale SSH 経由で回収したもの。

## 内容

- `systemd/` — ユーザー systemd ユニット（`~/.config/systemd/user/` から）
  - `claude-session.service` — Claude Code 用の常駐 tmux セッション
  - `oss-agent-collect.service` / `oss-agent-collect.timer` — OSS Contributor Agent の日次収集（06:30 JST）
- `claude/` — Claude Code の設定（`~/.claude/` から）
  - `settings.json` — フック設定（Stop / PreToolUse で japanese-guard を実行）
  - `hooks/japanese-guard.py` — 日本語ガード用フックスクリプト

## 回収しなかったもの

- `~/.config/oss-agent/env` と `~/.claude/.credentials.json` — 秘密情報を含むため、GitHub には上げない。
- OSS Contributor Agent の本体ソース — 専用リポジトリ `moruku36/oss-contributor-agent` と
  インスタンス側 git が同期済み（作業ツリー clean、未プッシュコミットなし）であることを確認したため、ここには含めない。
- 実行時成果物（`reports/digest-*.md`、`state/ledger.sqlite3`）— 本体リポジトリの .gitignore の
  方針（runtime state は commit しない）に従い GitHub には上げず、回収したコピーを手元バックアップとして保持する。
