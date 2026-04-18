# Strangler Fig — Phase 2: Incremental Modernization

> Instead of blowing up the old system all at once, you build new services around it. You slowly reroute traffic to the new pieces until the old system is completely surrounded — and strangled. Then you turn it off.

**System:** Billing Service | **Phase:** Active modernization | **Input:** locked `spec.md`

---

## Why Not a Big-Bang Rewrite

A big-bang rewrite means: build the whole new system in parallel, then cut over on day X.

Day X always arrives before the new system is ready. The team is exhausted. The new system has unknown bugs because it was never under real load. The cutover is delayed. Scope creeps in. Eventually the project is either abandoned or shipped in a broken state that takes months to stabilize.

The Strangler Fig Pattern eliminates day X entirely.

```
Week 1:   [Legacy Billing] handles 100% of traffic
Week 4:   [New Invoice Service] handles invoice generation — legacy handles everything else
Week 8:   [New Tax Service] handles tax calculation — routed through new invoice service
Week 14:  [New Reconciliation Service] handles payments — legacy handles only dunning
Week 18:  [Legacy Billing] handles 0% of traffic — turned off
```

At no point is the entire system at risk. Each new service is small, independently deployable, and validated against the spec before traffic is rerouted to it. Rollback is always one routing change away.

---

## The AI-Assisted Rebuild Pipeline

With a locked `spec.md`, AI agents become spec-aware assistants. They are not guessing intent — intent is written down. They are executing against a verified record.

```
spec.md (locked)
     ↓
[Agent: Architect]   reads spec → produces architectural plan + data contracts
     ↓
[Agent: Planner]     reads plan → breaks into PR-sized atomic tasks
     ↓
[Agent: Implementer] reads one task + relevant spec clauses + contracts → implements
     ↓
[Agent: Validator]   runs contract tests + golden tests → confirms conformance
     ↓
Route traffic to new service → next module
```

Each agent is given exactly what it needs and nothing more. This is the same incremental disclosure principle from [`../../GUIDED_EXECUTION.md`](../../GUIDED_EXECUTION.md) — applied to a modernization context.

---

### Phase 2A — Architectural Plan and Data Contracts

The first agent reads `spec.md` in full and produces two outputs:

**1. Architectural plan**

```markdown
## Billing Service — Modernization Plan

### Target architecture
- InvoiceService     → handles INV-01, INV-02 (generation, proration)
- TaxService         → handles INV-03 (tax calculation, legacy path)
- DunningService     → handles INV-04 (dunning emails, suspension)
- ReconciliationService → handles INV-05 (nightly payment reconciliation)

### Routing strategy
- API Gateway intercepts billing requests
- Feature flag per module routes traffic to legacy or new service
- Legacy service remains live until all modules pass validation
```

**2. Data contracts** — locked before any implementation begins

```markdown
## api-contract.md — TaxService

Input:
  accountId:    string (UUID)
  createdAt:    ISO-8601 date
  region:       enum [US_EAST, US_WEST, EU, APAC]
  invoiceType:  enum [STANDARD, CREDIT, ADJUSTMENT]
  baseAmount:   decimal (2dp, positive)

Output:
  taxAmount:    decimal (2dp)
  rateApplied:  decimal (4dp)
  ratePath:     enum [LEGACY, STANDARD]

Invariant refs: INV-03, SCN-07, SCN-08
```

**Critical rule:** Data contracts are locked before Phase 2B begins. No implementation starts without a signed-off contract. A contract change requires a spec update first.

See [`../agent-os/example-java/api-contract.md`](../agent-os/example-java/api-contract.md) for a worked example of a locked data contract.

---

### Phase 2B — Atomic Tasks

The planner agent reads the architectural plan and breaks it into tasks. Each task is:
- One objective
- One to two files touched
- A clear acceptance criterion from the spec
- PR-sized — reviewable in under an hour

```markdown
## tasks.md — TaxService implementation

TASK-01  Scaffold TaxService project structure
  Files: TaxService.java, TaxServiceTest.java
  Done when: project compiles, no logic yet

TASK-02  Implement standard tax rate calculation
  Files: TaxService.java, TaxServiceTest.java
  Spec refs: INV-03 (standard path), SCN-09, SCN-10
  Done when: SCN-09 and SCN-10 characterization tests pass against new service

TASK-03  Implement legacy tax rate path for pre-2014 accounts
  Files: TaxService.java, TaxServiceTest.java
  Spec refs: INV-03 (legacy path), SCN-07
  Done when: SCN-07 characterization test passes against new service

TASK-04  Implement credit invoice exemption from legacy path
  Files: TaxService.java, TaxServiceTest.java
  Spec refs: SCN-08
  Done when: SCN-08 characterization test passes against new service

TASK-05  Contract test — validate TaxService output against api-contract.md
  Files: TaxServiceContractTest.java
  Done when: all contract assertions pass for all region × invoice-type combinations
```

---

### Phase 2C — Implementation

Each implementer agent receives:

```
You are implementing TASK-03.

Your inputs:
- spec.md (read INV-03 only — do not read beyond your task scope)
- api-contract.md (TaxService section)
- TASK-03 description above

Your constraints:
- Do not change any behavior not described in INV-03
- Do not modify the data contract
- Do not add new fields to the input or output
- Your task is done when SCN-07 passes. Not before.
```

The agent cannot drift from intent. The spec clause is the definition of correct. The scenario is the test. There is nothing to guess.

---

## Validation as Definition of Done

In a classic rewrite, done means "it compiles and the demo looked fine."

In spec-driven modernization, done is a provable claim. A service is ready to receive production traffic only when three validation tiers pass.

---

### Tier 1 — Contract tests

Prove the new service's API matches the signed-off contract exactly.

```java
@Test
void tax_service_output_matches_contract_for_legacy_account() {
    TaxRequest request = TaxRequest.builder()
        .accountId("acct-001")
        .createdAt(LocalDate.of(2013, 11, 15))
        .region(Region.US_WEST)
        .invoiceType(InvoiceType.STANDARD)
        .baseAmount(new BigDecimal("1000.00"))
        .build();

    TaxResponse response = taxService.calculate(request);

    // Contract: output fields exist and have correct types
    assertThat(response.getTaxAmount()).isNotNull();
    assertThat(response.getRateApplied()).isNotNull();
    assertThat(response.getRatePath()).isEqualTo(RatePath.LEGACY);

    // Spec: INV-03 + SCN-07
    assertThat(response.getTaxAmount()).isEqualByComparingTo("87.50");
    assertThat(response.getRateApplied()).isEqualByComparingTo("0.0875");
}
```

---

### Tier 2 — Golden tests

Compare new service output against known-good output from the legacy system for the same inputs.

```java
@Test
void new_tax_service_matches_legacy_for_all_region_account_age_combinations() {
    List<TaxTestCase> goldenCases = GoldenDataset.loadTaxCases(); // 2,400 real account snapshots

    for (TaxTestCase tc : goldenCases) {
        TaxResponse legacy = legacyBillingService.calculateTax(tc.toRequest());
        TaxResponse modern = taxService.calculate(tc.toRequest());

        assertThat(modern.getTaxAmount())
            .as("Case %s: tax amount mismatch", tc.getId())
            .isEqualByComparingTo(legacy.getTaxAmount());
    }
}
```

Golden tests catch the cases the spec didn't explicitly document — real production inputs that expose edge cases no human thought to write scenarios for.

---

### Tier 3 — Traffic replay

Run a sample of real production traffic through both the old and new service simultaneously. Compare outputs.

```
Replay run: 2025-07-01 | Sample: 5% of last 30 days | Cases: 12,400 invoices

MATCH     INV-standard-monthly         12,104 / 12,104   100%
MATCH     INV-proration-upgrade            187 / 187      100%
MISMATCH  INV-legacy-tax-US-WEST            7 / 109       6.4%  ← investigate
MATCH     INV-credit-invoice               98 / 98        100%
```

The 6.4% mismatch on legacy tax for US-WEST is not a bug in the new service. It surfaces an undocumented edge case: accounts in US-WEST with a specific county code use a sub-rate that was never in the spec. The legacy system had it. Nobody knew.

**This is exactly what traffic replay is for.** It finds the behavior the spec missed. The response: update the spec, add a scenario, add a characterization test, update the implementation. Then replay again.

---

## The Conformance Mindset

| Old definition of done | Spec-driven definition of done |
|---|---|
| It compiles | Contract tests pass |
| The demo worked | Golden tests match legacy for 2,400+ cases |
| QA signed off | Traffic replay shows ≥ 99.9% match |
| We deployed it | Spec clauses are traceable to passing tests |

The goal is not completion. The goal is conformance.

Conformance means: the new service's behavior is provably equivalent to what is defined in the spec, which is provably equivalent to the behavior the legacy system was exhibiting.

You are not building the system you wish you had. You are building the system you have — but clean, understood, and maintainable.

---

## Rerouting and Shutdown

Once a module passes all three validation tiers:

1. **Shadow mode** — new service runs in parallel, receives traffic, produces output, but legacy output is what's actually used. Differences are logged.
2. **Canary** — 5% of traffic routed to new service. Monitor for 48 hours. Zero regression → proceed.
3. **Progressive rollout** — 25% → 50% → 100% over one week. Legacy remains on standby.
4. **Legacy shutdown** — only after 100% traffic has run on the new service for 30 days without regression.

The old Billing Service is not deleted immediately. It is decommissioned: traffic removed, dependencies documented, code archived. It is kept for 90 days in case of an unforeseen rollback need. Then it is gone.

---

## Checklist — Phase 2 Complete

- [ ] Architectural plan reviewed and approved
- [ ] All data contracts locked and signed off before implementation
- [ ] Each module implemented from atomic tasks with spec clause references
- [ ] Contract tests pass for all modules
- [ ] Golden tests pass against ≥ 2,000 real production cases per module
- [ ] Traffic replay shows ≥ 99.9% match across all invoice types
- [ ] Each mismatch from replay is either resolved or documented as a spec update
- [ ] Each module has completed shadow → canary → progressive rollout
- [ ] Legacy service has run at 0% traffic for 30 days

**Phase 2 is done when conformance is proven — not when the new system feels ready.**
