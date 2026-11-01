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

## 3. Getting an instance

Use the VM.Standard.A1.Flex shape with an Ubuntu ARM64 image, sized at most 2 OCPU / 12 GB to stay within the free allowance.

A1 capacity in the Tokyo region is often scarce, and instance creation can fail with an out-of-capacity error. The strategy is patience: retry at different times of day, and fall back to the Osaka region if Tokyo stays unavailable.

Keep the boot volume within the free block-storage allowance (200 GB total). The original runs fine on 30 GB, so starting around 50 GB leaves comfortable headroom.

## 4. Base setup: porting the original's design

The OS is Ubuntu (ARM64). The initial setup is scripted, so the box can be rebuilt with the same steps if it ever has to be recreated.

Persistence mirrors the original: a systemd unit creates the tmux session "claude" at boot, and .bashrc auto-attaches to the existing session on login.

Security updates are applied automatically with unattended-upgrades. Automatic reboots, however, stay disabled — reboot timing is a human decision, so the Windows Update problem is not recreated in the cloud.

## 5. Network and access

Inbound stays fully closed. The VCN security list gets no allow rules, and SSH port 22 is not opened.

Administrative access goes through Tailscale. SSH is accepted only from within the tailnet, with public-key authentication only.

Claude Code's Remote Control works over outbound HTTPS only, so it does not conflict with this posture. Sessions are reachable from a phone or browser with no ports opened.

## 6. Installing Claude Code and the Pro sign-in

Claude Code is installed with the native installer, which supports Linux ARM64, so it runs as-is on A1.

Authentication uses the Claude Pro subscription. Remote Control officially requires a claude.ai sign-in on a Pro, Max, Team or Enterprise plan, and API keys are not supported. An ANTHROPIC_API_KEY present in the environment actually blocks Remote Control, so "never set an API key" is a design requirement. This also keeps API spend at zero.

On a headless box, sign-in works by approving the CLI's URL and code from another device. In the persistent session, Remote Control is enabled from startup, mirroring the original's `remoteControlAtStartup: true` setting.

Note that Pro's usage limits (the 5-hour window and the weekly cap) apply on the server exactly as they do locally. For long-running sessions, those limits are the bottleneck.

## 7. Staying free: the ¥0 discipline checklist

- [ ] Keep the instance within the A1 free allowance (2 OCPU / 12 GB on a free tenancy)
- [ ] Keep total block volumes within 200 GB
- [ ] Do not select paid images or paid Marketplace plans
- [ ] Check outbound data transfer in the OCI console monthly (the free allowance cap is unverified)
- [ ] Set a spend alert so any actual charge triggers a notification, even on a free tenancy
- [ ] If the tenancy is ever upgraded to PAYG, budget alerts become mandatory
- [ ] Do not leave unused resources behind (orphaned volumes and the like)

## 8. Build phases and acceptance checks

Execution is handled by Fuwatchi (Muse, running on a cloud VM), working cloud-to-cloud. The Windows-side agent is not used: the local Windows PC has known TLS/npm asset-fetch failures that could block locally driven work.

Only two steps require moruku himself: OCI console sign-in and MFA approval, and approving the Claude Pro (claude.ai) login when Claude Code is installed on the instance. Everything else — instance creation, base setup, Tailscale, and verification — is done by Fuwatchi from the cloud side.

| Phase | Work | Acceptance condition |
|---|---|---|
| 0 | Tenancy checks | Confirm a free tenancy, the home region, and that a card is registered (registration is already done) |
| 1 | Instance creation | A1.Flex at 2 OCPU / 12 GB, Ubuntu ARM64, boot volume around 50 GB can be created |
| 2 | Base setup | systemd + tmux auto-creation, .bashrc auto-attach, automatic updates without automatic reboots all work; a reboot test restores the session |
| 3 | Tailscale | SSH works over the tailnet while inbound stays fully closed |
| 4 | Claude Code + Remote Control | Starts under the Pro sign-in and is reachable from a phone; ANTHROPIC_API_KEY confirmed unset |
| 5 | Move real work over | One real development repository is moved, and dependency fetching and builds pass without the local problems |

## 9. Caveats: limitations and unverified items

A1 capacity is the biggest unknown; there is no guarantee Tokyo capacity can be obtained. The wait is unpredictable and may take days of retries.

The ARM baseline means some x86-only binaries and toolchain pieces may not run. Whether your own toolchain is fully available for ARM64 needs checking in advance.

Free tiers carry a reclaim risk: idle instances may be subject to reclamation (the exact conditions in OCI's official policy are unverified). The mitigation is to use the box as a genuine, active development environment.

Pro's limits are a limitation too. Always-on sessions will hit the 5-hour and weekly caps, so this is not an unlimited autonomous execution platform.

## 10. Summary

Three points. Keep the original's pattern — systemd + tmux, inbound fully closed, Remote Control — and swap only the foundation to OCI Always Free. Authenticate with the Pro sign-in alone, keeping API spend at zero. Treat the free-tier numbers (2 OCPU / 12 GB, 200 GB) as design ceilings, and keep any expansion as a separate PAYG decision.
