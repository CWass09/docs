# TXN-01 — Pre-submission gauntlet and release

**Frontend build specification · VAERION Systems Healthcare · RCM**

| Field | Value |
| --- | --- |
| Story ID | `TXN-01` (supersedes backlog row `R-19`/`R-20`) |
| Pipeline stage | **Transmission** — the gate between `claim` and `transmission` in `source facts → readiness → claim → transmission → acknowledgment → adjudication/remittance → reconciliation → recovery or closure` |
| Title | Gauntlet — must resolve, review before release, recorded advisory |
| Epic | Claims workspace (blueprint component 5, *Pre-submission gauntlet*; component 6, *Claim assembly and routing*) |
| Primary role | Biller (`billing_specialist`) |
| Secondary roles | RCM specialist (`hc_rcm`), Coder/CDI, Provider, Finance/Admin, Compliance (read-only) |
| Route | `/dashboard/billing/claims/[claimId]/release` |
| Replaces | The release path implied by `components/billing/billing-content.tsx` (mock data, collapsed `ClaimStatus`, float money). See §20. |
| Depends on | `R-01` state chip · `R-02` money summary · `R-03` evidence drawer · `R-10` role shells |
| Blocks | `TXN-04` route authority · `ACK-01` lifecycle timeline |
| Build status | **NOT BUILT.** No evidence run exists for this surface. Every acceptance criterion in §18 is a target to be proven on a running build, not a recorded verdict. |
| Spec version | 1.0 · 2026-09-18 |

---

## 1. The story

**As a** biller preparing a claim for external submission,
**I want** a top-line verdict followed by a check matrix showing, for every configured check, its state, why it reached that state, the evidence behind it, its financial impact and who can resolve it,
**So that** I release on the basis of what was actually checked — and can never release on the basis of a badge that hides a check the system could not run.

### Job stories

1. *When* I open a claim that readiness says is ready, *I want* to see which checks actually ran and which could not, *so I can* decide whether "ready" means anything today.
2. *When* a check fails, *I want* to see the payer rule, the claim data that tripped it and the person who can fix it, *so I can* route the work instead of guessing.
3. *When* I release, *I want* the confirmation to restate exactly what leaves the building and over which route, *so that* my attestation is informed.
4. *When* the route is not authorized, *I want* the Submit control to be visibly held with the missing dependency named, *so that* I never believe a test connector is a production route.

### Why this screen exists

The blueprint's first sentence is that the distinctions between the arrows are the foundation of the product. This screen is the last point at which a human can act before the system crosses the `claim → transmission` arrow. Everything it renders is in service of one rule: **a claim that was not checked must not look like a claim that passed.**

---

## 2. Scope

### In scope

- The gauntlet result screen: verdict header, check matrix, grouping, filtering, expansion, per-check remediation routing.
- Evidence drawer for a single check.
- Re-check (single check and all checks).
- Acknowledgement flow for `review_before_release` findings.
- Release confirmation dialog, attestation, submission result banner.
- Route authority display and the held state of the Submit control.
- All loading, empty, partial, error, permission-denied, stale and conflict states of the above.

### Out of scope (and where it lives)

| Not here | Where |
| --- | --- |
| Editing claim lines, codes, modifiers | `CLM-03` claim edit |
| The submission payload preview and raw 837 access | `TXN-05` payload preview (opened from this screen, specified separately) |
| The lifecycle timeline of acknowledgments and responses | `ACK-01` |
| Configuring which checks are enabled for a tenant | Administration |
| Contract rates and expected reimbursement figures | `RCN-04` / `CLM-07` — **must not appear on this screen for clinical roles** |

---

## 3. Domain glossary

The designer needs these to write and read the screen; they are not decoration.

| Term | Meaning on this screen |
| --- | --- |
| **Gauntlet** | The configured set of checks run against a claim before release. Not a score. Not a single verdict. |
| **Check run** | One execution of one check against one claim version, at a point in time. Has its own id, timestamp, rule version and input snapshot. |
| **Outcome** | What the check concluded: satisfied, must resolve, review before release, recorded advisory — or that it could not conclude at all (unavailable, not configured, not applicable, stale). |
| **Unassertable** | The check could not run or its source was unavailable. **This is never a pass and never counts toward one.** |
| **Release** | The authorized human act of approving a claim for external transmission. Produces a `SubmissionAttempt`. Does not mean the payer received anything. |
| **Route authority** | The separate record proving a payer route is enrolled, credentialed, in production mode and unexpired. A configured connector is not a route authority. |
| **CDI** | Clinical documentation integrity — documentation supports the coded level of service. |
| **Coding consensus** | Independent coding passes agree on the code set. |
| **OIG** | Provider is not on the OIG exclusion list. |
| **PECOS** | Provider enrollment is active in Medicare's PECOS. |
| **NCCI** | National Correct Coding Initiative procedure-to-procedure edits. |
| **MUE** | Medically Unlikely Edits — units exceed the per-day maximum for a code. |
| **Timely filing** | The payer's filing deadline for this service date. |
| **Prepayment integrity** | Duplicate, unbundling and pattern checks before money is claimed. |
| **Denial-risk assessment** | A Monte Carlo estimate of denial likelihood. **A prediction, never a fact** — it can only ever be advisory. |

---

## 4. Authority matrix

The screen renders by role over one record. It never forks the data.

| Role (code union) | Sees the screen | Sees raw payer/rule payloads | Can re-check | Can acknowledge a review finding | Can release | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| `billing_specialist` (Biller) | Yes | No — normalized rendering + custody reference | Yes | Yes | **Yes**, when release authority is granted | The primary actor |
| `hc_rcm` (RCM specialist) | Yes | No | Yes | Yes | No | Recovery scope; release is a billing act |
| Coder / CDI | Only checks they own (CDI, coding consensus, NCCI, MUE) | No | Yes, own checks | No | **No** | Coding access never confers release (blueprint 6) |
| Provider | Only the evidence request routed to them | No | No | No | No | Never sees expected reimbursement |
| `administrator` / Finance | Yes, read-only | No | No | No | No | May see financial impact column |
| Compliance | Yes, read-only, including audit tab | Yes, where their authority covers it | No | No | No | Inspects everything, changes nothing |
| Patient | **Never** | — | — | — | — | Route returns 404, not 403 |

**Enforcement rule (must be stated on the design handoff):** every cell above is enforced server-side. The UI reflects the server's answer; it never *is* the answer. The existing `RoleGate` component (`components/core/error-and-utils.tsx:381`) hardcodes `currentUserRole = "physician"` and is a visibility helper only — it must not be the gate for any control on this screen. See §20 finding F-3.

---

## 5. Navigation

### Entry points

| From | Control | Behavior |
| --- | --- | --- |
| Claim detail (`/dashboard/billing/claims/[claimId]`), Readiness tab | Primary button **"Run pre-submission checks"** | Navigates here; claim context preserved in breadcrumb |
| Claims list row action menu | **"Review and release"** | Same route; returns to the list with filters intact on exit |
| Work Next queue item of type `release_ready` | Row click | Same route; the queue item is marked in-progress by the server, not by the client |
| Direct URL / deep link | — | Renders from the URL alone; no client state may be required |

### Exits

| Exit | Destination |
| --- | --- |
| Breadcrumb "Claim CLM-…" | Claim detail, Readiness tab |
| **Release** succeeded | Stay on screen, render the submitted state (§12.7), offer "View lifecycle" → `ACK-01` |
| Per-check **"Send to owner"** | Task created; stay on screen; toast with a link to the task |
| Per-check **"Fix on claim"** | Claim edit at the relevant line, with a return link back to this screen |
| Browser back after release | Screen renders the submitted state — never the pre-release state |

### Breadcrumb

`Billing & Claims / Claim CLM-2026-004417 / Release` — rendered by `BreadcrumbsAuto` (`components/core/page-components.tsx:89`) with `customLabels` supplying the claim number.

---

## 6. Data contract

Every field a designer needs is here with its type, unit and unknown semantics. Where a value can be absent, the absent case has its own rendering in §10.

```ts
// ---------- Primitives ----------

/** Integer minor units. NEVER a float. 12000 = $120.00. */
export type MoneyCents = number & { readonly __brand: "MoneyCents" };

/** RFC3339 UTC instant, e.g. "2026-09-18T14:03:11Z". Rendered in the tenant's timezone. */
export type Instant = string;

export type ClaimPath =
  | "professional"      // 837P
  | "institutional"     // 837I
  | "dental"            // 837D
  | "secondary_cob"
  | "workers_comp"
  | "pharmacy";

// ---------- The gauntlet ----------

export type CheckKey =
  | "cdi"
  | "coding_consensus"
  | "eligibility"
  | "policy_benefits"
  | "oig_exclusion"
  | "pecos_enrollment"
  | "ncci_edits"
  | "mue_units"
  | "timely_filing"
  | "prepayment_integrity"
  | "denial_risk";

/**
 * Outcome of one check run.
 * The last four are UNASSERTABLE: the check reached no conclusion.
 * Unassertable is never a pass and never contributes to a pass count.
 */
export type CheckOutcome =
  | "satisfied"
  | "must_resolve"
  | "review_before_release"
  | "recorded_advisory"
  | "unavailable"        // source or service did not answer
  | "not_configured"     // tenant has not enabled/credentialed this check
  | "not_applicable"     // check does not apply to this claim path/payer
  | "stale";             // last run predates the current claim version

export type CheckGroup =
  | "must_resolve"
  | "review_before_release"
  | "recorded_advisory"
  | "cannot_assert"
  | "satisfied";

export interface CheckRun {
  checkRunId: string;
  key: CheckKey;
  /** Display name, server-supplied so terminology stays configurable. */
  label: string;
  outcome: CheckOutcome;
  group: CheckGroup;
  /** True when this check is required for this claim path + payer. Drives gating (§11.4). */
  required: boolean;
  /** One sentence, plain language, case-specific. Never a rule name alone. */
  why: string;
  /** Present only when outcome is unassertable. Names the missing dependency. */
  unassertableReason?: string;
  /** Financial exposure if released as-is. Absent when not quantifiable — render "Not quantified". */
  impactCents?: MoneyCents;
  /** Days remaining against the governing deadline, for timely_filing and similar. */
  impactDays?: number;
  /** Role that can resolve. Null when nobody can (an unavailable external source). */
  ownerRole: string | null;
  ownerUserId?: string;
  ownerDisplayName?: string;
  /** Remediation route offered to the signed-in user. Empty array = no action available to me. */
  actions: CheckAction[];
  evidence: CheckEvidence;
  ranAt: Instant | null;          // null when never run
  ruleVersion: string | null;     // e.g. "ncci-2026.3"
  /** The claim version this run evaluated. Compared to claim.version to derive `stale`. */
  claimVersionEvaluated: number | null;
  acknowledgement?: Acknowledgement;
}

export interface CheckAction {
  id: string;
  kind: "fix_on_claim" | "send_to_owner" | "request_evidence" | "recheck" | "open_source" | "acknowledge";
  label: string;                  // server-supplied verb, e.g. "Fix units on line 2"
  /** False when the signed-in role lacks authority; render disabled with `disabledReason`. */
  enabled: boolean;
  disabledReason?: string;
  href?: string;                  // for fix_on_claim / open_source
}

export interface CheckEvidence {
  /** What the check consumed. Field/value pairs, already minimized for the caller's role. */
  inputs: Array<{ label: string; value: string; source: string }>;
  /** Authority the rule derives from, e.g. "CMS NCCI PTP edits, Q3 2026". */
  authority: string | null;
  /** Supporting and contrary evidence, kept separate. */
  supporting: EvidenceItem[];
  contrary: EvidenceItem[];
  /** 0..1 calibrated, or null for "unknown" — never render an unlabelled score. */
  confidence: number | null;
  confidenceBasis: string | null;
  /** Custody reference to the retained raw artifact. Not the artifact itself. */
  artifactRef: string | null;
  /** True when the signed-in role may open the raw artifact. */
  artifactAccessible: boolean;
}

export interface EvidenceItem {
  label: string;
  detail: string;
  sourceRef: string;
  observedAt: Instant | null;
}

export interface Acknowledgement {
  acknowledgedBy: string;
  acknowledgedByDisplayName: string;
  acknowledgedAt: Instant;
  reason: string;                 // free text, required, min 10 chars
}

export interface GauntletResult {
  claimId: string;
  claimNumber: string;
  /** Monotonic claim version. Any change invalidates runs evaluated against older versions. */
  claimVersion: number;
  claimPath: ClaimPath;
  payerName: string;
  planName: string | null;
  serviceDateFrom: string;        // ISO date
  serviceDateTo: string;          // ISO date
  totalChargesCents: MoneyCents;
  /** Present only for roles authorized to expected reimbursement. Absent — not zero — otherwise. */
  expectedAllowedCents?: MoneyCents;
  checks: CheckRun[];
  /** Server-computed. The client never derives the verdict itself (§11.4). */
  verdict: {
    releasable: boolean;
    blockingCount: number;
    reviewCount: number;
    advisoryCount: number;
    cannotAssertCount: number;
    satisfiedCount: number;
    /** Total checks the tenant has configured for this claim path. */
    configuredCount: number;
    /** Human-readable reason release is blocked, when it is. */
    blockedReason: string | null;
  };
  route: RouteAuthority | null;   // null = no route configured at all
  generatedAt: Instant;
  /** Server-side freshness budget in seconds; the screen warns past it (§12.6). */
  freshnessBudgetSeconds: number;
}

// ---------- Route authority (blueprint 12.4) ----------

/** Each field is a separate fact. There is no single "connected" flag. */
export interface RouteAuthority {
  routeId: string;
  payerId: string;
  payerName: string;
  clearinghouseName: string | null;
  transactionType: "837P" | "837I" | "837D";
  /** Technical config present and valid. */
  configured: boolean;
  environment: "test" | "production";
  credentialCustody: "present" | "missing" | "expired";
  payerEnrollment: "enrolled" | "pending" | "not_enrolled" | "unknown";
  clearinghouseAcceptance: "accepted" | "pending" | "rejected" | "unknown";
  providerIdentityVerified: boolean;
  permittedTransactionTypes: string[];
  evidenceRef: string | null;
  expiresAt: Instant | null;
  operationalOwner: string | null;
  /** Server's single answer to "may this claim be transmitted on this route right now". */
  releaseAuthorized: boolean;
  /** Ordered, human-readable missing dependencies. Empty when releaseAuthorized. */
  blockers: string[];
}

// ---------- Release ----------

export interface ReleaseRequest {
  claimId: string;
  claimVersion: number;           // optimistic concurrency; mismatch → 409
  routeId: string;
  attestation: true;              // must be explicitly true
  acknowledgedCheckRunIds: string[];
}

export interface SubmissionAttempt {
  attemptId: string;
  claimId: string;
  routeId: string;
  environment: "test" | "production";
  /** What was OBSERVED. Never "accepted", never "paid". */
  transportState: "queued" | "submitted_to_connector" | "receipt_received" | "transport_failed";
  submittedBy: string;
  submittedAt: Instant;
  controlNumber: string | null;
  connectorReceiptRef: string | null;
  failureReason: string | null;
}
```

### API endpoints

| Method | Path | Purpose | Notes |
| --- | --- | --- | --- |
| `GET` | `/api/rcm/claims/{claimId}/gauntlet` | `GauntletResult` | Returns `ETag: W/"{claimVersion}"` |
| `POST` | `/api/rcm/claims/{claimId}/gauntlet/recheck` | Re-run all or one check | Body `{ keys?: CheckKey[] }`; returns `GauntletResult` |
| `GET` | `/api/rcm/check-runs/{checkRunId}/evidence` | `CheckEvidence` (full) | Lazy-loaded by the drawer |
| `POST` | `/api/rcm/check-runs/{checkRunId}/acknowledge` | Record an acknowledgement | Body `{ reason: string }`, min 10 chars |
| `DELETE` | `/api/rcm/check-runs/{checkRunId}/acknowledge` | Withdraw acknowledgement | Allowed until release |
| `GET` | `/api/rcm/claims/{claimId}/routes` | `RouteAuthority[]` | Selectable routes for this claim path |
| `POST` | `/api/rcm/claims/{claimId}/release` | `SubmissionAttempt` | Requires `Idempotency-Key` and `If-Match: W/"{claimVersion}"` |

### Error semantics — every one of these has a distinct rendering (§12)

| Status | Meaning | Screen behavior |
| --- | --- | --- |
| `401` | Session expired | Hand to the app shell's session-timeout provider; preserve intent for post-login return |
| `403` | Role lacks authority | Permission state (§12.5). Names the role that holds the authority. Never a blank screen |
| `404` | Claim not found **or** caller has no tenant-scope | Not-found state. Identical for both causes — it must not confirm existence |
| `409` | `claimVersion` mismatch — the claim changed | Conflict state (§12.8). Preserves the operator's acknowledgements and offers a reload diff |
| `422` | Release rejected: a blocking check re-appeared server-side | Dialog stays open, renders the new blocker inline; never closes silently |
| `423` | Route authority absent/expired — release refused | Held state (§11.5). Names each blocker |
| `503` | One or more check sources unavailable | Partial result: unassertable checks render as unassertable. **The screen still renders.** |
| `504` | Gauntlet timed out | Partial result with a re-check affordance; no check is assumed satisfied |

---

## 7. Layout

### Canvas and grid

| Property | Value |
| --- | --- |
| Shell | Existing `app/dashboard/layout.tsx` + `components/layout/app-shell.tsx` (dark sidebar, workspace nav). This screen renders inside it. |
| Page wrapper | `PageContainer` — `space-y-6 p-6` (24px padding, 24px vertical rhythm) |
| Max content width | 1440px, centered |
| Column gap | 24px |
| Base spacing scale | 4 / 8 / 12 / 16 / 24 / 32 / 48 px (Tailwind 1/2/3/4/6/8/12) |
| Corner radius | `--radius: 0.5rem` (8px). Chips are full-round (`rounded-full`, from `badge.tsx`) |
| Border | 1px `--border` |
| Card elevation | Flat: 1px border, no shadow. Shadow reserved for overlays (dialog, sheet, popover) |

### Breakpoints

| Name | Width | Layout |
| --- | --- | --- |
| `xl` Desktop | ≥ 1280px | Two columns: main `1fr`, right rail `360px` fixed |
| `lg` Small desktop | 1024–1279px | Two columns: main `1fr`, right rail `320px` |
| `md` Tablet | 768–1023px | Single column. Rail content moves **above** the matrix as a 2-up card row. Matrix keeps table form |
| `sm` Mobile | < 768px | Single column. Matrix becomes stacked cards (§7.4). Sticky bottom action bar, 64px tall |

### 7.1 Desktop wireframe (≥1280px)

```
┌──────────────────────────────────────────────────────────────────────────────────────────────┐
│ [sidebar]  Billing & Claims / Claim CLM-2026-004417 / Release                                 │
│            ┌────────────────────────────────────────────────────────────────────────────────┐│
│            │ VERDICT HEADER  (sticky, top: 0, z-40, height 88px, bg --card, border-b)        ││
│            │ ┌────────────────────────────────────────────────────┬─────────────────────────┐││
│            │ │ ⛔ 2 must resolve   ⚠ 3 review   ◐ 1 cannot assert │  [ Re-check all ]       │││
│            │ │ ✓ 5 satisfied of 11 configured                     │  [ Release claim  ▸ ]   │││
│            │ │ Claim CLM-2026-004417 · Aetna PPO · 837P · DOS     │  Held: route not        │││
│            │ │ 09/02–09/02/2026 · $1,240.00 charges               │  authorized             │││
│            │ └────────────────────────────────────────────────────┴─────────────────────────┘││
│            └────────────────────────────────────────────────────────────────────────────────┘│
│            ┌───────────────────────────────────────────────────┐ ┌──────────────────────────┐│
│            │ FILTER BAR                                        │ │ RIGHT RAIL (360px)       ││
│            │ [All 11] [Must resolve 2] [Review 3] [Advisory 1] │ │ ┌──────────────────────┐ ││
│            │ [Cannot assert 1] [Satisfied 5]   ⌕ Filter checks │ │ │ ROUTE AUTHORITY      │ ││
│            ├───────────────────────────────────────────────────┤ │ │ Aetna via Availity   │ ││
│            │ GROUP: MUST RESOLVE (2)                           │ │ │ ⛔ Not authorized     │ ││
│            │ ┌───────────────────────────────────────────────┐ │ │ │ Configured      ✓    │ ││
│            │ │ ⛔ MUE units          Why…       $240  Coder ▸ │ │ │ │ Environment  test    │ ││
│            │ │    Line 2: 97597 × 4 exceeds the per-day max…  │ │ │ │ Credentials   ✓      │ ││
│            │ │    [Fix units on line 2] [Send to coder]  ⌄    │ │ │ │ Payer enrollment ⛔  │ ││
│            │ ├───────────────────────────────────────────────┤ │ │ │ Expires 2026-11-01   │ ││
│            │ │ ⛔ PECOS enrollment   Why…   Not quantified    │ │ │ │ [View route record]  │ ││
│            │ └───────────────────────────────────────────────┘ │ │ └──────────────────────┘ ││
│            │ GROUP: REVIEW BEFORE RELEASE (3)                   │ │ ┌──────────────────────┐ ││
│            │ ┌───────────────────────────────────────────────┐ │ │ │ WHAT WILL BE SENT    │ ││
│            │ │ ⚠ Timely filing      21 days left   RCM   ▸   │ │ │ │ 837P · 4 lines       │ ││
│            │ │ ⚠ Denial risk        advisory-only prediction │ │ │ │ Billed    $1,240.00  │ ││
│            │ └───────────────────────────────────────────────┘ │ │ │ Expected     —       │ ││
│            │ GROUP: CANNOT ASSERT (1)                          │ │ │ │ [Preview payload]   │ ││
│            │ ┌───────────────────────────────────────────────┐ │ │ └──────────────────────┘ ││
│            │ │ ◐ Eligibility  Payer did not respond (503)    │ │ │ ┌──────────────────────┐ ││
│            │ │   Last answered 2026-09-16 14:02 · [Re-check] │ │ │ │ CLAIM CONTEXT        │ ││
│            │ └───────────────────────────────────────────────┘ │ │ │ Owner · Deadline ·   │ ││
│            │ GROUP: SATISFIED (5)                   [collapsed]│ │ │ Source · Last event  │ ││
│            └───────────────────────────────────────────────────┘ │ └──────────────────────┘ ││
│            Checks generated 2026-09-18 14:03 · claim version 7   └──────────────────────────┘│
└──────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 7.2 Regions

| Region | Width | Behavior |
| --- | --- | --- |
| **Verdict header** | full | `position: sticky; top: 0; z-index: 40` (app-shell header is z-50). 88px tall; compresses to 56px on scroll past 200px, keeping counts + actions |
| **Filter bar** | main column | Sticky under the header at `top: 88px` (56px when compressed), z-30 |
| **Check matrix** | main column | Grouped, in the fixed order: Must resolve → Review before release → Cannot assert → Recorded advisory → Satisfied |
| **Right rail** | 360px | Not sticky below `lg`; sticky `top: 112px` at `xl` |
| **Footer meta** | main column | Static, 12px muted text |

**Group order is fixed and never re-sorted by the client.** Within a group, order is server-supplied (by impact descending, then deadline ascending).

### 7.3 Matrix row anatomy (desktop)

Row height 56px collapsed; expands in place to auto. Columns, left to right:

| # | Column | Width | Content | Alignment |
| --- | --- | --- | --- | --- |
| 1 | Outcome chip | 148px | Icon + text, e.g. `⛔ Must resolve` | left |
| 2 | Check | 180px | `label` + `required` marker | left |
| 3 | Why | `1fr` (min 320px) | `why`, single line truncated at row width with a tooltip carrying the full text | left |
| 4 | Evidence | 120px | `[Evidence]` text button + confidence chip when present | left |
| 5 | Impact | 120px | `impactCents` as currency, or `impactDays` as "21 days left", or `Not quantified` | **right**, `tabular-nums` |
| 6 | Owner | 140px | `ownerDisplayName` or role name; `—` when null | left |
| 7 | Action | 180px | Primary action button + overflow menu | right |
| 8 | Expander | 40px | Chevron; whole row is the click target | center |

Expanded row reveals a 3-column detail grid (inputs / authority + version / remediation), 16px padding, `--muted` background at 40% opacity, and the full `why` text unt runcated.

### 7.4 Mobile (< 768px)

Each check becomes a card, 16px padding, 12px gap:

```
┌──────────────────────────────┐
│ ⛔ Must resolve      $240.00  │   ← chip left, impact right (tabular)
│ MUE units                    │   ← label, text-base semibold
│ Line 2: 97597 × 4 exceeds    │   ← why, 2 lines max, "More" expands
│ the per-day maximum of 2.    │
│ Owner: Coder · Ran 14:03     │   ← meta, text-xs muted
│ [Fix units on line 2]  [⋯]   │   ← full-width primary + overflow
└──────────────────────────────┘
```

Sticky bottom bar (64px, `bg --card`, `border-t`): left = counts summary (`2 must resolve`), right = **Release claim** (or the held control). The bar is always visible; it never covers the last card (list gets `padding-bottom: 80px`).

---

## 8. Component inventory

All components exist in the repo unless marked **NEW**. Reuse first.

| Component | Source | Usage here |
| --- | --- | --- |
| `PageContainer`, `PageHeader` | `components/core/page-components.tsx` | Page frame and title block |
| `BreadcrumbsAuto` | same, line 89 | Breadcrumb with `customLabels={{ [claimId]: claimNumber }}` |
| `Card`, `CardHeader`, `CardTitle`, `CardContent` | `components/ui/card.tsx` | Rail cards, mobile check cards |
| `Table`, `TableHeader`, `TableRow`, `TableCell` | `components/ui/table.tsx` | Matrix at ≥768px |
| `Badge` | `components/ui/badge.tsx` | Base of the outcome chip (`variant="outline"` + token classes) |
| `Button` | `components/ui/button.tsx` | All actions. Variants per §9.4 |
| `Sheet` | `components/ui/sheet.tsx` | Evidence drawer, `side="right"` |
| `Dialog` | `components/ui/dialog.tsx` | Release confirmation |
| `AlertDialog` | `components/ui/alert-dialog.tsx` | Destructive confirm on "Withdraw acknowledgement" |
| `Tooltip` | `components/ui/tooltip.tsx` | Truncated `why`, chip definitions, disabled-control reasons |
| `Collapsible` | `components/ui/collapsible.tsx` | Group collapse (Satisfied default-collapsed) |
| `Skeleton` | `components/ui/skeleton.tsx` | Loading state |
| `Separator`, `ScrollArea`, `Textarea`, `Checkbox`, `Select` | `components/ui/*` | As named in the flows |
| `EmptyState` | `components/core/core-ui-components.tsx:818` | No-checks-configured state |
| `SavingIndicator` | `components/core/error-and-utils.tsx:286` | Acknowledgement save feedback |
| `useAnnouncer` | `components/core/error-and-utils.tsx:517` | Screen-reader announcements (§14.3) |
| `useConfirmation` | `components/core/core-ui-components.tsx:795` | Withdraw-acknowledgement confirm |
| `ErrorBoundary` | `components/core/error-and-utils.tsx:35` | Wraps the matrix; a check render error must not blank the page |
| `maskPHIField` / `createMaskedPatientDisplay` | `lib/hipaa/phi-masking.ts` | Patient context in the rail |
| `logAuditEvent`, `logPatientView` | `lib/hipaa/audit-logger.ts` | §16 |
| **`OutcomeChip`** | **NEW** `components/rcm/outcome-chip.tsx` | The canonical state chip for `CheckOutcome`. Text + icon always; never colour alone |
| **`CheckMatrix`** | **NEW** `components/rcm/check-matrix.tsx` | Grouped table/cards, expansion, keyboard model |
| **`EvidenceDrawer`** | **NEW** `components/rcm/evidence-drawer.tsx` | Implements `R-03`; reused by denials and remittance later |
| **`RouteAuthorityCard`** | **NEW** `components/rcm/route-authority-card.tsx` | Field-by-field authority record |
| **`MoneySummary`** | **NEW** `components/rcm/money-summary.tsx` | Implements `R-02`; integer cents only |
| **`ReleaseDialog`** | **NEW** `components/rcm/release-dialog.tsx` | Confirmation + attestation |

---

## 9. Visual specification

### 9.1 Typography

| Element | Class | Size / weight |
| --- | --- | --- |
| Page title (claim number) | `text-2xl font-bold tracking-tight md:text-3xl` | 24/30px, 700 |
| Verdict counts | `text-sm font-medium` | 14px, 500 |
| Group heading | `text-xs font-semibold uppercase tracking-wide text-muted-foreground` | 12px, 600 |
| Check label | `text-sm font-medium` | 14px, 500 |
| `why` text | `text-sm text-muted-foreground` | 14px, 400 |
| Money and counts | `font-mono tabular-nums text-sm` | 14px — **always tabular** |
| Codes (CPT, CARC, control numbers) | `font-mono text-xs` | 12px |
| Meta / timestamps | `text-xs text-muted-foreground` | 12px |

Fonts are `--font-sans: Geist` and `--font-mono: Geist Mono`, already configured in `app/globals.css`.

### 9.2 Outcome chip tokens

Every chip is **icon + text**. Colour is never the sole carrier (blueprint 9, 13).

| Outcome | Label | Icon (lucide) | Text / border token | Background |
| --- | --- | --- | --- | --- |
| `must_resolve` | Must resolve | `AlertCircle` | `--destructive` | `--destructive` @ 10% |
| `review_before_release` | Review before release | `AlertTriangle` | `--warning` | `--warning` @ 12% |
| `recorded_advisory` | Advisory | `Info` | `--info` | `--info` @ 10% |
| `satisfied` | Satisfied | `Check` | `--success` | `--success` @ 10% |
| `unavailable` | Cannot assert — unavailable | `HelpCircle` | `--muted-foreground` | `--muted`, 4px diagonal hatch at 8% opacity |
| `not_configured` | Cannot assert — not configured | `MinusCircle` | `--muted-foreground` | same hatch |
| `stale` | Cannot assert — stale | `Clock` | `--muted-foreground` | same hatch |
| `not_applicable` | Not applicable | `MinusCircle` | `--muted-foreground` | transparent, dashed border |

**The hatch is load-bearing.** It is what makes "we could not check this" visually distinct from "this is fine" at a glance and in greyscale.

### 9.3 Forbidden visual treatments

1. No aggregate score, percentage, letter grade, progress bar or single "clean" badge anywhere on this screen.
2. No green state for any claim carrying an unassertable required check.
3. No colour-only status. No status conveyed by a bare dot.
4. `--success` is reserved for `satisfied` and for a confirmed transport receipt. It must never render on the Release button.
5. Expected reimbursement must not appear for Coder, Provider or any clinical role — absent, not blanked, not `$0.00`.
6. No float currency. All money arrives as `MoneyCents` and is formatted at the edge.

### 9.4 Button variants

| Action | Variant | Notes |
| --- | --- | --- |
| Release claim | `default` (primary blue) | Never green. Disabled/held per §11.5 |
| Re-check all | `outline` | Shows `Loader2` spinner, label → "Re-checking…" |
| Per-check primary (Fix, Send to owner) | `secondary`, size `sm` | |
| Evidence | `ghost`, size `sm`, with `FileSearch` icon | |
| Overflow | `ghost` icon button, `MoreHorizontal` | 40×40 hit target |
| Withdraw acknowledgement | `ghost` with `--destructive` text | Confirms via `AlertDialog` |

---

## 10. Copy deck

Sentence case throughout. No exclamation marks. No "Oops". Copy states what is true and what to do next.

### 10.1 Verdict header

| Situation | String |
| --- | --- |
| Blocking present | `{n} must resolve` |
| Review present | `{n} review before release` |
| Advisory present | `{n} advisory` |
| Unassertable present | `{n} cannot assert` |
| Satisfied | `{n} satisfied of {configuredCount} configured` |
| Sub-line | `Claim {claimNumber} · {payerName}{planName ? " " + planName : ""} · {claimPath label} · DOS {from}–{to} · {charges} charges` |
| Never render | ~~"All clear"~~ ~~"100% pass"~~ ~~"Ready to submit"~~ |

### 10.2 Group headings

`Must resolve ({n})` · `Review before release ({n})` · `Cannot assert ({n})` · `Recorded advisory ({n})` · `Satisfied ({n})`

Group caption (12px muted, always present under the heading):

| Group | Caption |
| --- | --- |
| Must resolve | `Release is blocked until these are resolved.` |
| Review before release | `Release is allowed after someone with authority acknowledges each one, with a reason.` |
| Cannot assert | `These checks reached no conclusion. An unanswered check is not a passed check.` |
| Recorded advisory | `Recorded for the file. These do not block release.` |
| Satisfied | `Checked and satisfied against claim version {claimVersion}.` |

### 10.3 Unassertable reasons (server-supplied `unassertableReason`, rendered verbatim)

| Cause | String |
| --- | --- |
| Source timeout | `The eligibility service did not respond. Last answered {timestamp}.` |
| Not credentialed | `This tenant has no credentials configured for {source}.` |
| Not applicable | `Not applicable to {claimPath} claims for {payerName}.` |
| Stale | `Last run evaluated claim version {v}; this claim is now version {claimVersion}.` |

### 10.4 Release control

| State | Button label | Helper text beneath |
| --- | --- | --- |
| Releasable | `Release claim` | `You will attest to this release. The claim leaves on {routeLabel}.` |
| Blocked by checks | `Release claim` (disabled) | `Blocked: {blockingCount} check{s} must be resolved first.` |
| Blocked by route | `Release held` (disabled, `Lock` icon) | `Held: {firstBlocker}. {n} more.` with a link `View route record` |
| No route configured | `Release held` (disabled) | `No payer route is configured for {payerName} {transactionType}.` |
| Test environment | `Release to test route` | `This is a test route. Nothing reaches the payer.` — rail badge reads `Test environment` |
| In flight | `Releasing…` (disabled, spinner) | `Do not close this tab.` |
| Released | `Released` (disabled) | `Submitted to connector {timestamp}. No payer response yet.` |

### 10.5 Release dialog

- Title: `Release claim {claimNumber}`
- Body sections: `What will be sent` · `Route` · `Unresolved warnings ({n})` · `Attestation`
- Attestation checkbox label: `I have reviewed the unresolved warnings above and I am authorized to release this claim.`
- Confirm button: `Release claim`
- Cancel: `Cancel`
- Warning list heading when non-empty: `Releasing with {n} acknowledged warning{s}`
- Empty warning list: `No unresolved warnings.`
- Footnote, always present: `Releasing records a submission attempt. It does not mean the payer has received or accepted this claim.`

### 10.6 Post-release banner

| Transport state | Banner |
| --- | --- |
| `submitted_to_connector` | `Submitted to connector at {time}. Awaiting a transport receipt.` |
| `receipt_received` | `Transport receipt received at {time}. Control number {controlNumber}. This is not payer acceptance.` |
| `transport_failed` | `Transmission failed: {failureReason}. The claim was not sent. Your release attempt is recorded and can be retried.` |
| `queued` | `Queued for transmission at {time}.` |

**Never** render `Accepted`, `Approved`, `Paid`, `Success` or a green check on any of these.

### 10.7 Acknowledgement flow

- Trigger: `Acknowledge`
- Dialog title: `Acknowledge: {check label}`
- Reason field label: `Why is it acceptable to release with this finding?`
- Helper: `Recorded against your name and this claim version. Minimum 10 characters.`
- Validation error: `Give a reason of at least 10 characters.`
- Saved chip on the row: `Acknowledged by {name} · {time}` with `Withdraw`
- Withdraw confirm body: `Withdrawing removes your acknowledgement. Release will be blocked again until someone acknowledges it.`

### 10.8 Empty, error and permission copy

| State | Title | Body | Action |
| --- | --- | --- | --- |
| No checks configured | `No pre-submission checks are configured` | `This tenant has no checks enabled for {claimPath} claims. Releasing now means releasing unchecked.` | `Open check configuration` (admin only) |
| Gauntlet never run | `These checks have not been run` | `Run them to see what would block this claim.` | `Run checks` |
| Load failure | `The checks could not be loaded` | `{serverMessage}. Nothing about this claim has changed.` | `Try again` |
| Partial (503) | inline banner | `{n} check{s} could not run. They are listed under Cannot assert and are not counted as satisfied.` | `Re-check those` |
| Permission denied | `You do not have access to release this claim` | `Release is held by the biller role. You can view the checks and route work to their owners.` | `Back to claim` |
| Not found | `This claim is not available` | `It may not exist, or it may be outside your tenant.` | `Back to claims` |
| Conflict (409) | `This claim changed while you were reviewing` | `It is now version {v}. Your acknowledgements are kept. Reload to see the current checks before releasing.` | `Reload checks` |

---

## 11. The check catalog and gating rules

### 11.1 The eleven checks

`required` and applicability are server-supplied per tenant, claim path and payer. The table gives the product intent the designer needs to write and lay out each row.

| Key | Label | Owner role | Typical blocking outcome | Impact expressed as | Remediation action |
| --- | --- | --- | --- | --- | --- |
| `cdi` | Documentation integrity | Provider | `must_resolve` when documentation does not support the level billed | Charge at risk (cents) | `Request evidence` → provider task |
| `coding_consensus` | Coding consensus | Coder | `review_before_release` when passes disagree | Charge at risk | `Send to coder` |
| `eligibility` | Eligibility | Front desk | `must_resolve` when coverage is not on file; `unavailable` when the payer did not answer | Full charge | `Verify coverage` → coverage screen |
| `policy_benefits` | Policy benefits | Front desk | `review_before_release` when benefits are stale | Patient responsibility estimate | `Re-verify benefits` |
| `oig_exclusion` | OIG exclusion | Compliance | `must_resolve` on any hit | Not quantified | `Open compliance record` |
| `pecos_enrollment` | PECOS enrollment | Admin | `must_resolve` when enrollment is inactive | Full charge | `Open provider record` |
| `ncci_edits` | NCCI edits | Coder | `must_resolve` on a PTP conflict without a valid modifier | Line charge | `Fix modifiers on line {n}` |
| `mue_units` | MUE units | Coder | `must_resolve` when units exceed the per-day maximum | Excess units × unit charge | `Fix units on line {n}` |
| `timely_filing` | Timely filing | RCM | `review_before_release` inside the risk window; `must_resolve` past the deadline | Days remaining | `Escalate` |
| `prepayment_integrity` | Prepayment integrity | RCM | `review_before_release` on a potential duplicate | Full charge | `Compare with claim {n}` |
| `denial_risk` | Denial risk | RCM | **`recorded_advisory` only — never blocking** | Predicted denial probability, labelled a prediction | `View model evidence` |

**`denial_risk` may never be rendered in the Must resolve or Review groups**, whatever its value. It is a prediction; the blueprint forbids a prediction being treated as a fact (blueprint 1, 14.5). If the API ever returns it in a blocking group, the client renders it under Recorded advisory and logs a contract violation (§16).

### 11.2 Outcome → group mapping (client-side rendering rule)

```
must_resolve          → group "must_resolve"
review_before_release → group "review_before_release"  (acknowledged ones stay in the group, chip changes)
recorded_advisory     → group "recorded_advisory"
satisfied             → group "satisfied"
unavailable
not_configured        → group "cannot_assert"
stale
not_applicable        → group "cannot_assert", rendered last within it, muted
```

### 11.3 Counting rule

`satisfiedCount` counts **only** `outcome === "satisfied"`. Unassertable checks are counted in `cannotAssertCount` and appear in their own group. There is no denominator that hides them: the header always reads `{satisfied} satisfied of {configured} configured`, so `5 satisfied of 11 configured` with 1 cannot-assert is legible as incomplete.

### 11.4 Release gating

The server owns `verdict.releasable`. The client must not compute it. The client's only job is to render the reason and prevent a pointless request.

| Condition | Release control |
| --- | --- |
| Any `must_resolve` unacknowledged | Disabled. Helper names the count |
| Any `review_before_release` without an `acknowledgement` | Disabled. Helper: `Acknowledge {n} finding{s} to continue.` |
| Any **required** check in `unavailable` / `not_configured` / `stale` | Disabled. Helper: `{label} could not be checked. A check that did not run cannot be treated as passed.` |
| Any **optional** check unassertable | Enabled, but the dialog lists it under unresolved warnings |
| `route.releaseAuthorized === false` | Held (§11.5) — a distinct state from disabled |
| `verdict.releasable === true` and route authorized | Enabled |

> **Decision D-1 (confirm before build).** An unavailable *required* check blocks release outright; there is no operator override. The alternative — an authorized override with a reason — was considered and rejected for v1 because an override path is exactly how "unavailable" silently becomes "passed". If the business needs an override, it belongs to Compliance, not Billing, and needs its own story.

### 11.5 Held ≠ disabled

A **disabled** control means *you have work to do on this claim*. A **held** control means *this product is not authorized to do this yet*. They must not look the same.

- Held: `Lock` icon in the button, `outline` variant, muted foreground, cursor `not-allowed`, and a persistent rail card enumerating every blocker from `route.blockers`.
- Disabled: standard disabled styling, helper text naming the count of items to clear.
- Both expose the reason as `aria-describedby` — never tooltip-only, since a tooltip is unreachable by touch.

---

## 12. Screen states

Each is a distinct rendering. None may be reached by accident, and none may be silently substituted for another.

| # | State | Trigger | Rendering |
| --- | --- | --- | --- |
| 12.1 | **Loading** | Initial fetch | Header skeleton (2 lines), 6 matrix row skeletons at 56px, rail card skeletons. No spinner-only screen. Aria: `aria-busy="true"` on the main region |
| 12.2 | **Loaded** | 200 | §7 layout |
| 12.3 | **Never run** | `checks.length > 0`, all `ranAt === null` | `EmptyState` with `Run checks` primary; matrix hidden |
| 12.4 | **Not configured** | `configuredCount === 0` | `EmptyState` per §10.8. Release control held with `No checks are configured for this claim path.` |
| 12.5 | **Permission denied** | 403 | Full-page state per §10.8, breadcrumb intact, no matrix, no counts |
| 12.6 | **Stale** | `now - generatedAt > freshnessBudgetSeconds` **or** any check `claimVersionEvaluated < claimVersion` | Amber inline banner above the filter bar: `These results are from claim version {v}. The claim is now version {current}.` + `Re-check all`. Affected rows move to Cannot assert with the `stale` chip. Release disabled |
| 12.7 | **Released** | Successful POST | Matrix becomes read-only (no actions, no re-check), banner per §10.6, `View lifecycle` link. Browser back must re-render this state, never the pre-release one |
| 12.8 | **Conflict** | 409 | Modal over the current screen per §10.8. Acknowledgements preserved in client state and re-submitted after reload. **Operator input is never discarded** |
| 12.9 | **Partial** | 207/503 | Screen renders fully; failed checks are unassertable; inline banner per §10.8 |
| 12.10 | **Offline** | Network loss | Existing `OfflineIndicator` (`core-ui-components.tsx:442`); release control disabled with `You are offline. Nothing has been sent.` |
| 12.11 | **Session expired** | 401 | App-shell session timeout provider takes over; on return, the screen restores the same claim and any unsent acknowledgement drafts |

---

## 13. Interaction flows

### 13.1 Expand a check

1. Click anywhere on the row (excluding buttons) or press `Enter`/`Space` on the focused row.
2. Row expands in place, 200ms ease-out height transition (`prefers-reduced-motion`: no transition).
3. `aria-expanded` flips on the row; the detail region gets `role="region"` and `aria-label="{check label} detail"`.
4. Only one row expands at a time by default; `Shift+click` keeps others open.
5. Expansion state is **not** persisted across reloads.

### 13.2 Open evidence

1. `Evidence` button, or `E` with the row focused.
2. `Sheet` opens from the right, 480px (100% under 768px), focus moves to its heading.
3. Contents, in order: *What this check is* · *Why this outcome, for this case* · *Inputs* (label/value/source table) · *Authority and version* · *Supporting evidence* · *Contrary evidence* · *Confidence* · *Custody reference* · *What you can do*.
4. Supporting and contrary evidence are separate sections with distinct headings — never merged into one list.
5. Confidence renders as `{value} — {basis}` or, when null, `Confidence unknown` in muted text. Never a bare number, never a progress bar.
6. `Open raw artifact` appears only when `artifactAccessible === true`; otherwise render the custody reference as text with `Your role cannot open the raw transaction.`
7. Closing returns focus to the triggering `Evidence` button.

### 13.3 Acknowledge a review finding

1. `Acknowledge` on a `review_before_release` row → dialog (§10.7).
2. Reason is required, min 10 characters; the confirm button stays disabled until valid.
3. On submit: `SavingIndicator` → optimistic chip on the row → server confirmation.
4. **Read-back requirement:** the row only shows `Acknowledged by …` after the server response returns the persisted acknowledgement. A failed save reverts the chip and surfaces an inline error with the text preserved in the dialog.
5. Announce via `useAnnouncer`: `Acknowledged {label}. {n} remaining to acknowledge.`

### 13.4 Re-check

- **Single:** `Re-check` on the row. That row enters a skeleton state; the rest of the screen stays interactive.
- **All:** header button. All rows skeleton; header counts are replaced by `Re-checking…`; the Release control is disabled for the duration.
- Re-check is **idempotent and safe** — it never mutates the claim.
- If a re-check returns a *worse* outcome, the row animates to its new group with a 300ms position transition and the announcer says `{label} moved to {group}`.

### 13.5 Release

1. `Release claim` → `ReleaseDialog`.
2. Dialog restates: what will be sent (path, line count, billed total), the route (payer, clearinghouse, environment, authority state), unresolved warnings (each acknowledged finding with its reason and who acknowledged it), and the attestation.
3. Confirm is disabled until the attestation checkbox is ticked. The checkbox is never pre-ticked.
4. On confirm: POST with `Idempotency-Key` (UUID generated once per dialog open, reused across retries) and `If-Match`.
5. Dialog stays open with a spinner and `Do not close this tab.` Cancel is disabled while in flight.
6. Success → dialog closes → state 12.7 → announce `Claim released. Submitted to connector. No payer response yet.`
7. `409` → dialog stays open, renders the conflict inline, offers `Reload checks`.
8. `422` → dialog stays open, lists the newly blocking check, and offers `Show me` which closes the dialog and scrolls to that row.
9. `423` → dialog closes, screen enters the held state and the rail card expands.
10. Network failure → dialog stays open: `The release could not be confirmed. Nothing was sent. Retry to try again with the same request.` Retry reuses the same idempotency key.

> **The screen never shows a success state it has not read back from the server.** No optimistic release. No toast standing in for a result.

---

## 14. Accessibility

Target: WCAG 2.2 AA. Healthcare staff, dense data, long shifts, keyboard-first.

### 14.1 Structure and landmarks

- One `<h1>`: `Release claim {claimNumber}`.
- Groups are `<section>` with `aria-labelledby` pointing at the group heading.
- The matrix is a real `<table>` at ≥768px with `<caption class="sr-only">Pre-submission checks for claim {n}, grouped by outcome</caption>`, `scope="col"` headers, and row-level `aria-expanded`.
- Under 768px the cards are a `<ul>` with one `<li>` per check.
- `SkipLink` (`core-ui-components.tsx:846`) targets the matrix.

### 14.2 Keyboard map

| Key | Action |
| --- | --- |
| `Tab` / `Shift+Tab` | Move between regions: header actions → filters → each group → rail |
| `↑` / `↓` | Move between rows within the matrix (roving tabindex, one tab stop for the table) |
| `Enter` / `Space` | Expand or collapse the focused row |
| `E` | Open the evidence drawer for the focused row |
| `A` | Acknowledge the focused row, when eligible |
| `R` | Re-check the focused row |
| `Esc` | Close drawer or dialog, returning focus to the trigger |
| `Cmd/Ctrl + Enter` | Open the release dialog (only when the control is enabled) |
| `?` | Open the shortcut sheet (existing `components/shortcuts`) |

Single-letter shortcuts are disabled while focus is in a text input.

### 14.3 Announcements (`useAnnouncer`)

| Event | Priority | Message |
| --- | --- | --- |
| Results loaded | polite | `{blocking} must resolve, {review} review, {cannotAssert} cannot assert, {satisfied} of {configured} satisfied.` |
| Re-check finished | polite | `Checks updated. {blocking} must resolve.` |
| Acknowledged | polite | `Acknowledged {label}. {n} remaining.` |
| Release in flight | assertive | `Releasing claim. Do not close this tab.` |
| Release result | assertive | Per §10.6, verbatim |
| Load or release error | assertive | The error title plus `Nothing was sent.` where true |

### 14.4 Contrast and motion

- All chip text meets 4.5:1 against its background in both themes; the hatch pattern carries a 3:1 edge against the row.
- Focus ring: existing `--ring`, 3px, never removed.
- `prefers-reduced-motion: reduce` disables expansion, group-move and spinner-pulse animations; state changes remain instant and announced.
- Minimum hit target 40×40px, including the row expander and overflow menu.

### 14.5 Zoom and density

- Usable at 200% zoom and at 320px logical width with no horizontal scroll of the page (the matrix itself may scroll horizontally at ≥768px, with the outcome and check columns frozen).
- Density control (comfortable 56px / compact 44px rows) persists per user, per the financial table primitive `R-05`.

---

## 15. Performance

| Budget | Target |
| --- | --- |
| First contentful paint of the header (cached session) | ≤ 800ms |
| Matrix interactive | ≤ 1.5s on a 4× CPU-throttled mid-tier laptop |
| Evidence drawer open → content | ≤ 400ms (lazy fetch, skeleton until then) |
| Re-check round trip | No budget — it is an external call. The screen stays interactive throughout and shows elapsed time past 5s: `Still running ({n}s)…` |
| Matrix size | ≤ 20 rows realistically; no virtualization needed. Do not add it |

Loading strategy: server-render the header and claim context from the claim record, stream the gauntlet result. A slow check source must never delay the rest of the screen — the row renders as `unavailable` when the server says so, not as a perpetual spinner.

---

## 16. Audit and telemetry

Every item below is an audit event, not analytics. It uses `logAuditEvent` (`lib/hipaa/audit-logger.ts:51`) with `resourceType: "billing"`.

| Event | `accessType` | Payload additions |
| --- | --- | --- |
| Screen opened | `view` | `claimId`, `claimVersion`, `gauntletGeneratedAt` |
| Evidence drawer opened | `view` | `checkRunId`, `key` |
| Raw artifact opened | `view` | `artifactRef` — always audited, always role-gated |
| Re-check requested | `update` | `keys[]` |
| Acknowledgement created / withdrawn | `update` | `checkRunId`, `reason` |
| Release attempted | `update` | `routeId`, `idempotencyKey`, `acknowledgedCheckRunIds[]` |
| Release result | `update` | `attemptId`, `transportState`, `failureReason` |
| **Contract violation** | `update` | Client detected a server response that breaks a product rule (e.g. `denial_risk` returned as blocking, money as a non-integer). Logged and rendered defensively per §11.1 |

Audit rows carry actor, role, tenant, claim, timestamp and correlation id. They never carry the payload of a payer transaction.

---

## 17. Edge cases

| # | Case | Required behavior |
| --- | --- | --- |
| E-1 | Claim edited in another tab while this screen is open | Poll `ETag` every 60s; on change, enter state 12.6 (stale). Never auto-refresh under the operator |
| E-2 | Two billers release the same claim simultaneously | Idempotency key + `If-Match` means one wins; the loser gets 409 and sees the existing attempt, not a second submission |
| E-3 | Check returns an outcome the client does not know | Render under Cannot assert with the raw value in monospace and `Unrecognized outcome — treated as unassertable`. Never default to satisfied |
| E-4 | `impactCents` absent | Render `Not quantified`, never `$0.00` |
| E-5 | `ownerRole` null | Render `—` and, in the drawer, `No one can resolve this from inside the product. It depends on {source}.` |
| E-6 | All 11 checks satisfied but route unauthorized | Header shows all satisfied; Release is **held**, not enabled. The two facts are independent |
| E-7 | Route is `environment: "test"` | Persistent rail badge `Test environment` and dialog line `Nothing reaches the payer.` Release is allowed; the resulting attempt is labelled test everywhere downstream |
| E-8 | Payer returns eligibility after release | Irrelevant to this screen; the check row stays as it was at release. Post-release the matrix is a historical record, not a live view |
| E-9 | `configuredCount` < checks returned | Render all returned checks; log a contract violation. Never hide a check because the count disagrees |
| E-10 | Acknowledged finding's outcome worsens on re-check to `must_resolve` | Acknowledgement is void; row moves to Must resolve with `Your acknowledgement no longer applies — this finding is now blocking.` |
| E-11 | Claim path is one the tenant has not enabled (e.g. `institutional`) | Screen renders read-only with `Institutional claims are not enabled for this tenant.` Release held. Never fall back to the professional path |
| E-12 | Very long `why` (>400 chars) | Truncate to the row width with a tooltip; full text always visible in the expanded row and the drawer |
| E-13 | Timezone | All timestamps render in the tenant's configured timezone with a short zone label; the drawer shows UTC on hover |
| E-14 | Currency | USD only in v1. Formatter takes `MoneyCents` and renders `$1,240.00`. A non-integer input is a contract violation, rendered as `—` plus a logged violation |

---

## 18. Acceptance criteria

Typed truths, in the Story A convention. Each is independently demonstrable on a running build.

**Rendering and state**

- `[ui]` The matrix renders `Check · State · Why · Evidence/source · Impact · Owner/action` for every check the server returns, grouped as Must resolve, Review before release, Cannot assert, Recorded advisory, Satisfied.
- `[state]` An `unavailable`, `not_configured`, `stale` or `not_applicable` check renders in Cannot assert with its own chip and reason, is excluded from `satisfiedCount`, and is visually distinct from `satisfied` in greyscale.
- `[absence]` No aggregate score, percentage, grade, progress bar or single "clean" badge is rendered anywhere on this screen, in any state.
- `[state]` `denial_risk` never renders in a blocking group, whatever the API returns, and is always labelled a prediction.
- `[ui]` The header always reads `{satisfied} satisfied of {configured} configured`, so an incomplete run is legible without opening a group.

**Evidence**

- `[evidence]` Expanding a check shows its inputs, authority, rule version, applicability, result and remediation route.
- `[evidence]` The drawer renders supporting and contrary evidence under separate headings, and renders confidence with its basis or as `Confidence unknown` — never a bare score.
- `[rbac]` `Open raw artifact` appears only when `artifactAccessible` is true; otherwise the custody reference renders as text with the reason.

**Authority**

- `[rbac]` A coder can open this screen, re-check the checks they own, and cannot acknowledge or release; the release control is absent, not merely disabled.
- `[rbac]` A user outside the claim's tenant receives the not-found state, identical to a non-existent claim.
- `[absence]` No control on this screen relies on `RoleGate` alone; every gated action is refused by the server when called directly.

**Release**

- `[state]` Release is disabled while any `must_resolve` is open, any `review_before_release` is unacknowledged, or any required check is unassertable — each with its own helper text naming the cause.
- `[ui]` A route whose `releaseAuthorized` is false renders the control as **held** with a `Lock` icon and every blocker enumerated in the rail — visually distinct from disabled.
- `[ui]` The release dialog restates what will be sent, the route, the environment, every acknowledged warning with its reason and author, and requires an un-pre-ticked attestation.
- `[state]` After a successful release the claim renders as submission attempted; the words Accepted, Approved and Paid appear nowhere, and `--success` is not used on the release control.
- `[absence]` No success state renders before the server response is read back; a network failure during release states `Nothing was sent.`
- `[state]` A retried release reuses its idempotency key and produces one attempt, not two.
- `[event]` Release emits an audit event carrying actor, role, tenant, claim, claim version, route, idempotency key and acknowledged findings.

**Resilience**

- `[state]` A 503 from one check source renders that check as unassertable while the rest of the screen renders fully.
- `[state]` A 409 preserves the operator's acknowledgements and offers a reload; no operator input is discarded on any error path.
- `[ui]` Loading, never-run, not-configured, permission-denied, not-found, stale, partial, offline and conflict are nine visually distinct states.

**Accessibility**

- `[a11y]` Every outcome is conveyed by icon and text, never by colour alone, and the screen is fully operable by keyboard per §14.2.
- `[a11y]` Load, re-check, acknowledge and release outcomes are announced to screen readers with the copy in §14.3.
- `[a11y]` The screen is usable at 200% zoom and 320px logical width with no page-level horizontal scroll.

---

## 19. Test scenarios

```gherkin
Scenario: An unavailable required check cannot be released past
  Given claim CLM-2026-004417 has eligibility configured and required
  And the eligibility service returns 503
  When the biller opens the release screen
  Then eligibility appears under "Cannot assert" with "The eligibility service did not respond"
  And the header reads "5 satisfied of 11 configured"
  And the Release control is disabled with "Eligibility could not be checked"
  And no green or "clean" indicator appears anywhere on the screen

Scenario: A coder cannot release
  Given the signed-in user holds the coder role
  When they open the release screen for a releasable claim
  Then the release control is absent from the screen
  And re-check is available only on CDI, coding consensus, NCCI and MUE
  And a direct POST to /release from that session is refused by the server

Scenario: Release records an attempt, not an acceptance
  Given all required checks are satisfied
  And the route authority is authorized in production mode
  When the biller confirms the release dialog with the attestation ticked
  And the connector returns a transport receipt
  Then the banner reads "Transport receipt received … This is not payer acceptance."
  And the claim state chip reads "Submission attempted"
  And the word "Accepted" does not appear on the screen

Scenario: A configured but unauthorized route is held, not disabled
  Given the route is configured with valid credentials
  And payer enrollment is "not_enrolled"
  When the biller opens the release screen
  Then the release control renders as "Release held" with a lock icon
  And the rail lists "Payer enrollment: not enrolled" as a blocker
  And the control's reason is available to screen readers, not tooltip-only

Scenario: The claim changes mid-review
  Given the biller has acknowledged two review findings
  When the claim is edited elsewhere and becomes version 8
  Then a stale banner appears naming versions 7 and 8
  And affected rows move to "Cannot assert" with the stale chip
  And release is disabled
  And the two acknowledgements are still shown and are re-submitted after reload

Scenario: A double release produces one attempt
  Given the biller confirms release and the response is lost to a network drop
  When they retry from the same dialog
  Then the same idempotency key is sent
  And the server returns the original attempt
  And the screen shows exactly one submission attempt
```

---

## 20. Findings in the current codebase

Grounded in `CWass09/v0-vaerion-healthcare-ui-ux` at `c0d5e41`. These are not style opinions; each one makes this screen impossible to build honestly until it is resolved.

| # | Finding | Evidence | Consequence |
| --- | --- | --- | --- |
| F-1 | `ClaimStatus` collapses the lifecycle into `pending \| submitted \| processing \| approved \| denied \| paid \| appealed` | `lib/types/healthcare.ts:264` | There is no way to express "submitted to connector, receipt received, no payer response". `approved` and `paid` assert outcomes the system has not observed. **Must be replaced by the canonical state model (`R-01`) before this screen ships.** |
| F-2 | Money is `number` (float dollars) across `BillingClaim` and `BillingLineItem` | `lib/types/healthcare.ts:273,300` | Cent-level reconciliation (`RCN-01`) cannot hold. Migrate to `MoneyCents` integers |
| F-3 | `RoleGate` hardcodes `currentUserRole = "physician"` | `components/core/error-and-utils.tsx:381` | Role visibility is not authorization (blueprint 14.6). Every control on this screen must be server-gated |
| F-4 | Three divergent role unions: `UserRole` (healthcare.ts:3), `HealthcareRole` (hipaa/types.ts:137), a third local `UserRole` (error-and-utils.tsx:373) | as cited | The authority matrix in §4 cannot be expressed. Consolidate on one union including `billing_specialist` and `hc_rcm` |
| F-5 | `components/billing/billing-content.tsx` renders mock claims with `status: "approved"` and float amounts | `components/billing/billing-content.tsx:79-120` | This is the surface this story replaces. It should not be extended |
| F-6 | No RCM API client exists in `lib/` | `lib/` contains only `supabase`, `hipaa`, `types`, `utils` | The endpoints in §6 need a typed client with the error semantics table implemented, not ad-hoc fetches |

---

## 21. Open decisions

| ID | Decision | Recommendation | Owner |
| --- | --- | --- | --- |
| D-1 | Override for an unavailable *required* check | No override in v1 (§11.4) | Product + Compliance |
| D-2 | Who may acknowledge a `review_before_release` finding — biller only, or RCM too | Biller and RCM; never coder or provider | Product |
| D-3 | Whether Finance may see `impactCents` on this screen | Yes, read-only; it is financial exposure, not expected reimbursement | Product |
| D-4 | Freshness budget value (`freshnessBudgetSeconds`) | 900s (15 min), tenant-configurable | RCM ops |
| D-5 | Whether re-check is rate-limited per external source | Yes — external calls cost money; server-side limit with a clear client message | Engineering |

---

## 22. Definition of done

**Design deliverables**

1. Desktop (1440), small desktop (1280), tablet (1024) and mobile (390) frames for: loaded, loading, never-run, not-configured, stale, partial, permission-denied, not-found, conflict, released.
2. Every chip in §9.2 in light and dark, plus a greyscale proof that unassertable and satisfied remain distinguishable.
3. Expanded row, evidence drawer, acknowledgement dialog, release dialog (empty-warnings and with-warnings), held route rail card.
4. Redlines against the token names in §9 — not hex values.
5. Keyboard and focus-order annotation over the desktop frame.
6. Copy deck signed off against §10, verbatim.

**Engineering**

7. All nine states reachable in Storybook or a route-level fixture set, from fixtures — not a live backend.
8. Axe clean at AA; manual screen-reader pass of the §14.3 announcements.
9. The §19 scenarios automated.
10. `MoneyCents` formatter with a test asserting a float input renders `—` and logs a violation.
11. No `any` in the §6 contract; the client fails closed on unknown enum values (E-3).

---

## 23. Sources

**Blueprint** — `VAERION_RCM_Product_and_UX_Blueprint_2026-09-16.pdf`, section 1 (the arrows must stay distinct), 3 (product principles: unknown remains unknown, evidence before action, automation is governed, fail closed, minimum necessary), 5 and 5.1 (canonical revenue state model and state families), 6 (roles and authority boundaries), 7.5 (pre-submission gauntlet: the eleven checks, the check matrix, the three outcome groups, the release confirmation, and the prohibition on a single "clean" badge), 7.6 (claim assembly and routing, the submission drawer, post-submit language), 9 (shared UX primitives), 12.4 (external authority contract), 13 (safety, security and trust; no deceptive completion), 14.3 and 14.6 (deliberate holds: live transmission requires real authority evidence; role visibility is not authorization), 15 (UX acceptance criteria).

**Format exemplar** — `VAERION_User_Storyv2_1.xlsx`, sheet `Story A Steps`, row `A-02` (*Check-in — honest eligibility answer and Medicare MBI capture*): the convention of a titled story, a context line, and typed acceptance truths including `[ui]` and `[absence]`, with route mappings named explicitly.

**Code** — `CWass09/v0-vaerion-healthcare-ui-ux` @ `c0d5e4156be4bb5b1228bff31352c1db7d5a7852`: `app/globals.css` (design tokens, oklch, light/dark), `components.json` (shadcn new-york, neutral base, lucide), `package.json` (Next 16.2.6, React 19, Tailwind v4, Radix, recharts, framer-motion), `components/layout/app-shell.tsx` (workspace navigation, Billing workspace), `components/core/page-components.tsx` (`PageHeader`, `PageContainer`, `BreadcrumbsAuto`), `components/core/core-ui-components.tsx` (`EmptyState`, `SkipLink`, `useConfirmation`, `OfflineIndicator`), `components/core/error-and-utils.tsx` (`ErrorBoundary`, `RoleGate`, `SavingIndicator`, `useAnnouncer`, `useFocusTrap`), `components/ui/*` (shadcn primitives), `lib/types/healthcare.ts` (`ClaimStatus`, `BillingClaim`, `BillingLineItem`, `UserRole`), `lib/hipaa/types.ts` (`HealthcareRole`, `ROLE_ACCESS_MATRIX`, `AuditLogEntry`), `lib/hipaa/audit-logger.ts`, `lib/hipaa/phi-masking.ts`, `components/billing/billing-content.tsx` (the surface this replaces).
