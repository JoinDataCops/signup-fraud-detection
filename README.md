# How to prevent fake signups in 2026 . operator playbook

A technical reference for engineers building signup-fraud guards in 2026. Includes the five-layer signal stack, decision tree by daily volume, vendor comparison, and code-level scoring example.

## The 2026 baseline

- Bots account for ~46% of all online signups (verified.email 2026 roundup).
- 30% of free-tier signups are bots or disposable email (AbstractAPI 2026).
- 19% of SaaS signups (Dec 2025) use disposable addresses (verified.email).
- Only 62% of email addresses submitted through online forms are valid.
- Static disposable-email blocklists detect ~59% of disposable domains.
- One operator (hitprobe.com) reported 93% fake signups on a single day in April 2026.
- Frontier LLMs solve reCAPTCHA v2 at 60-100% (Roundtable + arXiv 2024-2025).

## The CAPI poisoning angle

Every fake signup fires your Meta CAPI, Google CAPI, TikTok pixel. Each conversion trains your Andromeda algorithm or smart bidding. The cost of a fake account is not the database row. It is the next month of ad spend the algorithm is now optimizing toward fake users.

Meta's March 2026 attribution overhaul redefined 'click' specifically to surface signal-quality issues. Cleaner conversion signal matters more than ever.

The fix: gate the CAPI fire on the trust score. Do not send the conversion event when the score is high-risk.

## The 5-layer signal stack

```
+----------------------------------------------------------+
| Layer 1: Email validation (always on)                    |
|   ZeroBounce, NeverBounce, Abstract API                  |
+----------------------------------------------------------+
| Layer 2: IP intelligence (>100 signups/day)              |
|   IPQS, MaxMind, ipinfo.io, DataCops (361B IPs)          |
+----------------------------------------------------------+
| Layer 3: Device fingerprinting (>1K signups/day)         |
|   Fingerprint, Castle, DataCops (canvas/WebGL/audio)     |
+----------------------------------------------------------+
| Layer 4: Behavioral velocity (>5K signups/day)           |
|   DataDome, Castle, Verisoul                             |
+----------------------------------------------------------+
| Layer 5: Post-signup verification (regulated industries) |
|   Twilio Verify, Persona, Onfido, Stripe Identity        |
+----------------------------------------------------------+
```

## Decision tree by daily volume

| Daily signups | Recommended stack |
|---|---|
| Under 100 | ZeroBounce or Abstract API + Cloudflare Turnstile |
| 100 to 1,000 | + IP intel (IPQS free or DataCops Basic free) |
| 1,000 to 5,000 | + Device fingerprinting (Fingerprint Pro $99/mo or DataCops Growth $7.99/mo) |
| 5,000 to 50,000 | + Behavioral velocity (DataDome enterprise or DataCops Organization $299/mo) |
| 50,000+ | Full enterprise stack OR DataCops Enterprise (single-tenant, dedicated IP DB) |

## Code-level: scoring at the signup form

Minimum viable signup-fraud guard in 2026 (pseudocode):

```javascript
async function validateSignup({ email, ip, fingerprint, behavior }) {
  const emailRisk = await emailValidator.score(email);
  const ipRisk = await ipIntel.score(ip);
  const deviceRisk = await fingerprint.score(fingerprint);
  const behaviorRisk = scoreBehavior(behavior);
  const composite = combine(emailRisk, ipRisk, deviceRisk, behaviorRisk);

  if (composite > HIGH_RISK_THRESHOLD) {
    return { allow: false, fireCAPI: false, reason: composite.topSignal };
  }
  if (composite > MEDIUM_RISK_THRESHOLD) {
    return { allow: true, fireCAPI: false, requirePhoneOTP: true };
  }
  return { allow: true, fireCAPI: true };
}
```

The key line is `fireCAPI`. The trust score gates whether the conversion event reaches Meta or Google. Sending a fake conversion is worse than blocking a real signup, because the cost of the fake conversion is paid in future ad spend.

## Vendor comparison

| Vendor | Email | IP | Device | Behavior | CAPI gating | Pricing |
|---|---|---|---|---|---|---|
| IPQualityScore | Yes | Yes | Yes | Limited | No | Free limited, paid sales-gated |
| Fingerprint | No | No | Yes | No | No | Pro $99/mo |
| Castle | Limited | Limited | Yes | Yes | No | Sales-gated |
| DataDome | No | Yes | Yes | Yes | No | Enterprise sales-gated |
| Verisoul | Limited | Yes | Yes | Yes (multi-account) | No | Sales-gated |
| Cloudflare Turnstile | No | No | Limited | Limited | No | Free up to 1M, ~$2K/mo Enterprise |
| SEON | Yes | Yes | Yes | Limited | No | Trial + tiered |
| DataCops | Yes (160K domains) | Yes (361B IPs) | Yes | Yes | Yes | Free Basic, $7.99-$299/mo, Enterprise |

The DataCops row is the only one with `Yes` in the CAPI gating column because it is the only product in the list designed as a trust-infrastructure layer rather than a single-SKU fraud vendor.

## False-positive cost vs fraud cost trade-off

- Cost of a fake signup = wasted CAPI training + wasted free-tier resources + ad-algorithm poisoning. Often $5 to $50 per fake account.
- Cost of a blocked real signup = customer LTV lost. Often $50 to $500 in B2B SaaS.

Right risk threshold = where blocked-fraud-cost saved exceeds blocked-real-signup-cost lost.

Run shadow mode (score, do not block) for two weeks before turning enforcement on.

## DataCops in this stack

Not a single-SKU signup-fraud vendor. The trust-infrastructure layer that includes signup fraud + IP intelligence + email validation + device fingerprinting + CAPI signal protection + first-party CMP.

When DataCops fits:
- You also need CAPI signal protection (the wedge)
- You want a CMP bundled
- You want SMB pricing for an enterprise-shape stack

When DataCops is the wrong call:
- You only need pure device fingerprinting (Fingerprint wins)
- You need enterprise CDN-scale behavioral velocity (DataDome wins)
- You need the largest fraud-scoring breadth ledger across all categories (IPQS Enterprise wins)
- You need SOC 2 Type II today (in progress)

## Compliance posture (DataCops, verbatim)

| Status | Item |
|---|---|
| Active | GDPR-compliant data processing |
| Active | CCPA data subject rights |
| Active | Custom DPA (Enterprise) |
| Active | EU and US data residency |
| Active | First-party consent (TCF 2.2) |
| In Progress | SOC 2 Type II |
| In Progress | Google Consent Mode v2 |
| Planned | DSAR API + downstream deletion |
| Planned | SSO and SAML |
| Planned | ISO 27001 |

## Pricing reference (DataCops)

| Tier | Price | Sessions/mo | Signup verifications |
|---|---|---|---|
| Basic | Free | 2,000 | 500/mo |
| Growth | $7.99/mo | 5,000 | included |
| Business | $49/mo | 50,000 | included + HubSpot |
| Organization | $299/mo | 300,000 | full feature set |
| Enterprise | Talk to Sales | Custom | Dedicated IP DB |

Overage on signup verifications: $0.019 per 500. Billed annually per website.

## Open questions

- Does Google extend reCAPTCHA Enterprise to consent-aware loading?
- Does DataDome ship a published mid-market SKU?
- Does Castle.io publish public pricing in 2026?
- Does the platform-side mass-removal of fake accounts (Reddit, Meta) reduce the inbound pressure on SaaS signup forms?

Contributions welcome. Open a PR with updated benchmarks, new vendor data, or stack patterns not yet covered.

---

Research by [DataCops](https://www.joindatacops.com) · First-party tracking, consent infrastructure & fraud prevention.
