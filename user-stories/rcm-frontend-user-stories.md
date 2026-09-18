# VAERION Healthcare — RCM frontend build backlog (Story R)

Derived from `VAERION_RCM_Product_and_UX_Blueprint_2026-09-16.pdf` (source snapshot 2026-09-16, checkout `design/dental-oral-wellness-pack` at `5a7ec3d5`). 50 stories, sequenced by the blueprint's own recommended design-delivery order (section 16).

No evidence run exists for this frontend. Every acceptance criterion below is a **target to be proven**, not a recorded verdict — unlike Story A, whose rows carry live pass/fail counts from an evidence run.

The spreadsheet form of this backlog is `VAERION_RCM_Frontend_User_Stories_v1.xlsx`, in the same `ID · Task Details · Hours · User story` layout as `VAERION_User_Storyv2_1.xlsx`.

## How to read a row

- **Typed truths.** Each acceptance criterion is tagged so it can be demonstrated or refuted on a running build: `[ui]` visible behavior · `[state]` fidelity to the canonical revenue state model (blueprint 5) · `[money]` amount separation and reconciliation · `[evidence]` why-for-this-case, source and confidence · `[rbac]` role, tenant and authority enforcement · `[event]` audit attribution · `[a11y]` keyboard and assistive-technology operation · `[absence]` a state the interface must never render.
- **`[absence]` truths** are the blueprint's honesty rules made testable: no fabricated $0, no green badge over an unavailable check, no success toast without a read-back, no silence rendered as acceptance.
- **Build status** is `NOT BUILT` (new surface), `REDESIGN` (a component named in blueprint 17 exists and is re-specified here, not certified), or `HELD` (buildable now, but a named dependency keeps it labelled not production ready).

## Waves

| Wave | Stories | Hours |
| --- | ---: | ---: |
| Wave 0 — Foundation: canonical objects, state vocabulary, money model, evidence, roles (blueprint 16.1) — R-01…R-10 | 10 | 156 |
| Wave 1 — Make the lifecycle usable: Work Next, claims, readiness/gauntlet, responses, denials (blueprint 16.2) — R-11…R-25 | 15 | 340 |
| Wave 2 — Remittance, reconciliation and A/R before executive reporting (blueprint 16.3) — R-26…R-31 | 6 | 128 |
| Wave 3 — Contracts, expected reimbursement and integrity as a controlled Admin/Finance flow (blueprint 16.4) — R-32…R-34 | 3 | 76 |
| Wave 4 — Standalone intake, mapping, identity exceptions and the two shells (blueprint 16.5, 11) — R-35…R-40 | 6 | 120 |
| Wave 5 — Patient financials as a bounded view over the reconciled ledger (blueprint 16.6) — R-41…R-44 | 4 | 80 |
| Wave 6 — Analytics, audit, governed learning, claim-path extensions and cross-cutting acceptance (blueprint 16.7, 12, 13, 15) — R-45…R-50 | 6 | 120 |
| **Total** | **50** | **1020** |

## Wave 0 — Foundation: canonical objects, state vocabulary, money model, evidence, roles (blueprint 16.1) — R-01…R-10

### R-01 — State chip — the canonical revenue state, never a page-local label

**Task:** State chip — one canonical lifecycle vocabulary  
**Hours:** 12

> As any RCM user,  
> I want to see one shared state vocabulary (source received, draft, needs completion, ready for review, held, approved for release, submission attempted, receipt/rejected/accepted for adjudication/status unknown, adjudicated, reconciled/denied/underpaid/ambiguous remit/patient balance, recovery in progress/closed/written off) rendered by a single chip component,  
> So that no screen invents its own label that hides the financial truth of the record.

**Context:** Blueprint 5 and 5.1 (canonical revenue state model, state families) and 9 (shared UX primitives). The state model is a state machine, not a progress bar: a claim can carry several submission attempts, responses, a corrected resubmission, a secondary filing and an appeal level at once.

**Build status:** NOT BUILT — new shared primitive. This row is a design target, not an evidence-run verdict; nothing here is claimed to pass today.

**Acceptance criteria (typed truths):**

- `[ui]` the chip renders text plus icon plus an accessible description; colour never carries the meaning alone
- `[state]` every rendered value maps to a named state in the section 5 machine; an unmapped value renders as "unknown state" rather than the nearest guess
- `[state]` the six state families (source quality, readiness, external transport, financial, recovery, integrity) keep separate vocabularies — a readiness result can never render as a transport state
- `[absence]` no "clean" or "complete" chip is emitted for a record whose inputs include an unavailable, stale or inapplicable source
- `[a11y]` chip state is announced by a screen reader and reachable by keyboard in both list and detail contexts

### R-02 — Money summary — every amount category stays its own number

**Task:** Money summary — billed/expected/allowed/paid/adjusted/PR/remaining  
**Hours:** 16

> As a biller, RCM specialist or finance user,  
> I want to see billed, expected, allowed, paid, contractual adjustment, other adjustment, patient responsibility and remaining balance as separately labelled figures with their source and time,  
> So that no single overloaded "paid" value can stand in for a financial outcome the system has not observed.

**Context:** Blueprint 9 (money summary primitive) and 12.3 (financial reconciliation contract: currency/precision, source remit, claim/line linkage, payer versus patient responsibility, adjustment type, posting state, reversible/correcting entry relation, balance-after).

**Build status:** NOT BUILT — new shared primitive consumed by claim detail, remittance, A/R and patient financial surfaces.

**Acceptance criteria (typed truths):**

- `[money]` each amount category renders as its own labelled figure; the component exposes no combined or inferred "paid" field
- `[money]` predicted, contracted, posted and observed values are visually distinguishable, right-aligned and currency- and precision-safe (integer minor units, no float rounding in display)
- `[evidence]` hover or detail on any figure names its source remit or claim line, posting state, actor and time
- `[absence]` a missing contracted rate renders "no contracted rate" with a review action — never $0, a guessed amount or a blank cell
- `[a11y]` each figure is announced with its label, not as a bare number; the summary is navigable by keyboard

### R-03 — Evidence drawer — every recommendation carries its reason, source and route to act

**Task:** Evidence drawer — why this, for this case  
**Hours:** 16

> As any user asked to act on a blocker, risk, variance, denial or automated proposal,  
> I want to open a drawer that states what the finding is, why it applies to this case, its authority and source, supporting and contrary evidence, confidence or explicit unknown, version, and the action available to me,  
> So that I act on evidence rather than on an unexplained badge.

**Context:** Blueprint 3 (evidence before action; automation is governed) and 9 (evidence drawer primitive). Every blocker, risk, variance, denial and automated proposal must open this drawer.

**Build status:** NOT BUILT — new shared primitive; the single largest consumer is the pre-submission gauntlet (R-19) and denial detail (R-24).

**Acceptance criteria (typed truths):**

- `[ui]` the drawer opens from the row, chip or card that raised the finding, without losing the caller's context or scroll position
- `[evidence]` the drawer shows what, why for this case, authority/source, supporting and contrary evidence, confidence or explicit unknown, rule/model version, and a route to act
- `[evidence]` confidence renders as a calibrated value with its basis, or as an explicit unknown — never an unlabelled score
- `[rbac]` raw source artifacts open only for roles authorized to them; other roles see the normalized rendering plus the custody reference
- `[absence]` a hypothesis is labelled as a hypothesis; the drawer never presents a proposed root cause as a determined finding

### R-04 — Lifecycle timeline — a receipt is not an acceptance, an acceptance is not cash

**Task:** Lifecycle timeline — submissions, acks, status, remits in sequence  
**Hours:** 20

> As a biller or RCM specialist,  
> I want to see every submission attempt, transport receipt, 999, 277CA, 276/277 status, payer portal or manual evidence, 835 and posting in one chronological trail with distinct markers,  
> So that I can tell what the payer actually said from what the system merely sent.

**Context:** Blueprint 1 (the arrows must stay distinct), 7.7 (response and status tracking) and 9 (lifecycle timeline primitive).

**Build status:** NOT BUILT — new shared primitive; depends on the response correlation contract in blueprint 12.2 being available from the API.

**Acceptance criteria (typed truths):**

- `[ui]` submission, transport receipt, 999, 277CA, 276/277, portal/manual evidence, 835 and financial posting each render as a distinct marker type
- `[state]` a receipt never collapses into an acknowledgment, an acknowledgment never into adjudication, an adjudication never into reconciled cash
- `[evidence]` every marker shows source, received time, correlation key, parser status, human-readable meaning and the work action it produced
- `[ui]` silence renders as "response overdue" with the governing time rule and a follow-up action attached
- `[absence]` an absent response is never rendered as acceptance, adjudication or payment

### R-05 — Financial table — dense, keyboard-operable, minimum-necessary

**Task:** Financial table and worklist row primitives  
**Hours:** 24

> As a biller or RCM specialist working a queue,  
> I want a table with sticky headers, frozen identity and amount/action columns, density control, saved views, persistent filters, bulk actions and a visible reset, whose rows tell me why an item is next,  
> So that I can work a queue at volume without opening every record or seeing PHI I do not need.

**Context:** Blueprint 9 (financial table, worklist row) and 13 (PHI minimization, accessibility). This primitive is consumed by claims, readiness, responses, remittances, denials, A/R and integrity worklists.

**Build status:** NOT BUILT — new shared primitive; the existing packages/clinical-ui list pages predate this specification and are in scope for re-fit, not keep-green.

**Acceptance criteria (typed truths):**

- `[ui]` sticky header, frozen identity and amount/action columns, right-aligned tabular currency, density control, saved views, persistent filters, bulk actions and a clear reset all function together
- `[ui]` a row carries state, amount, deadline, owner, exception reason and next action — enough to decide why it is next without opening the record
- `[rbac]` patient identifiers and clinical detail are redacted or minimized in list context according to role; opening detail is a deliberate, audited act
- `[a11y]` table, sort, filter, row selection, bulk action and the reset are fully keyboard and screen-reader operable
- `[ui]` selecting a row opens a detail pane; deep work routes to a full detail view and returns to the same queue position and filter state

### R-06 — Actionability card — no card without an owner and a next action

**Task:** Actionability card — issue, impact, owner, deadline, next action  
**Hours:** 8

> As any user receiving work,  
> I want each surfaced issue to name its impact, owner, deadline, next action and what happens if the action fails,  
> So that work is routed to whoever can actually resolve it instead of sitting in an undifferentiated list.

**Context:** Blueprint 9 (actionability card) and 2.5 (route an incomplete, denied, ambiguous or unauthorized transaction to the person who can resolve it).

**Build status:** NOT BUILT — new shared primitive.

**Acceptance criteria (typed truths):**

- `[ui]` the card names issue, impact, owner, deadline and next action in every instance
- `[ui]` a failed save or transport names the failed dependency and offers retry or recovery without discarding operator input
- `[absence]` the card never offers a primary action the signed-in role lacks authority for; it names the role that holds it instead

### R-07 — Change review — success is declared only after a read-back

**Task:** Change review — prior/current, attestation, conflict, read-back  
**Hours:** 16

> As any user making an authorized change,  
> I want to see prior versus current values, the source and reason, validation flags and any required attestation before I commit, and a verified read-back after,  
> So that a toast never becomes the evidence that my change was saved.

**Context:** Blueprint 9 (change review), 13 (no deceptive completion) and 15.3 (an authorized action saves, retains source/audit attribution and reads back after refresh or fresh sign-in).

**Build status:** NOT BUILT — new shared primitive; consumed by claim edit, posting correction, contract approval and refund flows.

**Acceptance criteria (typed truths):**

- `[ui]` prior and current values, source/reason, validation flags and the required attestation are shown before commit
- `[ui]` a conflict or failed save preserves the operator's edits and renders a saved-versus-edited comparison rather than discarding either side
- `[ui]` the success state appears only after the persisted record is re-read; the read-back survives a page refresh and a fresh sign-in
- `[absence]` no toast, redirect or optimistic row update stands in for a confirmed write
- `[event]` the change emits an audit event naming actor, time, tenant, object, action and pre/post version

### R-08 — Empty states — an empty queue is success, an unavailable source is not

**Task:** Empty, blocked, unavailable and error states  
**Hours:** 12

> As any user,  
> I want no records, unavailable source, permission denied, invalid input and partial data to look like five different things,  
> So that I never read a silent failure as a clean result.

**Context:** Blueprint 3 (unknown remains unknown; fail closed for money and authority), 9 (empty/blocked/error state) and 13 (no deceptive completion).

**Build status:** NOT BUILT — new shared primitive; applies to every list, drawer, tile and detail pane in the product.

**Acceptance criteria (typed truths):**

- `[ui]` a genuinely empty worklist renders as a success state, visually distinct from an error
- `[ui]` no records, unavailable source, permission denied, invalid input and partial data render as five distinguishable states with their own copy and recovery route
- `[ui]` every blocked or error state retains any draft and offers a meaningful retry or escalation
- `[absence]` an empty list or a cleared view never implies that an external operation completed

### R-09 — Audit event — attributable, reviewable, and not a PHI dump

**Task:** Audit event rendering and evidence custody references  
**Hours:** 12

> As a compliance officer or practice owner,  
> I want to read an audit trail that names actor, time, tenant, object, action, pre/post version, source evidence and correlation, including reads of protected claim detail,  
> So that I can inspect what happened without the audit surface itself becoming an uncontrolled disclosure.

**Context:** Blueprint 9 (audit event primitive) and 13 (audit, evidence custody, PHI minimization).

**Build status:** NOT BUILT — new shared primitive rendered inside the claim detail Audit tab (R-16) and the audit explorer (R-46).

**Acceptance criteria (typed truths):**

- `[event]` every entry names actor, time, tenant, object, action, pre/post version where relevant, source/evidence reference and correlation id
- `[rbac]` reads of protected claim detail appear in the trail as attributable events, not only writes
- `[absence]` the trail renders custody and hash references rather than protected payload content; raw artifact access is a separate authorized route
- `[a11y]` the trail is filterable, paginated and keyboard-operable, and exports carry the same minimization rules as the screen

### R-10 — Roles — the interface never implies authority the server does not grant

**Task:** Role workspaces and authority boundaries  
**Hours:** 20

> As a product owner defining the role shells,  
> I want front desk, provider, coder/CDI, biller, RCM specialist, finance/admin, compliance/owner and patient each to get their own workspace over the same records,  
> So that the same underlying record renders by job without creating separate truth or forked data.

**Context:** Blueprint 6 (people, jobs, authority boundaries; the hc_rcm role policy) and 14.6 (role visibility is not authorization). Expected reimbursement must stay out of point-of-care coding surfaces (blueprint 3, 7.11).

**Build status:** NOT BUILT — new shell work. Backend least-privilege enforcement is assumed and separately verified; the frontend must not be the only gate.

**Acceptance criteria (typed truths):**

- `[rbac]` each role sees only its own job surfaces: front desk coverage/estimate/collection, provider evidence requests, coder/CDI queue, biller release controls, RCM recovery, finance contracts/reports, compliance audit, patient own-balance
- `[rbac]` the hc_rcm role exposes recovery claims, denials, payments, dashboard, payer intelligence, audit and bounded chart read only — no clinical write, provider attestation, coding/CDI authority, eligibility/pre-visit configuration, settings or corporate authority
- `[ui]` expected reimbursement and contract-rate figures are absent from clinical charge-capture and coding surfaces regardless of the signed-in user's other permissions
- `[rbac]` an unauthorized role and a cross-tenant actor are refused both visually and at the server, and the refusal names the holder of the authority
- `[absence]` no control is merely hidden in place of an authorization check, and no disabled control implies that authority exists but is temporarily unavailable

## Wave 1 — Make the lifecycle usable: Work Next, claims, readiness/gauntlet, responses, denials (blueprint 16.2) — R-11…R-25

### R-11 — Home tiles — every total is clickable and reconciles to its rows

**Task:** Home — today's financial truth tiles  
**Hours:** 16

> As an RCM specialist or practice admin opening the product,  
> I want total actionable A/R, imminent deadlines, unreconciled remits, held claims and new denials on the landing surface,  
> So that the first screen states the practice's financial truth instead of a decorative dashboard.

**Context:** Blueprint 8.3 region 1 (today's financial truth) and 12 (analytics need cohort, source, period, denominator and freshness).

**Build status:** NOT BUILT — the standalone product opens on an exception-first Home, not an EHR-style chart (blueprint 8.1).

**Acceptance criteria (typed truths):**

- `[ui]` the tiles show total actionable A/R, imminent deadlines, unreconciled remits, held claims and new denials
- `[money]` each total is clickable and the rows it opens sum exactly to the displayed figure at claim, patient and practice level without double-counting a balance
- `[ui]` each tile names its cohort, filter, period and data freshness
- `[absence]` a tile whose source is unavailable renders unavailable with the failed dependency, never 0

### R-12 — Work Next — a server-ordered queue that explains its own order

**Task:** Home — Work Next queue with inspectable priority  
**Hours:** 24

> As an RCM specialist,  
> I want a prioritized queue with filters for assigned-to-me, team, payer, facility, financial impact and state, where the reason an item is ranked where it is can be inspected,  
> So that I work the highest-value recoverable item next instead of scrolling an undifferentiated ledger.

**Context:** Blueprint 2.3 (a small, prioritized exception queue), 7.9 (priority by recoverable amount, likelihood of valid recovery, time to deadline, impact, source completeness) and 8.3 region 2.

**Build status:** NOT BUILT — ordering is server-supplied; the frontend renders and explains it and must not re-sort into a different truth.

**Acceptance criteria (typed truths):**

- `[ui]` the queue is server-ordered and offers filters for assigned-to-me, team, payer, facility, financial impact and state
- `[evidence]` each item exposes its priority explanation — recoverable amount, likelihood of valid recovery, time to deadline, practice/patient impact and source completeness
- `[rbac]` the queue is tenant-bound and role-scoped; no item outside the signed-in scope appears even in aggregate counts
- `[ui]` claiming an item assigns visible ownership that survives refresh and a fresh sign-in
- `[absence]` client-side sorting never overrides or silently reinterprets the server's ordering rationale

### R-13 — Flow health — the funnel shows unknown and unavailable as real stages

**Task:** Home — flow health funnel with unknowns visible  
**Hours:** 12

> As a practice admin or RCM lead,  
> I want a source intake to readiness to submission to response to remit to recovery funnel where unknown and unavailable states are visible,  
> So that a stalled or silent stage is legible instead of disappearing from the chart.

**Context:** Blueprint 8.3 region 3 (flow health with unknown/unavailable states visible).

**Build status:** NOT BUILT — new surface.

**Acceptance criteria (typed truths):**

- `[ui]` the funnel renders intake, readiness, submission, response, remit and recovery with counts and amounts
- `[state]` unknown and unavailable counts render as their own stage segments rather than being dropped or merged into a neighbouring stage
- `[ui]` every stage segment is clickable into the worklist whose rows produce it
- `[a11y]` the funnel exposes its values as a keyboard-navigable table equivalent, not as an image-only chart

### R-14 — Watchlist — authority and contract changes surface before they cost cash

**Task:** Home — watchlist for route, contract and pattern changes  
**Hours:** 12

> As an RCM lead or practice admin,  
> I want payer route and authority changes, contract effective-date changes, overdue evidence, repeated denial clusters and data-quality anomalies on the Home surface,  
> So that a change in external authority is noticed before claims fail against it.

**Context:** Blueprint 8.3 region 4 (watchlist) and 12.4 (external authority contract: expiry/renewal, operational owner).

**Build status:** NOT BUILT — new surface; depends on the route authority record exposing expiry and enrollment state.

**Acceptance criteria (typed truths):**

- `[ui]` the watchlist shows payer route/authority changes, contract effective-date changes, overdue evidence, repeated denial clusters and data-quality anomalies
- `[evidence]` each item opens its evidence drawer naming the change, its source, its effective time and who owns the response
- `[ui]` an expiring or expired route authority appears here before it is used, with the claims it would block

### R-15 — Claims list — a work queue, not a static report

**Task:** Claims workspace — list view  
**Hours:** 28

> As a biller or RCM specialist,  
> I want to find, filter, group and assign claims with status, claim/source number, payer/plan, service date, owner, total charges, expected allowed where authorized, remaining A/R, deadline, exception reason, last event and next action,  
> So that I can operate the claim inventory without opening every record.

**Context:** Blueprint 7.4 (claims workspace list columns) and 13 (PHI minimization in list context). Existing surface: packages/clinical-ui/src/pages/ClaimsList/.

**Build status:** REDESIGN — an existing ClaimsList component predates this blueprint; this row re-specifies it rather than certifying it.

**Acceptance criteria (typed truths):**

- `[ui]` the list renders every required column: status, claim/source number, payer/plan, service date, assigned owner, total charges, expected allowed when authorized, remaining A/R, deadline, exception reason, last event, next action
- `[rbac]` the expected-allowed column appears only for roles authorized to expected reimbursement; it is absent, not blanked, for others
- `[rbac]` patient identifiers and clinical detail are minimized in the row per role, and opening detail is audited
- `[ui]` saved views, persistent filters, bulk assignment and a visible reset all survive navigation to a claim and back
- `[state]` status renders through the canonical state chip; no list-only status vocabulary is introduced

### R-16 — Claim detail — one record, nine tabs, no collapsed distinctions

**Task:** Claims workspace — claim detail header, tabs, right rail  
**Hours:** 32

> As a biller or RCM specialist,  
> I want a claim detail with a header (current state, next action, owner, deadline, amount summary), tabs for Overview, Lines and diagnoses, Readiness, Submission history, Responses, Remittance and ledger, Recovery, Documents and Audit, and a right rail of blockers, tasks, provenance and activity,  
> So that the full operational and evidentiary picture of a claim is reachable from one place.

**Context:** Blueprint 7.4 (claim detail layout) and 15.6 (claims, submissions, acknowledgments, status, remits, denials, appeal drafts and payments remain separately visible and correlated). Existing surface: packages/clinical-ui/src/pages/ClaimDetail/.

**Build status:** REDESIGN — existing ClaimDetail predates this blueprint; the tab set and the separation it enforces are the specification.

**Acceptance criteria (typed truths):**

- `[ui]` the header shows current state, next action, owner, deadline and the money summary from R-02
- `[ui]` all nine tabs exist and are deep-linkable, and the right rail shows critical blockers, linked tasks, source/provenance and activity
- `[state]` submissions, acknowledgments, status responses, remittances, denials, appeal drafts and payments each remain separately visible and correlated, with none collapsed into another
- `[ui]` an edit conflict or failed save preserves edits and requires the read-back from R-07 before success is shown
- `[rbac]` opening the detail from a list is an audited read, and tab content respects the signed-in role's scope

### R-17 — Claim edit — an override names its reason and never overwrites the source

**Task:** Claim editing — source facts versus local corrections  
**Hours:** 24

> As a biller,  
> I want to edit a claim within my authority while externally supplied facts stay visibly distinct from my local corrections,  
> So that the original source record remains intact and every override is explainable later.

**Context:** Blueprint 7.1 (a draft claim editor separates externally supplied facts from local corrections and requires a reason for an override), 4.2 (RCM never silently alters clinical facts or provider attestation) and 6 (biller must not override clinical evidence or attestation).

**Build status:** NOT BUILT — new behavior over the existing claim form.

**Acceptance criteria (typed truths):**

- `[ui]` externally supplied facts and local corrections are visually separated at field level, with the source value always readable
- `[ui]` an override captures a reason and records author, revision and time before it can be saved
- `[rbac]` clinical facts and provider attestation are not editable from the billing surface by any role; the surface offers a source-bound request instead
- `[rbac]` a coder cannot release a claim by virtue of coding access, and a biller cannot alter coding authority's determinations
- `[event]` each override emits an audit event carrying prior value, new value, reason and actor

### R-18 — Ready to Bill — five answers per row, no silent handoff

**Task:** Readiness worklist — what blocks billability and who fixes it  
**Hours:** 24

> As a biller,  
> I want each readiness row to answer what the source event is, what is ready, what blocks billability, who can resolve it and what financial or time impact is at stake,  
> So that a blocked encounter is routed to its resolver rather than parked in a queue nobody owns.

**Context:** Blueprint 7.3 (coding and charge readiness; the readiness worklist). Existing surface: packages/clinical-ui/src/pages/ReadyToBill/.

**Build status:** REDESIGN — an existing ReadyToBill surface exists; the five-answer row contract and the line-level provenance requirements are new.

**Acceptance criteria (typed truths):**

- `[ui]` every row answers all five questions: source event/encounter, what is ready, what blocks billability, who can resolve it, financial and time impact
- `[evidence]` each code or line shows source/provenance, code system, diagnosis linkage, units, modifiers, charge amount, author/revision and evidence state
- `[rbac]` the biller sees the operational readiness result and a link back to the responsible source, not an editable clinical narrative
- `[absence]` a multi-check readiness result is never reduced to a single green dot or a pass/fail boolean
- `[ui]` a blocked row offers the handoff that moves it to its resolver, and the receiver can return it with a reason

### R-19 — Gauntlet — must resolve, review before release, recorded advisory

**Task:** Pre-submission gauntlet — explainable check matrix  
**Hours:** 28

> As a biller preparing a claim for release,  
> I want a top-line verdict followed by a check matrix of Check, State, Why, Evidence/source, Impact and Owner/action across the configured checks (CDI, coding consensus, eligibility, patient policy benefits, OIG, PECOS, NCCI, MUE, timely filing, prepayment integrity and denial-risk assessment),  
> So that I release on the basis of what was actually checked, not a badge.

**Context:** Blueprint 7.5 (pre-submission gauntlet; eleven check categories; three outcome groups). Enabled checks are configuration- and evidence-aware — an unavailable source is not a pass.

**Build status:** NOT BUILT as specified — gauntlet routes exist in workers/healthcare/src/healthcare/rcmOrchestrator.js; the explainable matrix and unavailable-is-not-pass rendering are the new frontend contract.

**Acceptance criteria (typed truths):**

- `[ui]` the matrix renders Check, State, Why, Evidence/source, Impact and Owner/action for every applicable check, grouped as must resolve, review before release and recorded advisory
- `[state]` an unavailable, not-configured or inapplicable check renders in its own state with the reason — it never renders as a pass and never contributes to a pass count
- `[evidence]` expanding a check shows its input, authority, version, result, applicability and remediation route
- `[absence]` no single "clean" badge or aggregate score is displayed that could hide an unavailable or inapplicable check
- `[rbac]` the owner/action column names the role that can resolve each finding, and offers the action only to that role

### R-20 — Release — the confirmation states what will be sent, and by whom

**Task:** Release confirmation and attestation  
**Hours:** 16

> As a release-authorized biller,  
> I want a confirmation that summarizes exactly what will be submitted, over which route, with the final unresolved warnings and the attestation I am making,  
> So that release is a deliberate, attributable act rather than a button press.

**Context:** Blueprint 7.5 (release confirmation) and 5 (a successfully created claim is not submitted).

**Build status:** NOT BUILT — new flow sitting between the gauntlet (R-19) and the submission drawer (R-21).

**Acceptance criteria (typed truths):**

- `[ui]` the confirmation summarizes the claim content to be sent, the route, the environment/mode and every unresolved warning carried forward
- `[rbac]` only a release-authorized actor can confirm; coding or readiness access alone never enables it
- `[state]` after confirmation the claim renders as submission attempted — never as accepted, adjudicated or paid
- `[event]` the release records the actor, attestation, time and an immutable submission attempt
- `[absence]` a failed or refused release does not show a success banner and does not discard the prepared claim

### R-21 — Submission — a configured connector is not a production authority

**Task:** Submission drawer and route authority state  
**Hours:** 28

> As a biller or integration admin,  
> I want a submission drawer showing claim type/path, payer, clearinghouse route, environment/mode, authority state, readiness summary, immutable source revision, payload preview, preflight errors and the receipt,  
> So that I can see whether this route is actually authorized before anything leaves the building.

**Context:** Blueprint 7.6 (claim assembly and routing), 12.4 (external authority contract) and 14.3 (live payer transmission requires real route/enrollment/authority evidence; tests, configured adapters and an HTTP receipt do not supply it).

**Build status:** HELD — designable and buildable now, but the production Submit path stays held and labelled until route, enrollment and authority evidence exist per blueprint 14.3.

**Acceptance criteria (typed truths):**

- `[ui]` the drawer shows claim type/path, payer, clearinghouse/route, environment/mode, authority state, readiness summary and the immutable source revision
- `[state]` technical configuration, environment, credential custody, payer enrollment, clearinghouse acceptance, provider/facility identity, permitted transaction type, test versus production mode, evidence reference, expiry and operational owner render as separate facts, not one connected/disconnected flag
- `[absence]` a configured connector with a missing or expired authority record does not render a production-ready Submit; the control is held and names the missing dependency
- `[rbac]` raw transaction payload access is limited to authorized roles; others see an accessible redacted rendering
- `[ui]` the post-submit statement is limited to what was observed — submitted to connector, receipt received or response pending — and never "accepted" or "paid"
- `[ui]` retry is idempotency-aware: a retried attempt is visibly the same logical submission, not a second claim

### R-22 — Responses — correlated to their attempt, and overdue when silent

**Task:** Response and status tracking — correlation and silence  
**Hours:** 24

> As an RCM specialist,  
> I want every acknowledgment, status message and adjudication signal correlated to the submission attempt that produced it, with overdue silence called out,  
> So that I chase the claims the payer never answered instead of assuming they went through.

**Context:** Blueprint 7.7 (response and status tracking) and 12.2 (response correlation contract: claim id, attempt id, route, mode, control/reference ids, received time, raw-artifact custody reference, parser result, normalized state, retry/silence condition).

**Build status:** NOT BUILT as specified — claimStatusPoller.js supplies the data; the correlation-honest frontend is new.

**Acceptance criteria (typed truths):**

- `[state]` 999, 277CA, 276/277, portal or manual evidence and 835 are separately visible and each correlated to its originating attempt by control/reference id
- `[ui]` a rejection creates a corrective task with an owner; an accepted-for-adjudication claim remains visibly pending
- `[ui]` silence past the governing time rule renders as response overdue with a follow-up action, naming the rule applied
- `[evidence]` a parser failure renders as a parser failure with the retained raw artifact reference — not as an absent response
- `[absence]` transport success is never rendered as payer acceptance, and no response state is inferred from elapsed time alone

### R-23 — Denial queue — exception-first, with the priority open to inspection

**Task:** Denials and recovery — exception-first Work Next queue  
**Hours:** 20

> As an RCM specialist,  
> I want denials, rejections, underpayments and payer follow-up as one prioritized work queue with clustering of similar items,  
> So that recovery work is a governed queue with owners and deadlines rather than a report.

**Context:** Blueprint 7.9 (exception-first Work Next queue) and 10.3 (denial-to-prevention loop). Existing surface: packages/clinical-ui/src/pages/DenialsList/.

**Build status:** REDESIGN — an existing DenialsList exists; prioritization transparency and clustering are the new contract.

**Acceptance criteria (typed truths):**

- `[ui]` denials, rejections, underpayments and follow-up items appear in one queue with owner, deadline, recoverable amount and lane
- `[evidence]` similar items are clustered and the clustering basis is inspectable, including which items were excluded and why
- `[evidence]` the priority explanation is available per item and matches the ordering actually rendered
- `[ui]` an item with a deadline inside the configured risk window is visibly escalated with the governing timely-filing rule named

### R-24 — Denial detail — the payer's words, the system's hypothesis, kept apart

**Task:** Denial detail — lane, root cause hypothesis, deadline, evidence gaps  
**Hours:** 28

> As an RCM specialist,  
> I want the payer explanation and raw CARC/RARC alongside a clearly labelled root-cause hypothesis, the related claim, lines, contract variance and previous attempts, the recovery lane, the deadline clock, the owner and the evidence checklist with its gaps,  
> So that I can act on the denial without mistaking a system guess for the payer's reason.

**Context:** Blueprint 7.9 (denial detail contents; lanes: technical correction, clinical evidence, eligibility/authorization, underpayment/contract, manual investigation). Existing surface: packages/clinical-ui/src/pages/DenialDetail/.

**Build status:** REDESIGN — existing DenialDetail predates the hypothesis-labelling and evidence-gap requirements.

**Acceptance criteria (typed truths):**

- `[ui]` payer explanation and raw CARC/RARC/source response render together with a plain-language reading of each code
- `[evidence]` the root-cause hypothesis is labelled a hypothesis, carries its basis and confidence, and is visually separate from payer-supplied text
- `[ui]` related claim, lines, contract variance, previous attempts and the applicable payer rule are reachable from the detail
- `[ui]` the recovery lane, deadline clock, target appeal level, owner and escalation path are all present and editable only by authorized roles
- `[evidence]` the correction or appeal proposal shows its evidence checklist with the gaps stated explicitly rather than an implied-complete package

### R-25 — Appeal — generated is not sent, and sent is not won

**Task:** Appeal package — draft, recorded, sent, outcome  
**Hours:** 24

> As an RCM specialist assembling a recovery package,  
> I want an appeal package that stays a draft until external filing or delivery is independently confirmed, then records outcome, recovered amount and outcome source,  
> So that the ledger never counts an unsent appeal as recovery work completed.

**Context:** Blueprint 7.9 (an appeal package is a draft or recorded action until external filing/delivery is independently confirmed; "generated" is not "sent", "sent" is not "won") and 10.3.4.

**Build status:** NOT BUILT as specified — appealsFactoryEngine.js supplies generation; the draft/sent/outcome separation is the frontend contract.

**Acceptance criteria (typed truths):**

- `[state]` draft, recorded, sent/filed, won, lost, expired and closed are distinct states with distinct affordances
- `[absence]` generating a package never advances the state to sent; a sent state requires an independently confirmed external filing or delivery reference
- `[ui]` the outcome records recovered amount, outcome source and the date, and updates the ledger through the money model rather than a free-text note
- `[rbac]` only the authorized piece of the package is editable by each contributing role (specialist, coder, clinician), and each contribution is attributed
- `[evidence]` a prevention proposal produced from the outcome is governed — it never rewrites coding or payer rules automatically

## Wave 2 — Remittance, reconciliation and A/R before executive reporting (blueprint 16.3) — R-26…R-31

### R-26 — ERA inbox — file custody and parser outcome before any posting

**Task:** Remittance inbound queue — custody, parser outcome, duplicates  
**Hours:** 20

> As a payment poster,  
> I want an inbound remittance queue showing source file/transaction custody, parser outcome and duplicate status for each 835/ERA,  
> So that I can see what arrived and what could be read before any money moves.

**Context:** Blueprint 7.8 (remittance and reconciliation; inbound queue) and 13 (evidence custody).

**Build status:** NOT BUILT as specified — eraProcessingEngine.js supplies parsing; the custody-and-duplicate-honest queue is new.

**Acceptance criteria (typed truths):**

- `[ui]` each inbound item shows source file/transaction custody reference, received time, parser outcome and duplicate status
- `[state]` duplicate remit, unsupported transaction structure and parser failure are distinct review states, each with its own action
- `[absence]` a file that failed to parse never appears as an empty or zero-value remittance
- `[rbac]` raw 835 access is authorized-role only; the queue itself shows normalized custody metadata

### R-27 — Reconciliation — the balancing test is explicit, or the item is a review state

**Task:** Reconciliation detail — matching, amounts and the balancing test  
**Hours:** 32

> As a payment poster or RCM specialist,  
> I want matched claim and service lines with billed, expected, allowed, paid, patient responsibility, contractual adjustment, other adjustment and remaining balance, plus an explicit balancing test and exception reason,  
> So that a partially posted transaction can never display as reconciled.

**Context:** Blueprint 7.8 (reconciliation detail; ambiguous associations, unmatched lines, invalid totals, duplicate remits and unsupported structures are review states) and 12.3. Existing surface: packages/clinical-ui/src/pages/PaymentPostings/.

**Build status:** REDESIGN — existing PaymentPostings predates the explicit balancing test and review-state requirements.

**Acceptance criteria (typed truths):**

- `[ui]` matched claim and service lines render with every amount category from R-02 separately labelled
- `[money]` the balancing test result is displayed explicitly with its inputs; a failed balance names the exception reason
- `[state]` ambiguous association, unmatched line, invalid total, duplicate remit and unsupported structure each hold the item in a review state
- `[absence]` a transaction that posted partially never renders as reconciled, and no total is reduced twice across claim, patient and practice views
- `[ui]` a correcting or reversing entry renders in relation to the entry it corrects, not as an unrelated posting

### R-28 — Adjustment codes — plain language next to the raw code, never instead of it

**Task:** CARC/RARC/PLB plain-language rendering  
**Hours:** 12

> As a payment poster or RCM specialist,  
> I want every CARC, RARC and PLB rendered with a plain-language explanation alongside the raw code and detail,  
> So that staff understand the adjustment without losing the payer's exact assertion.

**Context:** Blueprint 7.8 (every CARC/RARC/PLB with a plain-language explanation plus raw code/detail).

**Build status:** NOT BUILT — new rendering shared by remittance, denial and patient-facing explanation surfaces.

**Acceptance criteria (typed truths):**

- `[ui]` each code renders with both its raw value and a plain-language explanation, with the code always visible
- `[absence]` an unmapped or unknown code renders as unknown with the raw value preserved, never as a generic "adjustment"
- `[rbac]` patient-facing renderings use the plain-language layer only and never expose payer-internal reasoning

### R-29 — Auto-posting — the proposal and the result are two different rows

**Task:** Automatic posting proposal versus posted result  
**Hours:** 20

> As a payment poster,  
> I want the automatic-posting proposal shown separately from what was actually posted, with what the automation did, what it proposes, why it stopped and how to undo it,  
> So that automation stays assistive and reversible instead of silently becoming the ledger.

**Context:** Blueprint 3 (automation is governed: show what automation did, what it proposes, why it stopped, who must decide, how to undo or correct it) and 7.8.

**Build status:** NOT BUILT — new governance surface over the deterministic posting engine.

**Acceptance criteria (typed truths):**

- `[ui]` the proposed posting and the posted result render as separate, comparable records
- `[evidence]` each automated action names its rule/version, confidence or deterministic basis, and why it stopped where it did
- `[ui]` an undo or correcting route exists for every automated posting and produces a correcting entry rather than a silent mutation
- `[rbac]` the decision to accept a held proposal is attributable to the authorized actor who made it

### R-30 — A/R aging — buckets are worklists, not a decorative chart

**Task:** A/R aging — clickable buckets as filtered worklists  
**Hours:** 20

> As an RCM specialist or finance user,  
> I want every aging bucket to open as a filtered worklist of owned work,  
> So that aging turns into action instead of a report someone reads once a month.

**Context:** Blueprint 7.10 (A/R aging buckets must be clickable, filtered worklists, not decorative charts).

**Build status:** NOT BUILT — new surface built on the financial table primitive.

**Acceptance criteria (typed truths):**

- `[ui]` each aging bucket is clickable and opens the filtered worklist whose rows compose it
- `[money]` bucket totals reconcile to their rows and to claim, patient and practice totals without reducing a balance twice
- `[a11y]` the aging visualization has a keyboard-navigable table equivalent with the same numbers

### R-31 — A/R item — why it is still open, and who moves it

**Task:** A/R item — balance composition, owner and next action  
**Hours:** 24

> As an RCM specialist,  
> I want each A/R item to show balance composition, last action, deadline, owner, payer versus patient responsibility split, proposed next action and the reason it remains open,  
> So that no receivable sits open without a named reason and a named owner.

**Context:** Blueprint 7.10 (every A/R item needs balance composition, last action, deadline, owner, responsibility split, proposed next action and the reason it remains open).

**Build status:** NOT BUILT — new surface.

**Acceptance criteria (typed truths):**

- `[ui]` the item shows balance composition, last action, deadline, owner, payer/patient split, proposed next action and the open reason
- `[money]` the payer and patient portions sum to the remaining balance and trace to their source postings
- `[ui]` the proposed next action routes into the corresponding recovery, statement or follow-up workflow with context preserved
- `[absence]` an item with no determinable next action renders as needing triage with an owner, not as an empty action cell

## Wave 3 — Contracts, expected reimbursement and integrity as a controlled Admin/Finance flow (blueprint 16.4) — R-32…R-34

### R-32 — Contracts — an ambiguous term stays ambiguous, never becomes a rate

**Task:** Contract intake to approval workflow  
**Hours:** 32

> As a finance or practice admin,  
> I want a source-to-approval workflow (upload/import, extract/map, review each change, acknowledge flags, apply version) showing effective dates, carve-outs, basis, rate source, page/snippet provenance, old-versus-new values, omitted codes and outlier flags,  
> So that a rate only enters pricing after a human has reviewed the change that produced it.

**Context:** Blueprint 7.11 (contracts need a separate source-to-approval workflow; an unreadable or ambiguous contract term must remain unreadable/ambiguous). Existing surface: packages/clinical-ui/src/pages/Contracts/.

**Build status:** REDESIGN — existing Contracts surface predates the staged approval workflow and provenance requirements.

**Acceptance criteria (typed truths):**

- `[ui]` the five stages render as an explicit, resumable workflow with a terminal applied or discarded state per version
- `[evidence]` each mapped term shows effective dates, carve-outs, basis, rate source and page/snippet provenance from the source document
- `[ui]` old-versus-new values, omitted codes and decimal-shift/outlier flags are shown per change and must be acknowledged before apply
- `[absence]` an unreadable or ambiguous term stays flagged as unreadable or ambiguous and is excluded from pricing — it never becomes an inferred rate
- `[event]` apply records the reviewer, the version applied and the effective window; a discarded version remains inspectable

### R-33 — Expected reimbursement — decision support, permission-isolated from coding

**Task:** Expected reimbursement and payment variance  
**Hours:** 24

> As a finance, admin or authorized RCM user,  
> I want expected allowed amounts and payment variance against the applied contract version, with no contracted rate shown honestly when none exists,  
> So that variance drives recovery without ever influencing code selection.

**Context:** Blueprint 3 (code to documentation), 7.11 (expected reimbursement is an admin/finance/authorized-RCM decision-support surface, visually and permission-wise isolated from point-of-care code selection) and 14 (payer outcomes are financial outcomes, not proof of coding correctness).

**Build status:** NOT BUILT as specified — contract pricing exists in the worker; the isolation and honest no-rate rendering are the frontend contract.

**Acceptance criteria (typed truths):**

- `[rbac]` expected reimbursement and contract rates are unavailable to clinical charge-capture and coding surfaces, enforced at the server and reflected in navigation
- `[money]` expected, allowed and paid render separately, and variance names the contract version and term it was computed against
- `[absence]` with no applicable contracted rate the surface shows no contracted rate plus the loading or review action — never a guessed or averaged amount
- `[ui]` a variance beyond the configured tolerance opens a recovery work item rather than only a report line

### R-34 — Integrity worklist — a review condition is not a conclusion

**Task:** Prepayment integrity worklist  
**Hours:** 20

> As an RCM specialist or compliance reviewer,  
> I want integrity findings (policy/edit issue, authority missing, potential duplicate, high risk, review required) with the trigger and scope explained,  
> So that review conditions are worked without the interface asserting a clinical or fraud conclusion.

**Context:** Blueprint 5.1 integrity family (explain the trigger and scope; do not label a clinical or fraud conclusion when the data only indicates a review condition) and 7.11. Existing surface: packages/clinical-ui/src/pages/IntegrityWorklist/.

**Build status:** REDESIGN — existing IntegrityWorklist predates the trigger-and-scope language requirements.

**Acceptance criteria (typed truths):**

- `[ui]` each finding names the trigger, the scope it applies to and the action available
- `[absence]` no finding is worded as a clinical determination or a fraud conclusion where the data indicates only a review condition
- `[evidence]` the finding opens its evidence drawer with rule identity, version and the data that matched
- `[rbac]` resolution and override are limited to authorized roles and are attributable

## Wave 4 — Standalone intake, mapping, identity exceptions and the two shells (blueprint 16.5, 11) — R-35…R-40

### R-35 — Intake inbox — what arrived, from where, and what failed

**Task:** Revenue intake inbox  
**Hours:** 20

> As a biller or integration admin,  
> I want an intake inbox with source, batch, received time, record count, source schema/version, success and failure counts and a safe downloadable error report,  
> So that an intake failure is visible work rather than an absent claim.

**Context:** Blueprint 7.1 (revenue intake UX needs) and 10.2 (standalone EHR-to-cash).

**Build status:** HELD — design and build now, labelled "integration design / not production ready" until the published foreign charge-event facade, source revision/idempotency contract and partner credentialing exist (blueprint 14.1).

**Acceptance criteria (typed truths):**

- `[ui]` each batch row shows source, batch id, received time, record count, source schema/version and success/failure counts
- `[ui]` the error report downloads in a form safe to share (no unnecessary protected content) and names each rejected record's reason
- `[state]` a partially accepted batch renders as partially accepted with both counts — never as received or as failed alone
- `[ui]` the surface carries the not-production-ready disclosure while the standalone contracts remain open

### R-36 — Mapping review — received field, target field, rule, result, provenance

**Task:** Mapping and validation review screen  
**Hours:** 28

> As an integration admin,  
> I want a mapping screen showing each received field, its normalized target field, validation result, provenance and the transformation rule applied,  
> So that a mis-mapped source field is caught before it becomes a claim.

**Context:** Blueprint 7.1 (mapping/review screen) and 12.1 (charge-event contract; each field needs present, unknown, not applicable and invalid semantics where those matter).

**Build status:** HELD — depends on the published charge-event contract in blueprint 12.1.

**Acceptance criteria (typed truths):**

- `[ui]` every received field renders with its target field, transformation rule, validation result and provenance
- `[state]` present, unknown, not applicable and invalid render as four distinct field states wherever the contract distinguishes them
- `[absence]` an unknown or not-applicable source value is never normalized into a default, a zero or an empty string
- `[ui]` a mapping change is a reviewed, versioned change with a read-back, not a live edit to in-flight records

### R-37 — Duplicates — three different states, three different messages

**Task:** Duplicate and idempotency feedback  
**Hours:** 12

> As an integration admin or biller,  
> I want "already received", "same event, different revision" and "possible duplicate" to be three distinct states with distinct actions,  
> So that a re-sent batch never silently creates a second claim and a real revision is not discarded as a duplicate.

**Context:** Blueprint 7.1 (strong duplicate/idempotency feedback) and 13 (concurrency/idempotency: durable deduplication/retry model and clear conflict UX).

**Build status:** HELD — depends on the source revision/idempotency contract (blueprint 14.1).

**Acceptance criteria (typed truths):**

- `[state]` already received, same event with a different revision, and possible duplicate render as three distinct states
- `[ui]` each state offers its own action: no-op with the existing record linked, revision review, or duplicate adjudication
- `[absence]` no duplicate state silently drops a record or silently creates a second claim; both outcomes are recorded and linked

### R-38 — Identity — an ambiguous suggestion is never a verified match

**Task:** Unresolved identity panel  
**Hours:** 16

> As an integration admin or biller,  
> I want an explicit unresolved-identity panel for patient, provider, facility and payer references that cannot be resolved with confidence,  
> So that staff cannot pick an ambiguous suggestion and proceed as if identity were verified.

**Context:** Blueprint 7.1 (explicit unresolved-identity panel) and 12.1 (patient reference and verification state).

**Build status:** HELD — depends on the RCM-owned patient/provider/payer identity model (blueprint 14.1).

**Acceptance criteria (typed truths):**

- `[ui]` unresolved patient, provider, facility and payer references are listed with the evidence for each candidate
- `[state]` verified, ambiguous and unresolved are distinct states carried through to the claim and its list rows
- `[absence]` selecting a candidate does not set verified status; verification requires its own evidence and is attributed
- `[ui]` a claim carrying an unresolved identity cannot reach release, and the block names identity as the reason

### R-39 — Integration status — first-class product area, honestly labelled

**Task:** Partner onboarding and integration status  
**Hours:** 20

> As an integration admin onboarding a partner EHR,  
> I want partner onboarding, credential state, adapter health, outbox/acknowledgment status and retry state as their own product area,  
> So that integration work is visible and owned instead of hidden inside a claim form.

**Context:** Blueprint 16.5 (design standalone intake, mapping, source-identity exceptions, partner onboarding and integration status as a first-class product area), 10.2.5 (controlled adapter/outbox with acknowledgments and retry states) and 14.1–14.2 (standalone RCM holds; the keyed rcm_claim lookup is a go-live blocker for higher-volume deployment).

**Build status:** HELD — build the surface, keep the production claim labelled not-production-ready until the standalone contracts and the keyed claim lookup land.

**Acceptance criteria (typed truths):**

- `[ui]` partner credential state, adapter health, outbox depth, acknowledgment status and retry state each render as their own fact
- `[state]` a returned correction or status update to an external source-of-record EHR shows sent, acknowledged, failed and retrying as distinct states
- `[absence]` no onboarding screen implies payer enrollment, production authorization or customer acceptance that the authority record does not carry
- `[ui]` the open standalone blockers, including the keyed claim lookup, are disclosed in-product where they constrain use

### R-40 — One core, two shells — the standalone product never pretends to have a chart

**Task:** Two shells — standalone navigation and embedded workspace  
**Hours:** 24

> As a product owner delivering both deployment shapes,  
> I want a standalone navigation (Home/Work Next, Revenue Intake, Readiness, Claims, Responses, Remittances, Denials and Appeals, A/R, Patient Financials, Contracts and Rates, Payers and Routes, Reports, Audit and Evidence, Administration) and an embedded role workspace over the same components,  
> So that both products share the revenue core without forking the experience.

**Context:** Blueprint 8.1, 8.2 and 11.3 (what is shared, configured or separate). A standalone user sees the source event where an embedded user sees the VAERION chart.

**Build status:** NOT BUILT — new shell work; the component layer beneath it is shared, not duplicated.

**Acceptance criteria (typed truths):**

- `[ui]` the standalone shell renders the full navigation above and opens on the exception-first Home, not a patient chart
- `[ui]` the embedded shell places RCM inside the existing role workspaces (front desk, provider, coding, billing/RCM, admin/finance, patient portal) using the same components
- `[ui]` claim, encounter, patient financial account and task deep-link to one another and return the user to the context they came from
- `[state]` in the standalone shell a foreign charge event renders as a source event — never as an encounter, and never with invented chart evidence
- `[rbac]` universal search returns claims, source events, remits, denials, contracts, payers and patients only within the signed-in role's tenant and minimum-necessary scope

## Wave 5 — Patient financials as a bounded view over the reconciled ledger (blueprint 16.6) — R-41…R-44

### R-41 — Patient account — one reconciled ledger, staff side

**Task:** Patient financial account — staff view  
**Hours:** 20

> As a front desk or RCM specialist,  
> I want a patient financial account showing responsibility by claim, posted payments, credits, open balance and the events that produced them,  
> So that staff answer a patient balance question from the reconciled ledger rather than an estimate.

**Context:** Blueprint 7.10 (A/R and patient financials) and 4.2 (payments, statements and patient financials consume reconciled results; they should not infer an amount from a prediction or an unposted remit).

**Build status:** NOT BUILT as specified — statements and payment surfaces exist; the single-ledger reconciliation contract is the new requirement.

**Acceptance criteria (typed truths):**

- `[money]` claim-level, patient-level and practice-level totals reconcile to the same postings without double reduction
- `[absence]` no patient balance is derived from a prediction, an estimate or an unposted remit; those render as separate, labelled figures
- `[ui]` every balance line traces to the posting, adjustment or statement event that created it
- `[rbac]` the account shows only this patient within the signed-in tenant, and access is audited

### R-42 — Patient statements — plain language, own record only

**Task:** Estimates and statements — patient-facing  
**Hours:** 24

> As a patient or authorized proxy,  
> I want statements, estimates and EOB-oriented explanations in plain language, with the basis, exclusions and expiry of any estimate stated,  
> So that I understand what I owe without being shown payer-internal reasoning or anyone else's information.

**Context:** Blueprint 6 (patient/proxy role), 7.2 (a patient-responsibility estimate clearly labelled as estimate, its basis, exclusions, confidence/missingness and expiry), 7.10 (plain language; never expose payer-internal reasoning) and 13 (patient phone layouts independently designed). Existing surface: packages/clinical-ui/src/pages/Statements/.

**Build status:** REDESIGN — existing Statements surface predates the estimate-labelling and phone-layout requirements.

**Acceptance criteria (typed truths):**

- `[ui]` an estimate is labelled an estimate and shows its basis, exclusions, missing inputs and expiry
- `[absence]` no fabricated $0 copay, deductible or balance is presented as a payer answer; an unavailable coverage answer says so
- `[rbac]` the surface exposes only the signed-in patient's own record and never payer-internal reasoning or worklist content
- `[a11y]` the phone layout is independently designed and fully operable with assistive technology
- `[money]` the statement figures reconcile to the same ledger the staff account shows, to the cent

### R-43 — POS collection — a receipt is a payment-rail result, not a toast

**Task:** Point-of-service collection and receipt  
**Hours:** 16

> As a front desk operator,  
> I want to take a point-of-service payment and see a result that reflects what the payment rail actually returned,  
> So that no collection is recorded that the rail did not confirm.

**Context:** Blueprint 7.10 (POS collection), 3 (fail closed for money and authority) and 13 (no deceptive completion).

**Build status:** NOT BUILT as specified — the confirmed-result requirement is the new contract.

**Acceptance criteria (typed truths):**

- `[state]` authorized, captured, declined, pending and failed render as distinct states carrying the rail's own reference
- `[absence]` no success toast, receipt render or ledger posting occurs without the external payment reference
- `[ui]` a failed or interrupted payment retains the attempt with a retry route and never leaves an orphan posting
- `[money]` the posted amount appears in the patient ledger with its source and time and reconciles to the statement

### R-44 — Refunds — dual control, external reference, no generic completion toast

**Task:** Refunds and credit balances with dual control  
**Hours:** 20

> As a finance or practice admin,  
> I want refund and credit-balance handling with the configured dual control, audit, external payment reference and recovery states,  
> So that money leaving the practice carries the same evidence standard as money arriving.

**Context:** Blueprint 7.10 (refunds and credit balances require configured dual control, audit, external payment reference and recovery states; no generic "refund complete" toast substitutes for a payment-rail result). Existing surface: packages/clinical-ui/src/pages/Refunds/.

**Build status:** REDESIGN — existing Refunds surface predates the dual-control and rail-result requirements.

**Acceptance criteria (typed truths):**

- `[rbac]` initiation and approval require two distinct authorized actors, both recorded
- `[absence]` no completion state is shown without the external payment reference returned by the rail
- `[state]` initiated, approved, sent, settled, failed and reversed render as distinct states with their own recovery actions
- `[event]` every step emits an audit event carrying actor, amount, source credit and reference

## Wave 6 — Analytics, audit, governed learning, claim-path extensions and cross-cutting acceptance (blueprint 16.7, 12, 13, 15) — R-45…R-50

### R-45 — Reports — every metric names its cohort, denominator and freshness

**Task:** Operational and financial metrics with stated denominators  
**Hours:** 24

> As a practice admin, finance user or owner,  
> I want clean submission rate, first-pass acceptance, denial rate by payer/reason/service, overturn and recovery rate, days in A/R, net collection, underpayment identified and recovered, charge lag, timely-filing risk, reconciliation exceptions, staff time on task and automation outcomes,  
> So that performance is measurable without a metric quietly changing its own definition.

**Context:** Blueprint 7.12 (core measures; every metric view needs cohort, source, period, denominator, inclusion/exclusion rules and data freshness) and 16.3 (reports must drill into actionable source work).

**Build status:** NOT BUILT as specified — dashboard routes exist in rcmOrchestrator.js; the definition-transparent, drillable reporting layer is new.

**Acceptance criteria (typed truths):**

- `[ui]` every metric view states cohort, source, period, denominator, inclusion/exclusion rules and data freshness
- `[ui]` every metric drills into the source rows that produced it, and those rows sum to the displayed value
- `[absence]` a payer payment is never presented as evidence that the original code was clinically correct
- `[a11y]` each chart has a keyboard-navigable table equivalent carrying the same values

### R-46 — Audit explorer — durable evidence, reviewable without over-disclosure

**Task:** Audit and evidence explorer  
**Hours:** 20

> As a compliance officer or owner,  
> I want to search and review audit events, evidence custody references and material state changes across claims, remits, contracts and recovery work,  
> So that I can reconstruct what happened and who did it, within the tenant boundary.

**Context:** Blueprint 7.12 (audit), 13 (audit, evidence custody, tenant isolation) and 15.3.

**Build status:** NOT BUILT — new surface consuming the audit event primitive (R-09).

**Acceptance criteria (typed truths):**

- `[rbac]` results are tenant-bound and role-checked, including aggregate counts
- `[event]` material state changes and protected-detail reads are both discoverable, with actor, time, object, action and correlation
- `[absence]` the explorer surfaces custody and hash references rather than protected payloads; export applies the same minimization
- `[ui]` a compliance role can inspect everything in scope and change nothing

### R-47 — Learning — an evidence ledger, not an "AI learned" badge

**Task:** Governed learning ledger  
**Hours:** 20

> As an admin or compliance reviewer governing automation,  
> I want each proposed rule or model change shown with source provenance, evaluated cohort, measurement, calibration or unknown status, tenant/privacy boundary, approver, promotion state and rollback path,  
> So that no rule promotes itself into production behavior.

**Context:** Blueprint 7.12 (learning views are evidence ledgers) and 14.5 (prediction, automated denial correction and learning remain governed assistance; no self-promotion or autonomous clinical/coding change).

**Build status:** NOT BUILT — new governance surface.

**Acceptance criteria (typed truths):**

- `[evidence]` each proposal shows source provenance, evaluated cohort, measurement, calibration or explicit unknown, and the tenant/privacy boundary it was evaluated within
- `[state]` draft, evaluated, approved, promoted and rolled back are distinct states with named approvers
- `[absence]` no proposal can promote itself, and no promotion alters coding or clinical behavior without the separate authority that governs it
- `[ui]` every promoted change exposes its rollback path and the outcomes observed since promotion

### R-48 — Claim paths — supported, scoped or held, stated in the product

**Task:** Claim-path profiles and capability disclosure  
**Hours:** 16

> As a biller working non-professional claims,  
> I want institutional, dental, pharmacy, workers' compensation, secondary/COB, attachment and specialty paths represented as explicit profiles with their own requirements,  
> So that an unsupported path is disclosed rather than silently handled as a professional claim.

**Context:** Blueprint 7.6 (specialty paths must not masquerade as office/professional claims), 14.4 (the UX must disclose supported path, scope and held capabilities rather than promise universal billing) and 16.7.

**Build status:** NOT BUILT — new configuration-aware disclosure layer across claim creation, gauntlet and submission.

**Acceptance criteria (typed truths):**

- `[ui]` each claim path renders as its own profile with its required fields, checks and route requirements
- `[absence]` no institutional, dental, pharmacy, workers' compensation, secondary or attachment claim is processed through the professional path by default
- `[state]` a path that is configured but not authorized, or not supported at all, renders as held or unsupported with the reason
- `[ui]` the acceptance matrix for the tenant's enabled paths is visible to authorized users

### R-49 — Accessibility — colour never carries meaning, and timers are operable

**Task:** Accessibility conformance across tables, timers and dialogs  
**Hours:** 24

> As a staff member using assistive technology,  
> I want tables, status, timers, errors, dialogs, charts and bulk actions to be keyboard and screen-reader operable across the product,  
> So that the product is usable independently of colour vision, pointer use or screen size.

**Context:** Blueprint 13 (accessibility: staff desktop/tablet and patient phone layouts are independently designed; colour never carries sole meaning).

**Build status:** NOT BUILT — cross-cutting acceptance pass over every wave-0 primitive and every surface consuming it.

**Acceptance criteria (typed truths):**

- `[a11y]` tables, status chips, deadline timers, errors, dialogs, charts and bulk actions are fully keyboard operable with a visible focus order
- `[a11y]` every state, including error, blocked and overdue, is conveyed by text and icon in addition to colour
- `[a11y]` staff desktop/tablet and patient phone layouts are independently laid out, not a single scaled breakpoint
- `[a11y]` live regions announce state changes (submission attempted, response received, posting held) without stealing focus

### R-50 — Acceptance — one synthetic journey exercising every honesty rule

**Task:** Acceptance journey — tenant isolation, handoffs and recoverable states  
**Hours:** 16

> As a product owner accepting the frontend,  
> I want a synthetic, tenant-isolated journey that proves a role finds its work, acts, hands off, and that errors and unknowns stay honest,  
> So that acceptance rests on a demonstrated journey rather than a screen-by-screen review.

**Context:** Blueprint 15 (UX acceptance criteria, all ten clauses) and 10 (key user journeys).

**Build status:** NOT BUILT — the acceptance harness for the frontend; runs the journey against the chosen deployment shape with external authority explicitly held.

**Acceptance criteria (typed truths):**

- `[ui]` each role reaches its correct work item from its intended starting point, and the UI states current state, source, blocker, owner and next action
- `[ui]` an authorized action saves with source and audit attribution and reads back after refresh and fresh sign-in
- `[rbac]` an unauthorized role and a cross-tenant actor are denied both visually and at the server within the same journey
- `[ui]` a receiver sees each downstream handoff and can act on it or return it with a reason
- `[state]` error, partial-source, conflict, duplicate, retry, empty and unavailable states are each exercised and none loses operator work
- `[money]` line, claim, patient and practice totals reconcile to their displayed source rows at the end of the journey
- `[absence]` external authority remains explicitly held wherever separate current evidence does not exist

## Basis of estimate

Hours are frontend engineering hours: the surface, its states (loading, empty, blocked, unavailable, permission denied, conflict, error), its tests, and one pass proving its typed truths. They exclude backend work, payer enrollment and design research.

Sizing anchors: shared primitive 8–24h · worklist surface 16–28h · detail workbench 24–32h · multi-stage configuration flow 28–32h · patient-facing surface 16–24h · cross-cutting acceptance pass 16–24h. Stories that consume a wave-0 primitive are costed for their own composition only; the primitive's cost sits in wave 0.

## What this backlog deliberately excludes

- Backend and worker changes (`claimsRoutes.js`, `rcmOrchestrator.js`, `eraProcessingEngine.js`, `claimStatusPoller.js`, `appealsFactoryEngine.js`, `transport/`) — this backlog specifies the frontend contract against them and names where that contract is missing.
- The standalone go-live blockers themselves: the published foreign charge-event facade, the RCM-owned identity model, the source revision/idempotency contract, partner credentialing and the keyed `rcm_claim` lookup (blueprint 14.1–14.2). Wave 4 designs against them and discloses them; it does not close them.
- External authority: payer enrollment, clearinghouse production acceptance and route authorization (blueprint 14.3). No row here claims live payer transmission.
- Clinical, coding and CDI authoring surfaces — RCM consumes their output and returns source-bound requests (blueprint 4.2).

## Sources

- `VAERION_RCM_Product_and_UX_Blueprint_2026-09-16.pdf` — sections 5 (state model), 6 (roles and authority), 7.1–7.12 (the twelve components), 8 (information architecture), 9 (shared UX primitives), 10 (journeys), 12 (data contracts), 13 (safety and trust), 14 (current truth and deliberate holds), 15 (UX acceptance criteria), 16 (design-delivery order).
- Repository evidence named in blueprint 17, including `packages/clinical-ui/src/pages/{ClaimsList,ClaimDetail,DenialsList,DenialDetail,PaymentPostings,ReadyToBill,Contracts,IntegrityWorklist,Refunds,Statements}/` — the surfaces marked `REDESIGN`.
- `VAERION_User_Storyv2_1.xlsx` (Story A) — the story and summary format this backlog follows.
