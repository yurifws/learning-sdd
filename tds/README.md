# TDS — Test-Driven Specs

> A methodology for turning specs from static documents into executable guardrails.
> Done is no longer when the code is written. Done is when all tests derived from the original intent are passing.

---

## The Problem TDS Solves

Teams write detailed specs. Engineers read them. Features ship. And the feature is almost — but not quite — right.

This drift isn't a communication failure. It's a structural one. **Requirements and tests live in separate worlds.** The spec is a document. The tests are in the codebase. The gap between them is where bugs are born.

TDS closes that gap by making requirements produce tests — directly, mechanically, with no room for interpretation.

---

## The Core Shift

```
Traditional workflow:
  Spec (document) → Code → Tests (hoping they match the spec)

TDS workflow:
  Spec → Scenarios (G/W/T) → Tests (failing) → Code → Tests (passing)
```

The spec doesn't describe intent anymore. It *is* the test plan.

---

## Reading Order

| Step | File | What it answers |
|---|---|---|
| 1 | [`SCENARIO_FORMAT.md`](SCENARIO_FORMAT.md) | How to write scenarios the AI and tests can follow |
| 2 | [`TEST_FIRST_GATE.md`](TEST_FIRST_GATE.md) | How to enforce test-first as governance, not preference |
| 3 | [`PROPERTY_BASED_TESTING.md`](PROPERTY_BASED_TESTING.md) | How to prove behavior is always true, not just for examples you thought of |
| 4 | [`DEFINITION_OF_DONE.md`](DEFINITION_OF_DONE.md) | How to create an objective, auditable definition of done |

Then see [`example/`](example/) for a full walkthrough applied to a real feature.

**Enforcement in practice:** See [`kiro/hooks/HOOKS.md`](../kiro/hooks/HOOKS.md) (ALWAYS-004) for how property tests are wired into automated hooks that block task completion if an invariant is violated.

---

## The Three Scenario Types

Every spec must cover all three. Missing any one is a gap.

| Type | Question it answers | Where bugs hide |
|---|---|---|
| **Happy path** | Does it work when everything is correct? | Baseline — must pass first |
| **Edge cases** | Does it handle weird but valid inputs? | Time zones, duplicates, empty lists, max lengths |
| **Error cases** | Does it fail gracefully and securely? | Wrong input, missing auth, downstream failure |

---

## Given / When / Then

The scenario format that bridges spec and test:

```
Given  [initial state of the system]
When   [action or trigger]
Then   [expected outcome]
```

This structure forces absolute clarity. Each clause is unambiguous, directly testable, and a perfect instruction for both humans and AI agents.

---

## Test-First as Governance

Test-first isn't a developer preference — it's a policy.

The **test-first gate** is a checklist that must pass before any implementation begins:

- [ ] Every requirement maps to at least one scenario
- [ ] Edge cases and error cases are explicitly covered
- [ ] Verification commands are specified
- [ ] Tests are confirmed to **fail** before any code is written

That last point is critical. A passing test before code exists means the test is not testing anything.

---

## Property-Based Testing

Example-based tests answer: *does it work for this specific input?*

Property-based tests answer: *is this behavior always true for any valid input?*

```python
# Example-based: checks ONE case
assert transfer(100, from_account, to_account) == success

# Property-based: checks ALL valid amounts
@given(amount=st.decimals(min_value=0.01, max_value=account.balance))
def test_transfer_always_debits_correct_amount(amount):
    transfer(amount, from_account, to_account)
    assert from_account.balance == original_balance - amount
```

Frameworks: **Hypothesis** (Python) · **FastCheck** (TypeScript/JavaScript)

---

## Definition of Done

A requirement is done when — and only when — every scenario derived from it has a passing, automated test. No exceptions.

```
Requirement → Scenario (G/W/T) → Test → ✅ Passing
```

This is tracked in an audit trail: a table that maps every requirement ID to its scenario, verification command, and PR. See [`DEFINITION_OF_DONE.md`](DEFINITION_OF_DONE.md).
