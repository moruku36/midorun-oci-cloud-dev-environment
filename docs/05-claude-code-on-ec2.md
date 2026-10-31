# 05 claude-code-on-ec2: always-on EC2 + smartphone Remote Control

English | [日本語](05-claude-code-on-ec2.ja.md)

Repository: https://github.com/minorun365/claude-code-on-ec2 (★17, created Feb 25, 2026)

A one-shot AWS CDK deployment of a "personal Claude Code server": Claude Code runs persistently on EC2, reachable from a phone, a Mac or VS Code through Claude Code's **Remote Control** feature.

The `remoteControlAtStartup: true` in [my-claude-code-settings](03-settings.md) belongs to this setup (Remote Control on from process start).

## Layout

```
cdk/                 TypeScript CDK project
  lib/claude-on-ec2-stack.ts   (10.7 KB — effectively the whole system)
scripts/
  setup.sh           manual setup variant (run over SSM)
  start-claude.sh    tmux session launcher (local reference)
README.md            includes a tmux cheat sheet and troubleshooting table
```

## Architecture

- **EC2**: `t4g.large` (Graviton2 / ARM64, 2 vCPU, ~8 GB), Amazon Linux 2023 (ARM64), 30 GB gp3 EBS (encrypted), in the default VPC's public subnet with an Elastic IP.
- **Persistence**: a tmux session named "claude" stays alive with Claude Code inside it. A systemd service regenerates the tmux session at boot, so **Claude Code comes back automatically after a reboot**. Logging in auto-attaches to the session; exiting Claude Code drops to bash, and a helper (`cw <dir>`) relaunches it in another directory.
- **Phone path**: Remote Control connects outbound over HTTPS to claude.ai, so **no inbound ports are opened on the EC2 side**. On the phone, claude.ai/code lists active sessions; terminal URLs and QR codes also connect. Enabled via `/config` inside Claude Code on the EC2 host. Requires a claude.ai **Pro or Max plan**.
- **Admin path**: SSM Session Manager, not SSH. VS Code Remote-SSH also rides SSM via a ProxyCommand, with no port forwarding.

## Setup flow (automated by the CDK User Data)

1. Install the session-manager-plugin locally; `cdk bootstrap` → `cdk deploy` (~5–10 minutes).
2. User Data runs automatically: `dnf update`, installs tmux/git, native Claude Code install, `.bashrc` auto-attach, the entrypoint script, systemd service registration/enable/start, and a completion marker.
3. Connect via SSM → launch Claude Code and sign in with browser auth → enable Remote Control in `/config` → connect from the phone.

`scripts/setup.sh` / `start-claude.sh` reproduce the same result manually without CDK.

## Security and data protection (observations from reading the stack)

- **Inbound fully closed**: the security group has no inbound rules at all (not even SSH/22). All access rides SSM.
- **The IAM role is broad**: `AmazonSSMManagedInstanceCore` plus **AdministratorAccess**. A comment in the stack itself says this is so the AWS CLI/SDK can be used from the instance and recommends scoping it down for production — a deliberate personal-server trade-off.
- **Three data protections**: EC2 termination protection; EBS `deleteOnTermination: false` so data survives termination; AWS Backup weekly snapshots (30-day retention, vault retained even if the stack is deleted).

## Cost estimate (README, us-east-1)

EC2 ~$49/mo + EBS ~$2.4 + Backup ~$1–3 = **~$52–55/month total**.

## What to learn from it

- A reference architecture for "my own always-on agent box" that leans safe: inbound closed, SSM for admin, outbound-only Remote Control.
- Reboot resilience (systemd + tmux auto-recovery) and data protection (termination protection, retained EBS, weekly backups) are part of the design, not afterthoughts.
- The AdministratorAccess role is a personal-use shortcut — scope it down for anything work-related.
