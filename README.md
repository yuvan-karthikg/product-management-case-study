# Product Management Case Study: Digital Insurance Purchase Conversion

A data-driven product case study covering problem identification, opportunity prioritization, solution design, MVP definition and launch strategy for a digital insurance product in a mobile app.

**Deliverable:** `Yuvan Insurance Case Study.pdf` (17 slides)
**Tools:** Excel (pivot tables), Python (pandas) for verification, Canva

***

## The Challenge

Many users who start the insurance purchase journey don't finish it. Customer support hears repeated confusion about policy details, add-on options and payment steps. The goal: find out what's happening and identify the highest-impact opportunity to improve conversion.

***

## Step 0: Data Check

Before analysing anything, I checked whether the dataset agrees with itself. It doesn't:

| Column | Says | Conversion |
|--------|------|-----------|
| `SignupCompleted` | 360 of 500 completed | 72.0% |
| `DropOffStage` | 253 of 500 have no drop-off stage | **50.6%** |

The two columns disagree on 247 of 500 users. 177 are marked completed but also have a drop-off stage, and 70 are marked not completed with no stage.

**Decision:** I used `DropOffStage` as the source of truth. It's the only column that shows *where* users leave, and it gives the more conservative baseline. I excluded `AbandonTimestamp` because it's one row per hour for all 500 users and carries no behavioural signal. In a real team, I'd confirm event definitions with data engineering first.

***

## Key Findings

### Where users drop off

| Stage | Drop-offs | % of all drop-offs | Step drop rate* |
|-------|-----------|--------------------|-----------------|
| Policy Selection | 77 | 31.17% | 15.4% (77 of 500) |
| Terms Agreement | 50 | 20.24% | 11.8% (50 of 423) |
| **Payment** | **120** | **48.58%** | **32.2% (120 of 373)** |
| **Total** | **247** | **100%** | |

\*Share of users who reached that step and left there, assuming the flow runs Policy Selection → Terms → Payment.

**Payment is the leakiest step: 1 in 3 users who reach it leave.**

### Why they leave: testing three explanations

**1. Price doesn't separate droppers from buyers**
- Average premium: payment drop-offs AED 100.85 vs completers AED 99.76
- Payment drop-offs are flat across salary bands: 33 in <2000 AED vs 34 in 4000+ AED
- Users who reported "Payment issues" actually reached the end *more* often (58% vs 51% overall)
- → Affordability isn't the main driver

**2. Neither does intent**
- Average intent score: completers 5.48, payment drop-offs 5.36, policy-selection drop-offs 5.97
- → Drop-off isn't about *who* the user is. It's about what the step asks of them

**3. Complexity over-indexes at payment**
- "Signup too complicated" is the #1 complaint overall (102 of 500 users)
- It's 25% of payment drop-offs vs 18% of completers
- Users who contacted support before signup complete less often: 42% vs 53%
- Time in flow is 126s for payment drop-offs vs 120s for completers (+5%). That's a sign of hesitation but weak on its own
- → Points to **cognitive overload at payment**

### What add-ons do (and don't do)

My first hypothesis was that add-on exposure drives payment drop-off. The data doesn't support it:
- Payment drop rate: 24.7% with add-ons vs 22.1% without. That's +2.6 pts and not statistically significant (p ≈ 0.54)
- 74% of payment drop-offs saw add-ons, but so did 72% of all users and 71% of completers, so this is the base rate
- Completion: 50.0% with add-ons vs 52.1% without

**Takeaway:** collapsing add-ons is good hygiene (one fewer decision at payment), but the lift has to come from simplifying the payment step as a whole.

***

## The Solution: Simplified Payment Step

**How it works**
- Single, clear payment summary with the final payable amount
- Optional add-ons collapsed by default (progressive disclosure)
- Payment option chosen earlier in the flow pre-selected
- One primary CTA: "Pay & Activate Policy"
- Lightweight trust cues (cancel anytime, coverage start date)

**Why this wins**
- Targets the leakiest step (32% step drop rate, 120 users)
- Addresses the signal that over-indexes at payment (complexity), not price
- No pricing, policy or backend changes, so it's fast to build and low risk
- Easy to validate with an A/B test

### MVP Scope

| Included | Excluded |
|----------|----------|
| Collapsed add-ons by default | Pricing changes |
| Clear cost breakdown (base + optional) | New education flows |
| Pre-selected payment option | Policy rework |
| Inline trust cues | Backend payment changes |
| One decision at a time | |

**Goal:** validate whether reducing decision load at payment increases payment completion.

***

## Success Metrics

**Primary KPI: payment-step completion** (paid ÷ reached payment)
- Baseline: 67.8% (253 ÷ 373)
- Target: 80.7% (40% fewer payment drop-offs, 120 → 72)
- Test size: ~180 users reaching payment per arm (95% confidence, 80% power)

**Secondary KPIs**
- Overall conversion: 50.6% → ~60%
- "Signup too complicated" among payment drop-offs: 25% → ≤18%
- Pre-signup support contacts ↓
- Time on payment step ↓ (directional, no hard target)

**Guardrails (must not get worse)**
- Add-on attach rate
- Average premium per policy
- Payment failure / retry rate
- Cancellations in the cooling-off period

**Decision rule:** ship if payment-step completion beats control at 95% confidence and no guardrail drops by more than 2 pts. Otherwise iterate.

***

## Expected Impact

Only payment-stage drop-offs are counted. A payment-page change can't recover users who left earlier in the flow.

| Scenario | Fewer payment drop-offs | New policies | Conversion | Relative lift |
|----------|-------------------------|--------------|------------|---------------|
| Conservative | 25% | +30 | 56.6% | +12% |
| **Target** | **40%** | **+48** | **60.2%** | **+19%** |
| Stretch | 50% | +60 | 62.6% | +24% |

**Revenue (target case):** +48 policies × AED 99.97 average premium ≈ **+AED 4.8K in first premium payments per 500 starters**. Recurring value is higher because 57% of users pay monthly or quarterly. I'd size it properly once payment terms are confirmed. The dataset mixes one-off and instalment premiums, so multiplying by 12 would overstate it.

***

## Launch Strategy

**Phase 1: Soft launch (15% of traffic, A/B test)**
- Monitor bugs, errors and user confusion
- Track the primary KPI, guardrails and qualitative feedback

**Phase 2: Feedback & optimization**
- Refine the add-on reveal experience
- Brief the support team on the new flow

**Phase 3: Full rollout (100%)**
- Roll out once the decision rule is met
- Monitor weekly; prepare follow-ups such as personalization

**Next bets after payment:** Policy Selection (77 drop-offs) and Terms (50). A policy comparison tool targets the "Unclear policy details" complaint (92 mentions).

***

## Alternatives Considered

| Option | Why not (for now) |
|--------|-------------------|
| **Dynamic / cheaper pricing** | Price doesn't separate droppers from buyers; adds another decision at payment; risks revenue |
| **Policy comparison tool** | Good idea, but targets a smaller leak (77 vs 120). Strong next bet |
| **Insurance savings pot** | Users want cover now, not in weeks; affordability isn't the main driver; complex build |

***

## What I'd Highlight About My Approach

- **Checked the data before trusting it:** found and resolved conflicting outcome columns
- **Tested explanations instead of assuming them:** price, intent and complexity, each checked against completers
- **Corrected my own first hypothesis:** the add-on effect disappeared once compared against the base rate
- **Sized impact conservatively:** counted only the users the fix can actually reach, with a range rather than a single number
- **Paired a primary KPI with guardrails:** so conversion isn't bought by losing add-on revenue

***

## Dataset Variables

500 users, 24 columns (synthetic case-study data).

| Variable | Description |
|----------|-------------|
| UserID | Unique identifier for each user |
| Age | Age of the user |
| SalaryBand (AED) | Salary range segment (<2000, 2000-3000, 3000-4000, 4000+) |
| PolicyType | Health, Life or Accident |
| PlanDuration | 6 Months, 1 Year or 2 Year |
| PaymentType | Monthly, One-Off or Quarterly |
| InsurancePremiumAmount | Premium cost in AED (monthly or one-time) |
| AddonExposure | Whether the user was shown add-on options |
| AddOnsSelected | Optional add-on cover (FamilyCover, AccidentAddOn, CriticalIllness) |
| AddonDropOff | Abandoned after seeing add-on pricing |
| SignupCompleted | Completed full insurance signup |
| TimeOnSignupPage(s) | Time spent in the signup flow (seconds) |
| DropOffStage | Policy Selection, Terms Agreement or Payment |
| TermsPageScrollDepth (%) | Terms & conditions scroll depth |
| AbandonTimestamp | Timestamp of exit (excluded, see Data Check) |
| IntentScore | Predicted interest level (1-10) |
| PolicyLiteracyScore | Understanding of policy info (0-10) |
| AcquisitionSource | App Store, Referral, Employer Push, WhatsApp Link, SMS Blast |
| SupportInteractionBeforeSignup | Contacted support before attempting signup |
| DaysSinceSignup | Days on the platform before trying insurance |
| PreviousProductUsed | Mobile Recharge, Money Transfer or none |
| PolicyRenewalIntent | Likely, Unlikely, Unsure or N/A |
| ReferralSharedAfterSignup | Shared a referral after signup |
| CustomerFeedback | Open-text feedback category |
