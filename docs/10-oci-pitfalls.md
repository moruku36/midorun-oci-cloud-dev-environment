# 10 OCI pitfalls: where AWS / Azure / GCP users get stuck on their first OCI build

English | [日本語](10-oci-pitfalls.ja.md)

A running log of what actually blocked us, what research turned up, and what to watch for, while building on OCI for the first time. Entries are appended as the build in [the design doc (09)](09-oci-cloud-dev-environment.md) progresses.

Each entry carries one of three status labels. **Hit** = actually encountered in this build. **Researched** = learned from documentation and official sources. **Watch** = not yet encountered, written down in advance because it is likely to bite.

## No.1 Sign-in starts with a "Cloud Account Name" (tenancy name) [Hit | 2026-11-01]

**Situation**: Opening cloud.oracle.com asks for a "Cloud Account Name" (the tenancy name) before any username or password. 1Password autofill did not fill this field usefully — the fill was cleared when the page did not change. Without the tenancy name, you cannot get past the first screen.

**What to do**: The tenancy name is in the OCI welcome email from signup. After signing in, it is also shown in the console's profile menu. Lesson: record the tenancy name in the notes field of your password-manager entry.

**On other clouds**: AWS signs in with an email address / IAM user, Azure with a Microsoft account, GCP with a Google account. Being asked for the account's own name first is unique to OCI, and it is the first wall you hit.

## No.2 The free A1 allowance was halved on 2026-06-15 [Researched]

**Situation**: Many articles and blog posts still say "Ampere A1 gives you 4 OCPU / 24GB free." Oracle's current Always Free documentation gives free tenancies 1,500 OCPU-hours plus 9,000 GB-hours per month, which works out to roughly 2 OCPU / 12GB always-on — half of the old figure, changed on 2026-06-15 without a big announcement. The 4 OCPU / 24GB figure now applies only within the allowance on PAYG tenancies.

**What to do**: Check free-tier numbers against the current official documentation, not how-to articles, and check an article's publication date before trusting it. [The design in doc 09](09-oci-cloud-dev-environment.md) is sized to the 2 OCPU / 12GB limit for this reason.

**On other clouds**: Free-tier terms change on every cloud. What makes this one nasty is that the number quietly halved while the older, larger figure kept circulating, so the gap is easy to miss.

## No.3 A1 capacity shortages in the Tokyo region [Hit | 2026-11-01]

**Situation**: On 2026-11-01, creating the instance exactly as designed in doc 09 (VM.Standard.A1.Flex, 2 OCPU / 12 GB, availability domain AP-TOKYO-1-AD-1, Ubuntu 24.04 ARM) failed with the exact error "Out of capacity for shape VM.Standard.A1.Flex in availability domain AD-1". Tokyo has only one availability domain, so "try another availability domain" was not an option here.

**What to do**: Decision: build in the Osaka region instead. Tokyo has heavy enterprise demand, so A1 capacity there is unlikely to free up any time soon. Osaka is commonly used as a DR destination, and the extra latency is acceptable for a development box. The VCN, subnet, and internet gateway already created in Tokyo were left in place — they are free and harmless, and can be reused if Tokyo is ever retried.

**On other clouds**: On AWS, popular instance types do run dry per AZ. Capacity being chronically tight specifically for the free-tier shape is an OCI-flavored problem.

## No.4 The GUI vocabulary does not line up with other clouds [Researched]

**Situation**: Compartment, VCN, Security List, Availability Domain, Shape, Identity Domain. The concepts exist on other clouds, but the names and the layering differ, so a first-timer cannot tell from the console screens what to click.

**What to do**: Keep the mapping table below at hand. Two anchors help most: a Compartment is a partitioning concept that spans the roles of account, subscription, and resource group; an Identity Domain is an IAM layer that changes the sign-in path itself.

| OCI | AWS | Azure | GCP |
|---|---|---|---|
| Compartment | ≈ account / OU-style partition | ≈ subscription / resource group | ≈ project / folder |
| VCN | VPC | VNet | VPC |
| Security List / NSG | Security Group | NSG | Firewall rules |
| Availability Domain | Availability Zone (AZ) | Availability Zone | Zone |
| Shape | Instance type | VM size | Machine type |
| Identity Domain | ≈ IAM identity foundation | ≈ Entra tenant | ≈ Cloud Identity / organization |

**On other clouds**: Every cloud has its own terms. OCI stands out because the resource-partitioning term and the two-layer firewall (Security Lists plus NSGs) map awkwardly, so vocabulary alone blocks first-timers more often.

## No.5 Idle Always Free instances can be reclaimed [Watch]

**Situation**: Always Free compute has an idle-reclamation policy: Oracle can reclaim instances that stay idle. The exact thresholds are to be verified — treat this as unconfirmed until checked.

**What to do**: The mitigation is to make the box genuinely used, not parked. Phase 5 of [the design in doc 09](09-oci-cloud-dev-environment.md) moves real development work onto it, so it is a daily-use dev box. This entry will be updated once the thresholds are verified.

**On other clouds**: Free tiers are usually managed by expiry dates and quotas. Being reclaimed based on how little you use the resource is not something other clouds' habits prepare you for.

## No.6 MFA with a FIDO security key cannot be completed from a remote/agent browser [Hit | 2026-11-01]

**Situation**: OCI sign-in required 2-step verification, and the registered method was a FIDO security key. The key lives on the user's local machine, and WebAuthn can only reach a key attached to the machine running the browser — so from a remote/agent browser, verification could not be completed and sign-in was impossible. A related trap: older Oracle documentation points to a "My profile" menu, but the current console has no "My profile" item at all.

**What to do**: Register an authenticator app (TOTP) as an additional method: Profile menu → "User settings" → Security tab → 2-step verification → Mobile app → Configure, then register the key in 1Password and set TOTP as the default method. The FIDO key was kept as a backup method. Lesson: if a remote or agent browser will ever need to sign in, register a portable TOTP factor in advance — a hardware key cannot follow the session.

**On other clouds**: A security key is equally local on the AWS, Azure, and GCP consoles; WebAuthn is machine-bound everywhere. The OCI-specific part is only where the fix lives (User settings → Security) versus what the older docs say ("My profile").

## No.7 A free tenancy can subscribe to only its home region [Hit | 2026-11-01]

**Situation**: After Tokyo ran out of A1 capacity (No.3), we tried to subscribe to Japan Central (Osaka) on the free tenancy. Region Management showed the warning "You have exceeded the maximum number of regions allowed for your tenancy", and the Subscribe action was disabled for every unsubscribed region — not just Osaka. A free tenancy is limited to its home region (Tokyo here), so the fallback region could not even be opened.

**What to do**: Upgrading the tenancy to Pay As You Go lifted the region limit, and Osaka could then be subscribed normally. If a design assumes a second region (capacity fallback or DR), either plan the PAYG upgrade from the start or treat "home region only" as a hard constraint of the free tenancy. Note the upgrade itself hit another wall first — see No.8.

**On other clouds**: AWS enables all standard regions on an account by default (opt-in regions aside), and Azure / GCP subscriptions are not pinned to a single home region. A region *subscription* quota of one is an OCI free-tier peculiarity.

## No.8 A Promo / Free Trial tenancy may have no payment method on file, which blocks the PAYG upgrade [Hit | 2026-11-01]

**Situation**: On the "Upgrade and Manage Payment" page, the Payment Method showed "-" and the "Upgrade your account" button stayed disabled, with the warning "Add a payment method to upgrade a subscription plan." The tenancy had been created as a Promo / Free Trial account, and no card had ever been registered — an assumption carried over from the design in doc 09, whose Phase 0 expected card registration to be done already. It was not.

**What to do**: The account owner adds the card himself: on the Upgrade and Manage Payment page, open the "···" actions menu on the Payment Method row and choose Add Payment Method. Entering payment credentials is the owner's job — an agent working in a remote browser cannot (and should not) do this step. Once the card is on file, the Upgrade button enables and the upgrade goes through.

**On other clouds**: Paid upgrades need a payment method everywhere, but the trap here is specific: a trial that started without a card leaves the tenancy with nothing on file, and the disabled Upgrade button does not say why until you read the Payment Method row.

## No.9 In Osaka, the plain Ubuntu 24.04 image is x86-only; Minimal is the only Arm 24.04 [Hit | 2026-11-01]

**Situation**: When creating the VM.Standard.A1.Flex instance in Japan Central (Osaka), the image list's plain "Canonical Ubuntu 24.04" turned out to be an x86_64 build, incompatible with the Arm shape. The only Ubuntu 24.04 image compatible with A1.Flex in that region was "Canonical Ubuntu 24.04 Minimal" (aarch64).

**What to do**: Use the Minimal aarch64 image and install the missing packages during base setup — Minimal omits some of what the full image bundles. When picking an image for an Arm shape, check the image's architecture, not just its name and version; whether an Arm build exists also varies by region.

**On other clouds**: AWS AMIs are likewise published per architecture, but the EC2 console filters them by the selected instance type. OCI's create flow lets an incompatible image–shape pair through far enough that the architecture check becomes your job.

## How this document grows

- When the build hits a new pitfall, append it at the end with the next number, a date, and a status label (Hit / Researched / Watch).
- A "Watch" entry is a placeholder. When it actually happens, upgrade it to "Hit" and rewrite the remedy based on what worked. Entries are never deleted.
- If research overturns an assumption, rewrite the remedy in place and note in the Situation what changed.
