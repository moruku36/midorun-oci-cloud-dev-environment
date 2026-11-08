# 11 Plan: reproduce the OCI development environment on Mac mini

English | [日本語・詳細計画](11-mac-mini-dev-environment-plan.ja.md)

Recorded: 2026-11-08. Status: **planned; implementation has not started**.

## 1. Goal and scope

Build a second development environment on the home lab's Mac mini so work can continue without OCI A1 capacity availability in Tokyo or Osaka, and independently of Windows reboots. The preferred design is a **dedicated Ubuntu 24.04 ARM64 VM**, retaining tmux, systemd, Claude Code, japanese-guard and Remote Control.

Keep OCI intact while accepting one small representative repository on the Mac. Equivalence means the development workflow and reproducible configuration, not identical availability, IAM, billing controls or power resilience.

This update performed public-source research, read-only Mac inspection and documentation changes only. It did not create a VM, install software, start or stop services, sign in, or change network, permissions or power settings. Before implementation, reconcile the owner's then-current instructions and existing permissions, and request only missing approvals.

## 2. Evidence and current baseline

### OCI: recorded state and recovered files, not a new live inspection

Reviewed main at [f2ac8fca](https://github.com/moruku36/midorun-oci-cloud-dev-environment/commit/f2ac8fca0a5baa5d1267ed5e8a1e9f93b87b38c6) on 2026-11-08. The instance and console were not accessed or started.

- [09](09-oci-cloud-dev-environment.md) records Osaka A1.Flex, 1 OCPU / 6 GB, 50 GB boot volume, Ubuntu 24.04 Minimal ARM64. The instance was reduced from 4 OCPU / 24 GB on November 2 because of capacity shortages.
- Recorded tools include tmux, git, Python, Node.js and Claude Code. Current installed versions were not rechecked.
- Access includes Tailscale, a historical public-SSH exception and Remote Control. [10](10-oci-pitfalls.md) records why reachability must be tested separately for each operator.
- [November 7 assets](../instance-assets/README.ja.md) contain three user-systemd files, Claude settings and the Japanese hook. Credentials and runtime data are excluded. The cost-guard script and configuration remain unrecovered.
- The optional [OSS Contributor Agent](https://github.com/moruku36/oss-contributor-agent) collector has a daily timer. Its credentials, cost, notifications and current state require separate reconciliation.
- OCI's JPY 750 budget and Usage-API shutdown guard stay in OCI. Current instance power state was not checked; historical “stopped” text is not a live observation.

**Important source finding:** the recovered [claude-session.service](../instance-assets/systemd/claude-session.service) creates an empty tmux session; it does not launch Claude Code. With RemainAfterExit=yes it also does not monitor or restart tmux or Claude after a crash. Start Claude manually in the initial trial. A launch wrapper and bounded restart policy, if needed, are a separate change. Do not infer automatic recovery from the older “resident” description.

### Mac: read-only observations on 2026-11-08 UTC

| Area | Observed | Limit |
|---|---|---|
| Hardware | Apple M4, 10 CPU cores (4 performance + 6 efficiency), arm64, 16 GB RAM | system_profiler and uname |
| OS | macOS 27.2, build 26B123 | sw_vers; VM-product compatibility still to be checked |
| Storage | Approximately 228 GiB APFS capacity, 142 GiB available | df; recheck immediately before implementation |
| Software | Docker Desktop and Tailscale applications; docker and kubectl on PATH | Presence does not establish running state or connectivity |
| Docker disk | Approximately 3.6 GB actually allocated to its sparse disk | A sparse maximum is not disk usage or free capacity |

Inspection did not require unlocking the Mac. Permission limits prevented reading Docker runtime/configuration and the process list. Container state, CPU/RAM limits, networks, installed VM managers and current Tailscale reachability remain unverified. Permissions were not changed or bypassed.

Public home-lab records provide dated context:

- [November 1 kitting record](https://github.com/moruku36/home-ai-engineering-lab/blob/main/docs/mac-mini-kitting-record-2026-11.md): purchased M4 / 16 GB / 256 GB machine, historical SSH/Tailscale success, and unresolved FileVault, power recovery, key-authentication and firewall items. Do not treat these as November 8 observations.
- [November 6 OpenClaw record](https://github.com/moruku36/home-ai-engineering-lab/blob/main/docs/openclaw-status-2026-11-06.ja.md): limited installation/browser/CLI checks, not full operational acceptance or credential isolation.
- [Operations design](https://github.com/moruku36/home-ai-engineering-lab/blob/main/docs/home-ai-lab-openclaw-operations.md): Dots owns construction and changes; OpenClaw operations are gated. The new VM is not automatically handed over.

Existing NUC services and the isolated security-analysis guest stay unchanged. This development VM is not an accepted sandbox for hostile code.

## 3. Preferred design and sizing

| Option | Benefit | Trade-off |
|---|---|---|
| **Dedicated Ubuntu ARM64 VM** | Retains Linux/systemd/apt/venv and separates host CLI configuration | Additional memory/storage; host, VM and guest startup need separate checks |
| Native macOS | No additional VM RAM | Translate systemd to launchd; GNU/BSD, path and permission differences |
| Existing Docker Desktop container | May reuse a runtime | An ordinary container is not equivalent to systemd + linger; socket, mounts and credential sharing need care |

**Lima with Apple Virtualization.framework (VZ) is a candidate, not an installed or accepted product.** [Lima's VM guide](https://lima-vm.io/docs/config/vmtype/) recommends VZ for native-architecture guests on macOS hosts. Pin the supported Lima version, macOS compatibility, official Ubuntu ARM64 image and checksum during implementation. Compare any existing VM platform before adding another.

Proposed starting budget: one VM, 2 vCPU, **4 GiB RAM initially**, 40 GiB virtual disk maximum. This is a low-load acceptance configuration, not the same memory capacity as OCI's 6 GB. Consider 6 GiB only after measuring existing Docker and other workloads. Do not assume CPU or build performance equivalence.

The host's 16 GB is shared with macOS, Docker, OpenClaw and local inference. Avoid concurrent heavy inference/builds in the first trial; do not stop existing services without permission. Budget both VM and backup space, preserving at least 50 GiB host free space. Record virtual maximum, actual allocation and APFS availability separately.

Provisional trial stop conditions: sustained yellow/red memory pressure, swap growth above 512 MiB in 10 minutes, host free space below 50 GiB, or increased errors in existing services. Stop the new VM trial and reassess. These are proposed limits, not measured performance results.

## 4. Access, data and permissions

Initial route: approved client → existing approved Mac tailnet route → local management SSH into the guest. [Lima's network guide](https://lima-vm.io/docs/config/network/) distinguishes localhost access from bridged access. Do not add public SSH, router forwards, a bridge or subnet routes. Inspect automatic forwarding and explicitly limit application exposure to approved loopback listeners.

Direct guest Tailscale membership is optional later. Review device enrollment, ACLs, keys and costs, obtaining any missing permissions first. Never copy host Tailscale credentials. Verify required routes separately from the owner's Windows device, phone and construction operator. Keep an unreachable path pending rather than opening a public port as a workaround.

[Remote Control](https://code.claude.com/docs/en/remote-control) uses outbound HTTPS. Its messages and tool activity pass through Anthropic; local execution does not mean no external transmission. Use public repositories and synthetic inputs for acceptance. Verify current plan eligibility, CLI version, account policies and connectivity.

- Store projects and venvs in the guest. Explicitly disable unneeded [host mounts](https://lima-vm.io/docs/config/mount/), and verify guest visibility. Do not share the host home, existing Claude state, SSH keys, Docker socket or browser profiles.
- Use a non-root development account and document required sudo and host/guest administration boundaries. A VM is not a guarantee of complete isolation.
- Publish only architecture, redacted acceptance summaries and remaining questions, excluding real addresses, device identifiers, private paths, admin URLs, raw logs and credentials.
- Keep direct official Claude CLI subscription login. Treat guest authentication as a new authorized login, not a credentials copy. Do not pass subscription tokens to OpenClaw or automatically switch to API billing.
- Do not copy OCI IAM, instance principal or cost-guard privileges. Leave existing OCI guardrails unchanged. Electricity, local storage and shared subscription limits still apply.

## 5. Asset mapping

| Asset | Action and acceptance requirement |
|---|---|
| Ubuntu/toolchain | Official pinned ARM64 software, systemd as PID 1, required commands, representative dependency install and build |
| tmux/user systemd | Adapt user, workspace and binary paths; authorize linger if needed; verify logout persistence and new session creation after reboot |
| Claude/Remote Control | Initially start manually inside tmux; confirm Sonnet and approval settings; verify a real CLI process and remote operation, not an empty session |
| japanese-guard | Review recovered code; replace hardcoded `/home/ubuntu` paths; English rejected, Japanese accepted, unrelated settings preserved |
| OSS collection | **Excluded from initial migration, disabled by default.** Reconcile repository, credentials, cost, recipients and duplicate execution before a separate bounded `--once` trial |
| Collector timer | 06:30 follows guest timezone; Persistent=true can cause catch-up execution. Confirm timing and exclusion before enabling anything |
| cost-guard | Do not migrate; verify the Mac trial never calls OCI shutdown APIs |
| Work in progress | Clone tracked files from authorized remotes; separately inventory uncommitted changes and runtime state; retain OCI originals and test approved backup/restore |

Use synthetic final text meeting the recovered hook defaults: at least 25 Latin letters and more than three times the Japanese character count. Check exclusions for short “OK” responses, code and URLs, plus the Stop-loop guard. Record any threshold environment overrides with the test conditions.

tmux survives connection loss, not a host/guest reboot. Conversation resumption, job restart and source recovery are separate acceptance items. Do not copy whole credential stores or databases as a migration shortcut.

## 6. Implementation gates

All stages are unstarted. This ordering is not a delivery-date commitment.

1. **P0 Inventory and permission:** reconcile workload state, deferred jobs, resource allocations, VM platform, access and backups. Resolve unknowns and authorize the necessary installation, VM, authentication and configuration actions.
2. **P1 Empty guest:** record trusted image/checksum and versions; create the minimal VM. Accept OS, resources, mounts, exposure and lack of impact on existing services.
3. **P2 Development base:** install tools and reviewed assets; test synthetic hook input, logout persistence and disabled optional timers.
4. **P3 Claude:** owner-authorized official login, manual launch and a small public-repository Remote Control operation. Confirm plan, data flow and stopping procedure.
5. **P4 Representative work:** one repository, dependencies, build and focused tests; record OCI differences, duration, memory, swap, disk and coexistence.
6. **P5 Recovery and handover:** accept the mandatory tests below before ongoing use. Preserve failed and unrun stages explicitly.
7. **P6 Optional extensions:** separately accept collectors, monitoring and automatic startup, including ownership, duplicate prevention, credentials, notifications and costs.

Start on demand. Unattended operation requires separate proof of Mac boot → login/unlock → VM manager → guest systemd → Claude launch. A successful locked-screen read is not a cold-boot test. Disabling FileVault or enabling automatic login is not a prerequisite of this plan.

## 7. Minimum acceptance checks

These tests are **planned, not performed by this documentation update**. Save time, versions/commit, command or action, outcome, redacted evidence and open issues for each.

- **Required coexistence:** representative build stays within resource limits without degrading accepted existing-service criteria.
- **Required access/exposure:** approved routes work; no unapproved LAN/Internet listeners or host-secret mounts.
- **Required workflow:** representative build/focused tests, English/Japanese hook behavior, one Remote Control operation and clean exit.
- **Required disconnect:** reconnect to the same tmux work without duplicate Claude processes.
- **Required guest restart:** persisted data, expected user-service state and documented manual Claude restart where applicable.
- **Required restore:** recover code and required settings to a separately authorized temporary environment and verify checksums/a small build. Reconfigure secrets only through approved channels.
- **Unattended-operation gate:** owner-available maintenance-window Mac restart/power recovery, explicitly recording manual unlock/start steps.
- **Only if adopted:** timer timezone, catch-up, exclusion, duplicate prevention, costs and notification acceptance.

Expired credentials or external outages must not trigger unlimited retries. Return a recorded pending state to the owner. CLI login success or tmux existence alone is insufficient evidence of equivalence.

## 8. Backup and rollback

Before changes, record existing configuration, ports, limits and stopped jobs without publishing private details. Preserve uncommitted work. Track non-secret VM definitions and versions in Git; choose an authorized encrypted backup destination, capacity and retention in P0. A snapshot on the same internal SSD is not device-failure protection.

If the trial causes problems, stop only the new development VM/jobs and reverse its recorded startup/forwarding additions. Do not stop Docker Desktop, OpenClaw, Tailscale or the NUC wholesale. Keep OCI and its data intact, but do not promise instant fallback: OCI restart capacity is not guaranteed. Confirm source/push state and use normal repository remotes for recovery. Retain VM disks/backups until restore is verified; separately reconcile authority before revoking new enrollments or credentials.

## 9. Open implementation decisions

- **P0 blockers:** actual Docker/OpenClaw workloads and allocations, existing VM platform, and construction-operator reachability through authorized permissions.
- Supported Lima/VZ version for macOS 27.2, official Ubuntu image, explicit mount and forwarding configuration.
- Current SSH keys, firewall, FileVault, sleep and power recovery; owner-selected security/unattended-operation balance.
- Representative repository, final guest RAM, coexistence schedule and off-device backup/restore method.
- Current subscription eligibility and shared limits; whether OSS collection stays in OCI, moves, or remains disabled.
- Automatic startup, monitoring and OpenClaw handover only after initial acceptance.

## 10. Keep the record current

Sources: [09](09-oci-cloud-dev-environment.md), [10](10-oci-pitfalls.md), [recovered assets](../instance-assets/README.ja.md) and [asset index](../scripts/README.md), pinned to the OCI source commit in section 2. Public home-lab main was read on November 8. Distinguish its historical results from today's observations and unknowns.

After implementation, update stages as unstarted, under validation, accepted or on hold and add the newest dated entry first in [README](../README.md). A merged plan is not a completed deployment.
