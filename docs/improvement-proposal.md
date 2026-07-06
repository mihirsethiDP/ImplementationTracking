# Improvement proposal — align the Adoption Console with the CloseTheLoop v2 model

Context taken from `D:/Roles` (CloseTheLoop v2 Roles & Permissions / "Role Studio").
That project defines the **product-module entitlement model** our adoption tracker
is currently missing.

---

## The core insight: we're measuring the middle layer against the wrong denominator

There are **three layers** to "is a feature actually being used at a plant":

| Layer | Question | Source of truth | Status in this tool |
|---|---|---|---|
| **1. Entitlement** | Is the plant *licensed* for it? | Roles v2 `PLANTMODS` / contract record | ❌ Missing |
| **2. Provisioning** | Is the feature flag *turned on*? | The Google Sheet (today) | ✅ This is all we track |
| **3. Usage** | Is anyone *actually using* it? | PostHog (proposed earlier) | ⏳ Planned |

Right now a plant reading **"28% adopted"** is misleading: some of those "off"
features were **never sold to that plant**. You can't fairly score an
implementation team against features the customer didn't buy. The Role Studio
rule — **effective access = user permission ∧ plant module** — has a direct twin
for us:

> **effective adoption = feature enabled ∧ plant licensed**

Applying it splits every "off" feature into two very different buckets:

- **Licensed but OFF** → a real **adoption gap** → action for Customer Success / Implementation.
- **Not licensed** → not a gap at all → an **upsell opportunity** → action for Sales.

This single change makes the whole dashboard honest and gives each number an owner.

---

## Recommendation 1 (P1) — Adopt the 6 product modules as the primary taxonomy

Our `client / operator / common` grouping is ad-hoc. Role Studio already has the
**authoritative** taxonomy: **8 modules** licensed per plant (Core, Issue Resolution,
Tasks, Data, Analytics, IoT, **Inventory**, **Floc Detector**). Regroup around it so
both tools speak the same language.

### Map at the PERMISSION level, not the feature level

A first-pass "feature → one module" table is wrong, because in v2 **modules cut
across permission sets** — each *permission* carries its own `mod:` tag (`PERMMOD`),
and one adoption-tool "feature" is really a **bundle of permissions**. So a feature
is licensed only if **every module its permissions touch** is licensed:

> **feature available at a plant = AND, over each permission the feature needs, of (that permission's module is licensed)**

Rigorous mapping (each feature → its v2 permission home(s) → module(s), verified
against `SETS` and `coverage-map.csv`):

| Adoption feature | v2 permission home(s) | Module(s) | v2 status |
|---|---|---|---|
| Dashboard (basic view) | `readplant.dash` | **core** | Always licensed (ships with every contract) |
| Insights | `readplant.insights` | **analytics** | Clean |
| Reports | `readplant.reports` | **analytics** | Clean |
| Events | `readplant.insights` (view); `tech.sitetpl` (config) | **analytics** (+core to configure) | Clean; `Events_View_R` was a GAP-FIX |
| Tickets | `work.sessions`, `work.raise` (+`approve.*` to close) | **ops** | Clean |
| Task List | `work.tasks`, `work.myshift` (+`approve.assign`) | **tasks** | Clean |
| Data Input | `work.data` | **data** | Clean |
| Remote Control | `flags.remote` | **iot** | Exception flag — always audited |
| **OCR Data Input** | `work.data` **+ OCR capture** | **data + (net-new AI/OCR)** | Cross-module; OCR sub-feature is **net-new** (not in the 121) |
| **Insight Digest (WhatsApp)** | `readplant.insights` **+ notification delivery** | **analytics + (notifications)** | Cross-module; delivery channel is a platform capability, not a mapped permission |
| **Dashboard Summary (WhatsApp)** | `readplant.dash`/`reports` **+ notification delivery** | **core/analytics + (notifications)** | Cross-module, same as above |
| **Inventory** | `work.inventory` (operators) + `approve.invlogs` (supervisors) | **inv** (Inventory Management) | **Resolved 2026-07** — dedicated module, role-split; both perms net-new (like APPR). Supersedes the old Phase-2 deferral |
| **Floc Detector** | *none — `noperm:true`* | **floc** (hardware add-on) | **Resolved 2026-07** — permissionless hardware SKU; pure contract entitlement, never grants/caps anything user-side |
| **Visualisation** | `Visualisation_View_R` → **RETIRED** | — | **Superseded by plant dashboard** — drop/merge into Dashboard (like Maintenance was dropped) |

> **Update:** the Roles repo now has **8 modules** (added `inv` and `floc`). Two of
> my three "no v2 home" features are now resolved; only **Visualisation (RETIRED)**
> remains out of scope. This is also proof the mapping must be **config, not code** —
> the taxonomy changed twice in a day.

### Consequences for the design (these are the "gotchas" the naive table hid)

1. **Core-housed features (basic Dashboard) are always licensed** — never an upsell,
   pure adoption plays. Don't let them inflate the "not licensed" bucket.
2. **Cross-module features need an AND** — a WhatsApp digest is only truly available
   if *both* Analytics **and** the notification capability are licensed. Single-module
   logic would over-report availability.
3. **Floc introduces an "entitlement-only" feature archetype.** It has no permission
   and no admin-toggled provisioning step — its whole story is "is the hardware sold
   & installed." So for Floc, *licensed = adopted*; the "licensed-but-off gap" concept
   doesn't apply. Track a **hardware-health / online** signal in place of the usage
   layer. This is the pattern for every future hardware SKU, so the tool should model
   feature *types*: `permission-gated` (most), `entitlement-only` (Floc), `cross-module`.
4. **Inventory is role-split** (`work.inventory` for operators, `approve.invlogs` for
   supervisors). Track adoption at the `inv` module level, but the two capabilities can
   drift independently — worth two sub-rows once entitlement data exists.
5. **Only remaining open items:** WhatsApp delivery's home (notification capability),
   and whether OCR capture is its own net-new sub-feature — a 15-min product sync.
   `Floc` and `Inventory` are now settled.

Benefit: "Adoption by area" becomes "Adoption by module," lined up with how plants
are actually licensed and sold — and honest about what v2 doesn't yet cover.

---

## Recommendation 2 (P1) — Entitlement-aware adoption maths

- Add a per-plant module-entitlement input (see "Where the data comes from").
- Change denominators: **coverage = enabled ÷ licensed**, not ÷ 14.
- New headline KPIs:
  - **Adoption gap** = licensed-but-off features (the actionable number).
  - **Upsell surface** = not-licensed features that peers in the same segment use.

---

## Recommendation 3 (P2) — Three-state cells, reusing the Role Studio lock pattern

Role Studio already renders a 🔒 lock on abilities a plant isn't licensed for.
Reuse it verbatim so the two products feel like one:

- 🟢 **On** — licensed & enabled
- ⚪ **Off** — licensed, not enabled (adoption gap; highlight in the Von Restorff accent)
- 🔒 **Locked** — not licensed (dimmed; hover → "Not in this plant's plan — upsell")

---

## Recommendation 4 (P2) — Make it an *implementation lifecycle* tracker, not a flag snapshot

The repo is called **Implementation Tracking**, but today it's a point-in-time
flag view. Add a lifecycle status per plant × module that reflects a real rollout:

`Not started → Licensed → Provisioned (flag on) → Adopted (used) → Healthy`

This turns the tool into an onboarding cockpit for the implementation team: filter
to "Licensed but not Provisioned" to get this week's rollout worklist. Pairs
naturally with the goal-gradient UI already built.

---

## Recommendation 5 (P3) — Usage layer via PostHog

As discussed previously: PostHog `capture()` in the product app fills layer 3.
With entitlement in place, the strongest view becomes the **funnel per module**:
Licensed → Provisioned → Active-in-30d. Drop-off between any two stages is a
precise, ownable action item.

---

## Where the entitlement data comes from

Options, cheapest first:

1. **New columns in the existing sheet** — one boolean column per module per plant.
   Fastest; keeps the current Apps Script contract. Manually maintained by CS.
2. **Join to a contract/entitlement sheet** — a separate `Entitlements` tab keyed
   by plant, mirroring Role Studio's `PLANTMODS`. Cleaner separation.
3. **Pull from the real backend** once v2 ships — entitlements live on the
   plant/contract record (per the Roles backend contract), so this tool reads the
   same source the runtime permission check uses. Best long-term.

Start with option 1 or 2 now; migrate to 3 when v2 lands.

---

## Design consistency (low effort, high polish)

Role Studio uses navy `#193458` + ops teal `#0E7C66` + admin purple `#5548C8`,
progressive disclosure, and plain-language labels. Our redesign already shares the
navy/teal palette — adopting the lock pattern and module names makes the adoption
console and Role Studio read as two views of one system.

---

## Suggested phased rollout (trackable)

- [x] **Phase 0 — Align taxonomy**: feature→module mapping grounded in `SETS`/`coverage-map.csv`; 2 open items remain (WhatsApp delivery, OCR sub-feature).
- [x] **Phase 1 — Entitlement-aware UI** *(shipped, entitlements mocked)*: config-driven mapping, 8 modules, three feature types, four cell states, coverage = adopted ÷ licensed, Gap + Upsell KPIs, module grouping.
- [x] **Phase 2 — Implementation lifecycle** *(shipped, health mocked)*: Not started → In progress → Live → Healthy pipeline, per-plant stage, rollout worklist, plant-page stepper + next-best-action.
- [ ] **Phase 3 — Wire real entitlement source**: replace the mocked `entitlement()` (and later `mockHealthy()`); decide sheet columns vs `Entitlements` tab vs v2 backend.
- [ ] **Phase 4 — Usage** (depends on product instrumentation): PostHog funnel per module → real `Healthy` stage.
