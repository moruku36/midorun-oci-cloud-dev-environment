# 09 Designing an OCI cloud development environment: rebuilding the EC2 setup on Always Free

English | [日本語](09-oci-cloud-dev-environment.ja.md)

A design for porting Minorun's "Claude Code on EC2" setup ([05](05-claude-code-on-ec2.md), the "original" below) to OCI (Oracle Cloud Infrastructure) Always Free, as a personal cloud development environment that costs ¥0 per month.

The original pays about $52–55/month for an EC2 t4g.large (2 vCPU / 8 GB class). This design reproduces the same pattern — always-on, self-recovering, inbound fully closed — inside the free tier. For the overall adoption picture, see [08 Adopting it in your own setup](08-takeaways.md).

## 1. Introduction: goals and constraints

The goal is to move development off the local Windows PC and into the cloud. That removes two recurring pain points: npm/TLS asset-fetch failures, and Windows Update reboots killing work in progress.

The box is a personal, always-on machine. Development work and long agent sessions run here; local devices act as thin clients.

The hard constraint is "free, permanently". Everything must fit inside OCI Always Free and the existing Claude Pro subscription, with zero API spend.

## 2. The OCI free tier as it stands, and the tenancy choice

The Ampere A1 (ARM) free allowance was halved on June 15, 2026. Per Oracle's official Always Free documentation, free tenancies get 1,500 OCPU-hours plus 9,000 GB-hours per month — about 2 OCPU / 12 GB when run around the clock.

Upgrading to a PAYG tenancy restores the allowance to 3,000 OCPU-hours / 18,000 GB-hours (about 4 OCPU / 24 GB), still free within the allowance. The trade-off is risk: anything outside the allowance turns directly into charges.

The recommendation is to start on the free tenancy. 2 OCPU / 12 GB matches or beats the original's t4g.large (2 vCPU / 8 GB); consider PAYG — with budget alerts — only if a measured shortfall appears.

**Changed on 2026-11-01 (superseded by events):** the tenancy was upgraded to Pay As You Go on 2026-11-01, so the build in fact runs on a PAYG tenancy, and the design ceiling is now the PAYG Always Free allowance of 4 OCPU / 24 GB. The trigger was not a compute shortfall: Tokyo A1 capacity ran out (see [the pitfalls log](10-oci-pitfalls.md) No.3), which made the Osaka region necessary, and the free tenancy could not subscribe to Osaka at all (No.7) — the PAYG upgrade lifted that region limit. The recommendation above is kept as the original reasoning; treat it as history.

## 3. Getting an instance

Use the VM.Standard.A1.Flex shape with an Ubuntu ARM64 image, sized at most 2 OCPU / 12 GB to stay within the free allowance.

A1 capacity in the Tokyo region is often scarce, and instance creation can fail with an out-of-capacity error. The strategy is patience: retry at different times of day, and fall back to the Osaka region if Tokyo stays unavailable.

Keep the boot volume within the free block-storage allowance (200 GB total). The original runs fine on 30 GB, so starting around 50 GB leaves comfortable headroom.

**As built on 2026-11-01:** instance `midorun` was created in Japan Central (Osaka, ap-osaka-1): VM.Standard.A1.Flex, 4 OCPU / 24 GB (the PAYG free allowance), boot volume 50 GB, RUNNING since 2026-11-01 11:49 JST. Public IPv4 129.225.187.56, private IPv4 10.0.0.241, SSH user `ubuntu`. One image deviation from the design above: in Osaka, the plain Canonical Ubuntu 24.04 image is x86-only, so the box runs Canonical Ubuntu 24.04 Minimal aarch64 — the only Arm Ubuntu 24.04 image offered there (see [the pitfalls log](10-oci-pitfalls.md) No.9). Minimal omits some packages the full image bundles; the missing pieces are added during base setup.

**Changed on 2026-11-02:** the shape was reduced to VM.Standard.A1.Flex, 1 OCPU / 6 GB. Osaka had no host capacity for the 4 OCPU / 24 GB shape (nor for 2 OCPU / 12 GB) on restart — see [the pitfalls log](10-oci-pitfalls.md) No.14. The operating model also changed: the instance is no longer always-on. It is kept Stopped and started only when used; after the verification work on 2026-11-02 it was stopped again.

## 4. Base setup: porting the original's design

The OS is Ubuntu (ARM64). The initial setup is scripted, so the box can be rebuilt with the same steps if it ever has to be recreated.

Persistence mirrors the original: a systemd unit creates the tmux session "claude" at boot, and .bashrc auto-attaches to the existing session on login.

Security updates are applied automatically with unattended-upgrades. Automatic reboots, however, stay disabled — reboot timing is a human decision, so the Windows Update problem is not recreated in the cloud.

**As built on 2026-11-01:** the base setup added the packages Minimal omits (tmux 3.4, git, ripgrep, build-essential and friends), Node.js v22.23.4 (NodeSource, arm64) and Claude Code 2.1.316 (npm global install). `loginctl enable-linger ubuntu` keeps the user services running across logouts; the user unit `claude-session.service` creates the persistent tmux session "claude" and is enabled, and the .bashrc auto-attach snippet is in place. The japanese-guard hook ([04](04-japanese-guard.md)) was installed to `~/.claude/hooks/japanese-guard.py` only after a line-by-line source review (Python standard library only; no network access, no command execution, no file writes), verified byte-identical to the reviewed source (sha256 b882d790…), and registered in `~/.claude/settings.json` for Stop and PreToolUse (wrap_up) alongside `remoteControlAtStartup: true`. Smoke tests passed: English replies are blocked for a Japanese rewrite, Japanese passes, and an English wrap_up is denied.

## 5. Network and access

Inbound stays fully closed. The VCN security list gets no allow rules, and SSH port 22 is not opened.

Administrative access goes through Tailscale. SSH is accepted only from within the tailnet, with public-key authentication only.

Claude Code's Remote Control works over outbound HTTPS only, so it does not conflict with this posture. Sessions are reachable from a phone or browser with no ports opened.

**Changed on 2026-11-01 (owner decision — deviation from the design above):** public port 22 stays open for now, restricted to key authentication (verified on the box: PasswordAuthentication no, PubkeyAuthentication yes, PermitRootLogin without-password). The reason is the operator's access path: the environment Fuwatchi runs in cannot reach the tailnet address — SSH to 100.75.1.65 is refused at the network level ("Operation not permitted") even though the box has joined the tailnet, while SSH to the public address works. Closing the public rule on schedule would have cut off the build operator (see [the pitfalls log](10-oci-pitfalls.md) No.12). Revisit if that access path changes.

## 6. Installing Claude Code and the Pro sign-in

Claude Code is installed with the native installer, which supports Linux ARM64, so it runs as-is on A1.

Authentication uses the Claude Pro subscription. Remote Control officially requires a claude.ai sign-in on a Pro, Max, Team or Enterprise plan, and API keys are not supported. An ANTHROPIC_API_KEY present in the environment actually blocks Remote Control, so "never set an API key" is a design requirement. This also keeps API spend at zero.

On a headless box, sign-in works by approving the CLI's URL and code from another device. In the persistent session, Remote Control is enabled from startup, mirroring the original's `remoteControlAtStartup: true` setting.

Note that Pro's usage limits (the 5-hour window and the weekly cap) apply on the server exactly as they do locally. For long-running sessions, those limits are the bottleneck.

## 7. Staying free: the ¥0 discipline checklist

- [ ] Keep the instance within the A1 free allowance (2 OCPU / 12 GB on a free tenancy; 4 OCPU / 24 GB on this PAYG tenancy, as upgraded 2026-11-01)
- [ ] Keep total block volumes within 200 GB
- [ ] Do not select paid images or paid Marketplace plans
- [ ] Check outbound data transfer in the OCI console monthly (the free allowance cap is unverified)
- [ ] Set a spend alert so any actual charge triggers a notification, even on a free tenancy
- [x] Budget guardrail in place on the PAYG tenancy: budget "guardrail-monthly" of ¥750/month, with alerts at 20% / 50% / 100% of actual spend and forecast, emailed to the account owner (set 2026-11-01 as USD 5; corrected to ¥750 on 2026-11-02 — the tenancy is JPY-denominated, see [the pitfalls log](10-oci-pitfalls.md) No.13)
- [x] Auto-stop safety net in place (2026-11-01, implemented before the Phase 2–4 setup as scheduled): a script on the instance itself, `/usr/local/bin/cost-guard.py`, run every 30 minutes by the systemd timer `cost-guard.timer`, authenticates with an instance principal (dynamic group `midorun-guard-dg`; policy `midorun-guard-policy` grants read usage-reports, manage instances, and use ons-topics — the root compartment is referenced as "tenancy" in the statements) and reads month-to-date spend through the Usage API. Past the threshold of 750 it publishes to the Notifications topic `cost-guardrail-alerts` (email subscription confirmed by the owner) and stops all running instances. The Usage API returns JPY amounts in this tenancy, so the threshold is a JPY comparison — it was originally set to "5" assuming USD, which stopped the instance on 2026-11-02 at about ¥11 of spend, and was corrected to 750 the same day (see [the pitfalls log](10-oci-pitfalls.md) No.13). Verified live: month-to-date read 0.0000 at setup (the script logs the JPY figure with a "USD" label), a test publish was delivered, DRY_RUN is off, and the timer is enabled and active. Note the mechanism is an on-instance script rather than an OCI Function; the effect — stop and notify past the threshold — is as designed
- [ ] Do not leave unused resources behind (orphaned volumes and the like)

## 8. Build phases and acceptance checks

Execution is handled by Fuwatchi (Muse, running on a cloud VM), working cloud-to-cloud. The Windows-side agent is not used: the local Windows PC has known TLS/npm asset-fetch failures that could block locally driven work.

Only two steps require moruku himself: OCI console sign-in and MFA approval, and approving the Claude Pro (claude.ai) login when Claude Code is installed on the instance. Everything else — instance creation, base setup, Tailscale, and verification — is done by Fuwatchi from the cloud side.

| Phase | Work | Acceptance condition |
|---|---|---|
| 0 | Tenancy checks | Done 2026-11-01: the tenancy was a Free Trial (Promo) account, home region Tokyo. No payment method was on file, so the owner added one, and the tenancy was upgraded to Pay As You Go the same day |
| 1 | Instance creation | Done 2026-11-01: `midorun` created in Osaka (Tokyo was out of capacity) — A1.Flex 4 OCPU / 24 GB, Ubuntu 24.04 Minimal ARM64, boot volume 50 GB, RUNNING |
| 2 | Base setup | Done 2026-11-01: packages added on Ubuntu 24.04 Minimal (tmux 3.4, git, ripgrep, build-essential, …), Node.js v22.23.4 (NodeSource, arm64), linger enabled for `ubuntu`, the user unit `claude-session.service` creates the persistent tmux session "claude" (enabled), .bashrc auto-attach in place; japanese-guard hook installed after source review (see §4) |
| 3 | Tailscale | Done 2026-11-01: Tailscale 1.104.4 joined the owner's tailnet as `midorun` (100.75.1.65, Tailscale SSH enabled). One deviation from the acceptance condition: public SSH stays open for now, by owner decision — see §5 |
| 4 | Claude Code + Remote Control | Done 2026-11-01: Claude Code 2.1.316 signed in with the owner's Claude Pro account (claude.ai authentication; no API key set anywhere), Remote Control active in the resident session (`/rc active`) |
| 5 | Move real work over | One real development repository is moved, and dependency fetching and builds pass without the local problems |

**Status as of 2026-11-01:** Phases 0–4 are complete. The auto-stop safety net (§7) was implemented first, as scheduled, followed by the base setup, Tailscale, and Claude Code with the Pro sign-in and Remote Control. One deviation remains, by owner decision: public SSH stays open (key-only) instead of being closed — see §5. What remains is Phase 5, moving real work onto the box.

**Status update 2026-11-02:** the currency mismatch in [the pitfalls log](10-oci-pitfalls.md) No.13 fired both the budget alerts and the cost-guard, stopping the instance at about ¥11 of month-to-date spend. The budget and the guard threshold were both corrected to ¥750 / 750 and verified. The shape was reduced to 1 OCPU / 6 GB to get the instance started again (No.14), and SSH reachability was triaged per route (No.15). The instance is now kept Stopped and started only when used.

## 9. Caveats: limitations and unverified items

A1 capacity is the biggest unknown; there is no guarantee Tokyo capacity can be obtained. The wait is unpredictable and may take days of retries.

The ARM baseline means some x86-only binaries and toolchain pieces may not run. Whether your own toolchain is fully available for ARM64 needs checking in advance.

Free tiers carry a reclaim risk: idle instances may be subject to reclamation (the exact conditions in OCI's official policy are unverified). The mitigation is to use the box as a genuine, active development environment.

Pro's limits are a limitation too. Always-on sessions will hit the 5-hour and weekly caps, so this is not an unlimited autonomous execution platform.

## 10. Development agent policy

The development agent for this environment is Claude Code (the CLI). Anthropic usage is API-based in principle, and a subscription sign-in is never routed through third-party tools (the Claude Code in this environment itself runs on the official CLI's Pro sign-in, as described in §6).

Antigravity is not used. Reason: Section 6 of Google's Antigravity Additional Terms treats access through third-party software as a breach of the agreement (it explicitly names using Antigravity OAuth with OpenClaw as an example), and in February 2026 a large number of paying users were suspended without warning. Enforcement applies at the account level and extends to Antigravity itself, its CLI, and Gemini CLI, and the details of how violations are determined are not disclosed. To avoid that risk, Antigravity is limited to use through the official client on the owner's own device.

## 11. Summary

Three points. Keep the original's pattern — systemd + tmux, inbound fully closed, Remote Control — and swap only the foundation to OCI Always Free. Authenticate with the Pro sign-in alone, keeping API spend at zero. Treat the free-tier numbers (2 OCPU / 12 GB, 200 GB) as design ceilings, and keep any expansion as a separate PAYG decision.
