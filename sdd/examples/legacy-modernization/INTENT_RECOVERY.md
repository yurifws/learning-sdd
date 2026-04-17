# Intent Recovery — Phase 1: The Archaeological Dig

> The first phase of modernization isn't writing new code. It's becoming a detective. Your job is to recover the lost intent of the system — to find out what it's actually supposed to do, not just what it currently does.

**System:** Billing Service | **Phase:** Pre-modernization | **Output:** `spec.md`

---

## What Intent Decay Looks Like

Ten years ago, someone wrote this:

```java
if (account.getCreatedAt().isBefore(LEGACY_CUTOFF) && invoice.getType() != InvoiceType.CREDIT) {
    amount = applyLegacyTaxRate(amount, account.getRegion());
} else {
    amount = applyStandardTaxRate(amount, account.getRegion());
}
```

Today, nobody knows:
- What `LEGACY_CUTOFF` represents (it's `2014-03-01`)
- Why credit invoices are excluded
- Whether `applyLegacyTaxRate` was a temporary fix or a permanent business rule
- Whether this logic is still correct for any account

The code is the only surviving record. The intent — the **why** — is gone.

Rewriting this without recovering the intent means guessing. And guessing means that some Tuesday, three months after launch, billing breaks for every enterprise account created before 2014.

---

## The Three Steps of the Archaeological Dig

### Step 1 — Map current behavior

Read the code. Run the system. Talk to anyone who still has context. Produce a raw inventory of what the system actually does:

| Behavior | Notes |
|---|---|
| Generates invoices on the 1st of each month | Unless account billing cycle is custom |
| Prorates on mid-cycle plan changes | Rounds down to nearest day |
| Applies legacy tax rate to pre-2014 accounts | Region-dependent; excludes credit invoices |
| Sends dunning emails at day 3, 7, 14 | Day 14 triggers account suspension |
| Reconciles payments nightly at 02:00 UTC | Fails silently if payment provider is unreachable |
| Allows manual invoice override by `BILLING_ADMIN` role | No audit log recorded |

This is not a spec yet. It is raw material. The goal of this step is coverage — find everything the system does, including the ugly parts.

---

### Step 2 — Lock behavior with characterization tests

A characterization test does not check that behavior is *correct*. It checks that behavior is *consistent*. It is a digital straitjacket: whatever the system does today, the test locks it in so that any change to that behavior triggers a failure.

This serves two purposes:
1. It gives you a safety net before you touch anything
2. It forces you to make implicit behavior explicit — you cannot write the test without understanding the input/output contract

```java
// CharacterizationTest.java — Billing Service

@Test
void legacy_tax_rate_applied_to_pre_2014_account_standard_invoice() {
    Account account = accountWithCreationDate("2013-11-15", Region.US_WEST);
    Invoice invoice = standardInvoice(account, BigDecimal.valueOf(1000.00));

    BigDecimal result = billingService.calculateTax(invoice);

    // We don't know if this rate is correct.
    // We know it's what the system currently produces.
    // Changing this output requires a deliberate spec update.
    assertThat(result).isEqualByComparingTo("87.50");
}

@Test
void standard_tax_rate_applied_to_credit_invoice_regardless_of_account_age() {
    Account account = accountWithCreationDate("2013-11-15", Region.US_WEST);
    Invoice invoice = creditInvoice(account, BigDecimal.valueOf(1000.00));

    BigDecimal result = billingService.calculateTax(invoice);

    assertThat(result).isEqualByComparingTo("75.00");
}
```

> Write characterization tests for every behavior you map. Run them against the old system. They must all pass before you touch a single line of code or write a spec clause. These tests are your ground truth.

---

### Step 3 — Translate behavior into a structured spec

With behavior mapped and locked, translate it into `spec.md`. This is not a free-form document. Structure matters — for humans and for AI agents reading it later.

A well-structured spec has three sections: invariants, scenarios, and the do-not list.

---

## The Spec Structure

### Invariants (EARS format)

Invariants are the non-negotiable rules. Write them in EARS syntax: structured `WHEN / THEN` clauses the AI can read, verify, and reason about unambiguously.

```
INV-01  The system SHALL generate an invoice for each active account on the 1st calendar day
        of the account's billing period.

INV-02  WHEN an account's plan changes mid-cycle,
        THEN the system SHALL prorate the charge for the remaining days of the billing period,
        rounding down to the nearest whole day.

INV-03  WHEN an account was created before 2014-03-01
        AND the invoice type is not CREDIT,
        THEN the system SHALL apply the legacy tax rate for the account's region.

INV-04  WHEN an invoice remains unpaid for 14 days past the due date,
        THEN the system SHALL suspend the account and send a suspension notice.

INV-05  WHEN the payment provider is unreachable during nightly reconciliation,
        THEN the system SHALL log the failure and retry at the next scheduled run.
        The system SHALL NOT mark payments as failed based on a single unreachable response.
```

---

### Scenarios

Scenarios translate invariants into concrete input/output pairs. They bridge the spec and the test suite.

```
SCN-01  Happy path: standard monthly invoice
  GIVEN an active account on a monthly plan at $99/month
  AND today is the 1st of the billing cycle
  WHEN the invoice job runs
  THEN the system generates invoice with amount $99.00
  AND sends the invoice to the account's billing email

SCN-04  Proration: mid-cycle upgrade
  GIVEN an account on a $50/month plan
  AND the account upgrades to $100/month on day 15 of a 30-day month
  WHEN the invoice is generated at end of cycle
  THEN the invoice contains: base $50.00 + prorated upgrade $25.00 = $75.00

SCN-07  Legacy tax: pre-2014 US-WEST account, standard invoice
  GIVEN an account created on 2013-11-15 in region US-WEST
  AND a standard invoice for $1,000.00
  WHEN tax is calculated
  THEN the system applies legacy rate 8.75% → tax = $87.50

SCN-08  Legacy tax exception: credit invoice always uses standard rate
  GIVEN the same pre-2014 US-WEST account
  AND a credit invoice for $1,000.00
  WHEN tax is calculated
  THEN the system applies standard rate 7.5% → tax = $75.00
  AND the legacy tax path is NOT taken
```

---

### The Do-Not List

The do-not list is the most important section most teams skip. It defines the boundaries of this modernization. It prevents scope creep and protects the team from the most common failure mode: building the system you wish you had instead of replacing the system you have.

```
## DO NOT — Billing Service Modernization

DO NOT change the tax rate logic, even if the rates appear incorrect.
  → Tax rates are a business and compliance decision. They are out of scope for this migration.
  → A separate compliance review is scheduled for Q3. This migration must not block or preempt it.

DO NOT remove the legacy tax path for pre-2014 accounts.
  → The condition is not a bug. It is a contractual commitment to early adopters.
  → Removing it would silently overcharge approximately 4,200 accounts.

DO NOT add audit logging for manual invoice overrides in this phase.
  → The absence of audit logging is a known gap. It is tracked separately.
  → Adding it now changes the data model, which is out of scope.

DO NOT improve the dunning email copy.
  → Email content is owned by the marketing team. Any changes must go through their review process.

DO NOT add new payment providers.
  → The new system must replicate the behavior of the old system for existing providers only.
  → New providers are a Phase 3 feature.
```

> The do-not list is a contract with the team and the stakeholders. Any item added to it requires explicit sign-off. Any item removed from it triggers a spec update and a new round of review.

---

## Where AI Agents Fit in Phase 1

AI agents accelerate the archaeological dig — but they must be constrained by the spec, not left to infer intent from code alone.

| Task | Agent role | Human role |
|---|---|---|
| Read legacy code and draft initial invariant list | First-pass extraction — finds patterns, surfaces EARS candidates | Review every clause — agents miss implicit business context |
| Generate characterization test stubs | Produces test scaffolding from mapped behavior | Verify tests against the running system — agents cannot run the legacy system |
| Flag ambiguous behavior | Surfaces `[NEEDS_CLARIFICATION]` items in the spec draft | Resolve each one with a domain expert or stakeholder |
| Translate finalized spec to EARS clauses | Formats and structures confirmed behavior | Approve each invariant — once committed, the spec is locked |

**Critical rule:** No AI agent may decide what behavior is correct. Agents surface candidates. Humans confirm intent. The spec is locked by human sign-off, not by AI confidence.

See [`../../CLARIFICATION_GATE.md`](../../CLARIFICATION_GATE.md) for the full process of surfacing and resolving ambiguity before the spec is locked.

---

## Output: A Locked spec.md

Phase 1 is done when:

- [ ] Every behavior in the system is mapped
- [ ] Characterization tests exist for every mapped behavior and all pass
- [ ] Every invariant is written in EARS format and reviewed
- [ ] Every scenario has a concrete input/output pair
- [ ] The do-not list is agreed on and signed off
- [ ] All `[NEEDS_CLARIFICATION]` items are resolved
- [ ] `spec.md` is committed to the repository

**The spec is now the system of record. The old code is evidence. The spec is truth.**

Phase 2 begins only when this checklist is complete.
