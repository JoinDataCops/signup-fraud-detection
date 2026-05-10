# How to prevent fake signups in 2026: the operator playbook

Let's be real. Fake signups are not a database-hygiene problem in 2026. They are an ad-attribution problem. Bots make up roughly 46% of all online signups per the verified.email 2026 disposable email roundup. 30% of free-tier signups are bots or users hiding behind disposable addresses. 19% of SaaS signups in December 2025 used disposable email. Only 62% of email addresses submitted through online forms are valid. And on one day in April 2026, an operator at hitprobe.com reported that 93% of all signups on a monitored SaaS were fake.

The attack pattern shifted. What started as manual VPN + temp-email cycling went through Puppeteer-stealth and anti-detect browsers (Kameleo, Hidemium) and is now industrialized agentic AI. Static blocklists fail at 59% benchmark detection. Static CAPTCHAs fail (frontier LLMs solve reCAPTCHA v2 at 60% to 100%). And the fake signups feed your Meta CAPI, your Google CAPI, your Andromeda algorithm, your smart bidding. That is the wedge nobody talks about. Stop the fake signup before the pixel fires, not after.

This is the operator playbook. Five-layer stack. Decision tree by traffic volume. Code-level examples. Honest comparison of the vendors that solve a slice. The bundling thesis at the end.

---

## Quick stuff people keep asking

**What percentage of signups are fake in 2026?** Roughly 30 to 46% depending on vertical, source, and offer. Free trials and freemium consumer SaaS see the highest rates. B2B SaaS sees lower rates but higher per-fake-account cost. Gaming hits 18.49% IVT industry-wide per Lunio's 2026 IVT report. AI-SaaS gets hit hardest because the fraudsters are after free GPU credits.

**What is the best way to detect signup fraud?** Layered defense. No single signal works. The five layers in 2026: (1) email validation including disposable detection, (2) IP intelligence (residential vs datacenter vs VPN vs proxy vs Tor), (3) device fingerprinting (canvas, WebGL, audio, screen, fonts), (4) behavioral velocity (form-fill speed, mouse paths, copy-paste detection), (5) post-signup verification (email click confirm, phone OTP, payment hold). Skip any one and the attack pattern that defeats it will eat you.

**How do bots create fake accounts?** In 2026, mostly via agentic AI orchestration. A headless browser fleet running Puppeteer-stealth or Playwright with fingerprint randomization, fed disposable email addresses (lifespan under 7 days), routed through residential proxies that look like real ISPs. The bot completes the signup, sometimes verifies the email, sometimes confirms a phone OTP via SMS-receiving services, then either takes the free credit or sells the account.

**Can email validation alone stop fake signups?** No. Static blocklists detect roughly 59% of disposable domains per industry benchmarks. Hyper-disposable domains live under 7 days. By the time your blocklist updates, the fraudster has cycled to a new domain. Email validation is necessary but not sufficient.

**How does device fingerprinting prevent fake accounts?** It catches the same browser fingerprint signing up multiple times even when IP and email change. Modern fingerprinting uses canvas, WebGL, audio, screen resolution, font list, and timezone. Anti-detect browsers (Kameleo, Hidemium) randomize most of these but leave behavioral artifacts that pure fingerprinting catches.

**What is account opening fraud?** Fraud at the moment of account creation. Distinct from login takeover (existing account compromised) or transaction fraud (real account, fraudulent purchase). Account opening fraud is upstream of both and the cheapest place to catch the attacker.

---

## The 5-layer signal stack (decision tree)

No single layer works. The cost of a false positive (real user blocked) is real. The cost of a false negative (fake user passes) is real. The decision tree is which layer to add when, based on signup volume.

### Layer 1: Email validation (always on)

What it does: Catches obvious disposable, malformed, and high-risk addresses. Static + dynamic blocklists. Domain age check. MX record check. Catch-all domain detection.

When to add: Day one. This is table stakes.

What it misses: Hyper-disposable domains under 7 days old. Real but throwaway addresses (Gmail aliases, Apple Hide My Email). Sophisticated alias techniques.

Vendors that do it well: ZeroBounce, NeverBounce, Abstract API, Hunter. Industry benchmark detection rate sits around 59% for disposable.

### Layer 2: IP intelligence (most signups/day above 100)

What it does: Categorizes the IP as residential, datacenter, mobile, carrier, VPN, proxy, or Tor exit. Maintains reputation scores per IP and per ASN.

When to add: Once you see datacenter/VPN traffic in your signup logs. Usually around 100 signups/day.

What it misses: Residential proxy networks (Bright Data, Oxylabs, IPRoyal) that look like real consumer ISPs. Compromised home routers acting as exit nodes.

Vendors that do it well: IPQualityScore, MaxMind, ipinfo.io. The biggest IP databases in the market track over 360 billion IPs and network ranges.

### Layer 3: Device fingerprinting (most signups/day above 1,000)

What it does: Generates a stable browser fingerprint from canvas, WebGL, audio, screen, fonts, timezone. Catches the same device signing up under multiple emails.

When to add: Once IP intelligence is no longer enough. Usually around 1,000 signups/day or once you see anti-detect browsers in the logs.

What it misses: Anti-detect browsers (Kameleo, Hidemium) that randomize fingerprint surface. Properly headless Playwright with fresh containers per signup.

Vendors that do it well: FingerprintJS (now Fingerprint.com), Castle.io. FingerprintJS is fingerprint-only. Castle layers behavioral signal on top.

### Layer 4: Behavioral velocity (most signups/day above 5,000 OR high-value verticals)

What it does: Tracks form-fill speed, mouse paths, copy-paste behavior, tab focus changes, keystroke timing. Bots fill forms instantly or with suspicious uniformity. Real users fill forms in jagged patterns.

When to add: When fingerprinting alone misses anti-detect-browser bots. Usually around 5,000 signups/day or in fraud-prone verticals (gaming, free trial SaaS, AI credits).

What it misses: Manual fraud (human in low-cost market). Slow-motion bot attacks designed to mimic human pace.

Vendors that do it well: DataDome, Castle.io, Verisoul. Increasingly the agentic-AI-aware vendors emphasize this layer.

### Layer 5: Post-signup verification (regulated industries, high-stakes accounts)

What it does: Email click confirm, phone OTP, ID document check, $1 payment hold. The high-friction, high-confidence layer.

When to add: Regulated industries (banking, healthcare, marketplaces) or when the cost of a fake account exceeds $50.

What it misses: Real users abandoning at the friction. Phone-OTP bypass via SMS-receiving services.

Vendors that do it well: Twilio Verify, Persona, Onfido, Stripe Identity. Each adds friction. Each is a real cost in conversion.

---

## The vendor reality check (4-line dossiers)

**1. IPQualityScore (IPQS)**

The Good: Broad fraud-scoring breadth. Industry benchmark for IP intelligence + email validation + device fingerprinting in one. Strong API. Continues to be the default for combined signals.

Frustrations: Pricing tiered with enterprise plans 'thousands per month'. Free tier intentionally limited (no device fingerprint, no transaction scoring). Sales-gated above the entry tier.

Wish List: Public mid-market pricing. Behavioral velocity layer.

Value for Money: **7.5/10.** Default IP-and-email vendor. Steep paid tier.

Pricing: Free tier limited, paid sales-gated, enterprise 'thousands per month'.

---

**2. Fingerprint (formerly FingerprintJS)**

The Good: Best-in-class device fingerprinting. Pro plan $99/mo, Enterprise sales-gated. Strong API. Wide JS SDK adoption.

Frustrations: Device-only. No email validation, no IP intelligence, no behavioral velocity. You still need other vendors. Sophisticated anti-detect browsers (Kameleo, Hidemium) randomize the fingerprint surface and slip through.

Wish List: Bundle behavioral signal on top of fingerprint. More aggressive anti-detect-browser detection.

Value for Money: **7/10.** Best at the one thing it does. You need 2-3 more vendors to complete the stack.

Pricing: Free tier limited, Pro $99/mo, Enterprise sales-gated.

---

**3. Castle.io**

The Good: Published June 2025 fake-account-creation taxonomy with strong content positioning toward AI-SaaS, social, gaming. Fingerprint + behavioral signal in one. Castle Risk Engine combines signals. Modern API.

Frustrations: Pricing sales-gated. Smaller market presence than IPQS or DataDome. No CAPI/attribution angle. Stops at the account-opening boundary.

Wish List: Public mid-market pricing. CAPI signal protection layer.

Value for Money: **7/10.** Strong content competitor in the category. The product is solid for fingerprint + behavioral.

Pricing: Sales-gated.

---

**4. DataDome**

The Good: Enterprise-grade bot mitigation at CDN scale. Strong behavioral velocity layer. Wide WAF integration (Cloudflare, AWS WAF, Fastly). Real ML pipeline.

Frustrations: Pricing sales-gated, six-figure contracts at scale. Enterprise CDN posture means slow procurement. Overkill for SMB and mid-market.

Wish List: Mid-market self-serve SKU. Lower friction onboarding.

Value for Money: **7/10 at enterprise scale.** Disqualified for SMB/mid-market on price and procurement complexity.

Pricing: Sales-gated, enterprise floor.

---

**5. Verisoul**

The Good: Newer entrant focused on multi-account detection and B2B signup fraud. Strong AI-SaaS positioning. Account-link analysis (find the same human across multiple fake signups).

Frustrations: Younger product, smaller community. Pricing sales-gated. Multi-account detection requires longer baseline data, slow to ramp.

Wish List: Public pricing. Faster cold-start detection.

Value for Money: **6.5/10.** Promising for B2B and AI-SaaS. Still maturing.

Pricing: Sales-gated.

---

**6. Cloudflare Turnstile**

The Good: Free up to 1M requests/month. Dominant CAPTCHA replacement. Privacy-positioned. Drop-in form widget. Cheapest baseline anti-bot layer in the market.

Frustrations: Form-layer only. Stops at the submit. Does not score signup fraud, does not protect CAPI, does not see disposable email or IP reputation. The bot that solves Turnstile (and 11.45% can per recent benchmarks) still creates the fake account.

Wish List: Behavioral signal layer on top of the Turnstile widget. EU-only data path option.

Value for Money: **8/10 as a free baseline.** **5/10 as a complete fraud stack (it is not).**

Pricing: Free up to 1M requests/mo, Enterprise ~$2K/mo.

---

**7. SEON**

The Good: Email + IP + device + social-graph enrichment in one API. Known for digital footprint analysis (does this email exist on social platforms). Mid-market pricing more accessible than IPQS Enterprise.

Frustrations: Social-graph signal degrades as more users use Apple Hide My Email and disposable addresses. Pricing scales fast at volume.

Wish List: Stronger anti-Apple-Hide-My-Email handling. Cheaper entry tier.

Value for Money: **7/10.** Solid mid-market alternative to IPQS.

Pricing: Free trial, paid tiers from low hundreds per month.

---

**8. DataCops (signup fraud as part of the trust-infrastructure layer)**

The Good: Bundles email validation against 160K+ fraud email domains, IP intelligence on 361B+ tracked IPs (146.4B datacenter, 11.9B VPN, 620M proxy/Tor), device fingerprinting (canvas, WebGL, audio, screen, fonts), and real-time risk scoring at the signup form. The unique angle: scores trust at the first-party CNAME tracking layer, so fake signups never reach Meta CAPI, Google CAPI, TikTok, or LinkedIn CAPI in the first place. Same trust signal protects analytics, attribution, and ad-algorithm training. Free Basic tier includes 500 signup verifications/mo. Branded thesis: 'Why CAPTCHA is dead' (humans behind the fraud + 99.9% of CAPTCHAs solved by bots).

Frustrations: SOC 2 Type II in progress. Newer brand than IPQS, Fingerprint, Castle. Behavioral velocity layer narrower than DataDome at enterprise CDN scale. Integration catalog narrower than enterprise CDPs.

Wish List: Deeper post-signup verification API for B2B SaaS. SOC 2 Type II completion.

Value for Money: **9/10 if you also need CAPI signal protection and CMP.** **7/10 as a pure signup-fraud vendor (IPQS and Castle compete head-to-head here).**

Pricing: Free Basic (2K sessions, 500 signup verifications/mo), $7.99/mo Growth, $49/mo Business, $299/mo Organization, Enterprise talk-to-sales. Overage on signup verifications $0.019 per 500.

---

## The CAPI poisoning angle (the wedge nobody else talks about)

This is the part that turns signup fraud from a database problem into an ad-spend problem.

In 2026, your signup form fires a CAPI event. Meta records the conversion. Google records the conversion. Your Andromeda algorithm or smart bidding learns from it. The model now thinks the demographic that just signed up converts. It bids higher to find more of them.

If the signup was fake, you just trained your ad algorithm on noise.

Meta's March 2026 attribution overhaul (DOJO AI coverage, March 2026) made this worse by redefining 'click' to surface signal-quality issues. Cleaner conversion signal matters more than ever.

The operator move: stop the fake signup before the CAPI fire, not after. Static signup-fraud tools that catch fakes after the conversion has already pinged Meta are solving last year's problem. The fix is trust scoring at the network layer that gates the CAPI fire on the trust score.

This is why the bundling thesis matters. A standalone fingerprint vendor (Fingerprint), a standalone IP-intel vendor (IPQS), and a standalone CAPI proxy (Stape) cannot dedup against each other. The same fake user defeats Fingerprint, gets through IPQS because the residential proxy looks clean, and the CAPI fires anyway. The bundled trust layer (DataCops, conceptually similar consolidated vendors) scores once and gates everything.

---

## Code-level: scoring at the signup form

The minimum viable signup-fraud guard in 2026, conceptually:

```javascript
// Pseudocode for a 2026 signup form guard.
async function validateSignup({ email, ip, fingerprint, behavior }) {
  const emailRisk = await emailValidator.score(email);     // Layer 1
  const ipRisk    = await ipIntel.score(ip);                // Layer 2
  const deviceRisk = await fingerprint.score(fingerprint);  // Layer 3
  const behaviorRisk = scoreBehavior(behavior);             // Layer 4

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

The key line is `fireCAPI`. The trust score gates whether the conversion event reaches Meta or Google. Sending a fake conversion is worse than blocking a real signup, because the cost of the fake conversion is paid in future ad spend. Block at high risk. Add friction at medium risk. Allow + fire CAPI only at low risk.

---

## Operator playbook by volume

| Daily signups | Recommended stack |
|---|---|
| Under 100 | Email validation only. ZeroBounce or Abstract API. Manual review of suspicious addresses. |
| 100 to 1,000 | Add IP intelligence (IPQS free tier or DataCops Basic free). Cloudflare Turnstile as form widget. |
| 1,000 to 5,000 | Add device fingerprinting. Fingerprint Pro or Castle. Or upgrade to DataCops Growth/Business for the bundled stack. |
| 5,000 to 50,000 | Add behavioral velocity. DataDome at enterprise scale, or DataCops Organization for the bundled trust + CAPI gating. |
| 50,000+ | Full enterprise stack. DataDome + IPQS Enterprise + post-signup verification + bundled CMP. Or DataCops Enterprise (single-tenant, dedicated IP DB). |

---

## False-positive cost vs fraud cost trade-off

The most common operator mistake is over-tuning for fraud catch-rate without measuring the cost of blocked real signups.

The formula:
- Cost of a fake signup = wasted CAPI training data + wasted free-tier resources + ad-algorithm poisoning over time. Often $5 to $50 per fake account in regulated/high-value verticals.
- Cost of a blocked real signup = customer LTV lost. Often $50 to $500 in B2B SaaS. Lower in freemium consumer.

The right risk threshold is where blocked-fraud-cost saved exceeds blocked-real-signup-cost lost. Most teams set thresholds too aggressive in the first week and burn real users. Run shadow mode (score but do not block) for two weeks before turning enforcement on.

---

## So what should you actually use?

No one-size-fits-all. The real question is what you actually need.

- Bootstrapped B2B SaaS, under 100 signups/day, B2B leads only? **ZeroBounce or Abstract API + Cloudflare Turnstile**.
- Freemium consumer SaaS, 1K to 10K signups/day, fraud poisoning Meta CAPI? **Bundled trust layer (DataCops) or IPQS + Fingerprint + Stape**.
- Free-trial SaaS with credit-card-required trial? **Stripe Radar + post-signup payment hold + IPQS**.
- AI-SaaS giving away GPU credits, getting hammered? **DataCops + behavioral velocity layer (Castle or DataDome) + post-signup phone OTP**.
- Gaming, marketplace, or regulated industry? **DataDome + Persona/Onfido for ID + post-signup OTP + email clickthrough**.
- Spending under $1K/mo on fraud tooling and want a baseline? **Cloudflare Turnstile (free) + DataCops Basic (free)**.

---

## The mistake I see people make

Buying signup-fraud tools that stop at the database. The bot that gets through your fingerprint check signs up, fires your CAPI, and trains your Meta Andromeda algorithm on a fake conversion. You do not just lose the database row. You lose the next month of ad spend that the algorithm is now optimizing toward fake users. Static signup-fraud tools that detect fakes after the conversion has pinged Meta are solving the 2023 problem. The 2026 fix is trust scoring at the network layer that gates the CAPI fire on the score. Block at high risk. Add friction at medium risk. Allow + fire CAPI only when the score earns it.

---

## Now your turn

What is your daily signup volume? What is your current stack catching, and what is leaking through? Drop the numbers below.

---

Research by [DataCops](https://www.joindatacops.com) · First-party tracking, consent infrastructure & fraud prevention.
