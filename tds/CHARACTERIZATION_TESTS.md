# CHARACTERIZATION_TESTS.md

> Scenario tests ask: does the system do what the spec says?
> Characterization tests ask: what does the system actually do right now?
> The direction is reversed — and that reversal is everything.

---

## The Situation

You are working with a system that has no spec. Or the spec exists but it has drifted so far from reality that it cannot be trusted. Or you are about to make a significant change to a system that has no test coverage at all.

You cannot write scenario tests yet — scenario tests are derived from a spec, and you don't have one you can trust.

You cannot write property-based tests yet — those require a clear statement of what the system must always do, and you don't know that either.

What you can do is capture what the system currently does and lock that behavior in place before you touch anything. That is a characterization test.

---

## The Fundamental Difference

| Test type | Direction | Asserts | What failure means |
|---|---|---|---|
| **Scenario test** | Spec → Test | Correctness — "the system does what was intended" | The code doesn't meet the requirement |
| **Property test** | Spec → Test | Universal truth — "this rule holds for any input" | An invariant was violated |
| **Characterization test** | Behavior → Spec candidate | Consistency — "the system still does what it did before" | Behavior changed — intentionally or not |

A characterization test that fails does not mean a bug was found. It means behavior changed. That is a decision point: either the change was intentional and the spec needs to be updated to reflect it, or it was unintentional and needs to be reverted.

---

## The Three Times You Need Them

### 1. Before touching legacy code with no test coverage

You cannot safely refactor or extend code you have no tests for. Characterization tests create a behavioral safety net. You capture current output and lock it in before you change anything.

### 2. Before a modernization migration

Before you replace a legacy module with a new one, you need proof that the new module produces identical outputs for identical inputs. Characterization tests run against both systems and serve as the conformance gate.

See [`examples/legacy-modernization/STRANGLER_FIG.md`](examples/legacy-modernization/STRANGLER_FIG.md) for how golden tests and traffic replay build on this concept at scale.

### 3. Before retrofitting a spec onto a running system

If you need to write a spec for a system that already exists, characterization tests give you the raw material. Each test is a behavioral observation — a spec candidate waiting to be reviewed, confirmed, and written as an EARS invariant.

---

## How to Write Them

The process is deliberately mechanical. You are not making judgments about correctness yet. You are making observations.

**Step 1 — Call the code**

Write a test that calls the unit under test with a specific input.

```java
@Test
void characterize_legacy_tax_us_west_pre_2014() {
    Account account = accountWithCreationDate("2013-11-15", Region.US_WEST);
    Invoice invoice = standardInvoice(account, new BigDecimal("1000.00"));

    BigDecimal result = billingService.calculateTax(invoice);

    // Step 2 comes here
}
```

**Step 2 — Run it and record the output**

Run the test, observe what the system returns, and write that output as the assertion — whether or not you know if it's correct.

```java
    // We don't know if 87.50 is the right answer.
    // We know it is what the system currently produces.
    assertThat(result).isEqualByComparingTo("87.50");
```

**Step 3 — Name what you observed, not what you expected**

The test name should describe behavior, not correctness. This distinction matters when the test fails.

```java
// ✗ Wrong — implies you know this is correct
void legacy_tax_rate_should_be_8_75_percent_for_pre_2014_us_west()

// ✓ Right — describes what was observed
void legacy_tax_calculation_returns_87_50_for_pre_2014_us_west_standard_invoice_of_1000()
```

**Step 4 — Run the full suite against the live system**

All characterization tests must pass against the current, unmodified system before any change is made. A test that doesn't pass on the original system is not a characterization test — it is a test for a behavior that doesn't exist.

---

## A Worked Example — Billing Service

The Billing Service has a tax calculation path that behaves differently based on account age and invoice type. Nobody wrote it down. Here are the characterization tests that map it:

```java
class TaxCalculationCharacterizationTests {

    // Observation 1: pre-2014 accounts use a higher rate for standard invoices
    @Test
    void pre_2014_us_west_standard_invoice_1000_produces_tax_87_50() {
        Account account = accountWithCreationDate("2013-11-15", Region.US_WEST);
        BigDecimal result = billingService.calculateTax(standardInvoice(account, bd("1000.00")));
        assertThat(result).isEqualByComparingTo("87.50");
    }

    // Observation 2: credit invoices bypass the legacy path, even for pre-2014 accounts
    @Test
    void pre_2014_us_west_credit_invoice_1000_produces_tax_75_00() {
        Account account = accountWithCreationDate("2013-11-15", Region.US_WEST);
        BigDecimal result = billingService.calculateTax(creditInvoice(account, bd("1000.00")));
        assertThat(result).isEqualByComparingTo("75.00");
    }

    // Observation 3: post-2014 accounts always use the lower rate
    @Test
    void post_2014_us_west_standard_invoice_1000_produces_tax_75_00() {
        Account account = accountWithCreationDate("2015-03-20", Region.US_WEST);
        BigDecimal result = billingService.calculateTax(standardInvoice(account, bd("1000.00")));
        assertThat(result).isEqualByComparingTo("75.00");
    }

    // Observation 4: the boundary date itself — which path does 2014-03-01 take?
    @Test
    void boundary_date_2014_03_01_us_west_standard_invoice_1000_produces_tax_75_00() {
        Account account = accountWithCreationDate("2014-03-01", Region.US_WEST);
        BigDecimal result = billingService.calculateTax(standardInvoice(account, bd("1000.00")));
        // The boundary date uses the standard rate — it is NOT in the legacy path.
        // This is a discovered fact, not a known rule.
        assertThat(result).isEqualByComparingTo("75.00");
    }
}
```

Each test is a raw behavioral observation. Together they reveal:
- There is a legacy path and a standard path
- Credit invoices always use the standard path
- The boundary is exclusive — 2014-03-01 falls outside the legacy path
- The legacy rate is 8.75%, the standard rate is 7.5%

These four observations become the raw material for the EARS invariant `INV-03` in `spec.md`.

---

## From Observation to Spec Candidate

Each characterization test is a draft spec clause. The translation is a human judgment — not mechanical — because you are deciding whether the observed behavior is intentional.

```
Observation: pre_2014_us_west_standard_invoice_1000_produces_tax_87_50

Review question: Is this behavior intentional? Is it still correct?
  → Yes: it's a contractual commitment to early adopters
  → Translate to spec candidate:

    [CANDIDATE] INV-03
    WHEN an account was created before 2014-03-01
    AND the invoice type is not CREDIT,
    THEN the system SHALL apply the legacy tax rate for the account's region.
```

Flag every spec candidate with `[CANDIDATE]` until it has been reviewed and confirmed by someone with domain knowledge. A confirmed candidate becomes a locked EARS invariant. An unconfirmed candidate stays flagged until someone can make the call.

**Never auto-approve.** A characterization test proves behavior exists. Only a domain expert can confirm it should exist.

---

## When a Characterization Test Fails

A failing characterization test is not a test failure in the usual sense. It is a change detection signal.

```
FAIL  pre_2014_us_west_standard_invoice_1000_produces_tax_87_50
      expected: 87.50
      actual:   92.50
```

This means the tax calculation changed. There are exactly two valid responses:

**Option A — The change was unintentional**

Revert the change. The characterization test protected behavior that was either specified or contractually important. Do not patch the test to match the new output.

**Option B — The change was intentional**

This means the spec needs to be updated first — before the test is updated.

```
1. Update the spec candidate → change INV-03 to reflect the new rate
2. Get sign-off from the domain expert (this is a tax rate change — it has compliance implications)
3. Update the characterization test to match the new agreed behavior
4. Update any scenario tests that were derived from INV-03
```

**The rule:** You never update a characterization test to make it pass. You update the spec, get approval, and then update the test to reflect the approved spec change.

---

## The Lifecycle of a Characterization Test

Characterization tests are scaffolding. They are not permanent documentation.

```
Phase 1 — Write characterization tests
  → Lock all current behavior before any change

Phase 2 — Translate observations to spec candidates
  → Review with domain experts, confirm or discard each candidate

Phase 3 — Spec is locked
  → Every confirmed candidate becomes a scenario test and/or property test

Phase 4 — Delete the characterization tests
  → The scenario and property tests now own the behavioral contract
  → Characterization tests become redundant and should be removed
```

A characterization test that outlives its corresponding scenario test is a maintenance burden. It creates two tests asserting the same behavior, with no clear ownership. Delete them once the spec-derived tests exist and pass.

---

## What Characterization Tests Are Not

| Misconception | Reality |
|---|---|
| "They prove the code is correct" | They prove the code is consistent — correctness is a human judgment |
| "They replace scenario tests" | They are temporary scaffolding that produces spec candidates for scenario tests |
| "Failing one means there is a bug" | Failing one means behavior changed — whether that is a bug depends on intent |
| "They should be kept forever" | They should be deleted once scenario tests replace them |
| "They are only for legacy code" | They are for any code you need to change safely before a spec exists |

---

## Coverage Checklist

Before making any change to untested or under-specified code:

- [ ] Characterization tests exist for every code path that will be touched
- [ ] All characterization tests pass against the unmodified system
- [ ] The boundary conditions have been explicitly tested (the edges reveal the rules)
- [ ] Every test is named as an observation, not an assertion of correctness
- [ ] Every observation has been reviewed as a spec candidate
- [ ] `[CANDIDATE]` clauses are flagged and assigned to a domain expert for confirmation

**Gate:** No change to the system is made until all characterization tests pass on the original code and all `[CANDIDATE]` spec clauses are assigned for review.
