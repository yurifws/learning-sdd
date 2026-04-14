# PROPERTY_BASED_TESTING.md

> Example-based tests ask: does it work for this specific input?
> Property-based tests ask: is this behavior always true for any valid input?
> The difference is the difference between sampling and proof.

---

## The Mental Shift

Example-based testing is spot-checking. You pick five inputs you thought of, confirm they work, and ship. 100% line coverage can still pass while a critical bug waits in an input combination you never imagined.

Property-based testing is making a promise. Instead of picking inputs, you state a universal rule — a truth that must hold for *any* valid input — and then unleash a framework that tries its absolute best to find one case where the rule breaks.

```
Example-based:  "Does this work for 500.00?"      → spot check
Property-based: "Is total balance always conserved?" → universal proof
```

This forces a different and more powerful question. Not *does the code work for this input*, but *what are the non-negotiable truths of this system?* Get those truths right, and you've proven correctness — not just observed it for a few cases.

---

## The Problem With Examples

You write a test for a money transfer:

```python
def test_transfer():
    account = Account(balance=1000.00)
    transfer(500.00, from_account=account, to_account=other)
    assert account.balance == 500.00
```

This test passes. But it only proves the transfer works for exactly `500.00` from a `1000.00` balance. What about:
- `0.01` (minimum valid amount)?
- `999.99` (just under balance)?
- `1000.00` (exactly the balance)?
- `100.005` (sub-cent precision)?
- `-50.00` (negative amount — should be rejected)?

You would have to write a separate test for each case you thought of. And the cases you didn't think of are exactly where the bugs live.

---

## Properties and Invariants

A **property** is a rule that must hold true for all valid inputs — not just the ones you picked.

**Example properties for a money transfer:**

```
Property 1: The total money in the system is always conserved.
  → from_account.balance + to_account.balance is the same before and after any valid transfer.

Property 2: A transfer never produces a negative balance.
  → After any valid transfer, from_account.balance >= 0 always.

Property 3: An invalid transfer never changes any balance.
  → If a transfer fails (insufficient funds, invalid amount), both balances are unchanged.
```

These properties are true for every valid input — not just `500.00`. A property-based test framework generates hundreds of random inputs and checks that the property holds for all of them.

---

## EARS → Invariant → Property

EARS requirements translate directly into system invariants, and invariants map directly to executable properties. The path is mechanical:

| EARS clause | System invariant | Executable property |
|---|---|---|
| `WHEN a request arrives without a valid JWT, the system SHALL return HTTP 401` | Every unauthenticated request → 401, always | `@given(request=unauthenticated_requests()) assert response.status == 401` |
| `The system SHALL never expose the destination URL in any error response` | No 4xx body ever contains the destination URL | `@given(token, bad_request) assert destination_url not in response.body` |
| `WHEN a transfer is made with any valid amount, the system SHALL conserve total balance` | total balance is unchanged by any valid transfer | `@given(amount=valid_amounts()) assert before == after` |

The through line: a clear EARS clause → a system invariant (a rule that never varies) → a concrete, runnable property. If an EARS clause uses the words `always`, `never`, `any`, or `exactly`, it is almost certainly expressing an invariant and should become a property-based test.

---

## The Counter-Example Engine

When you run a property test, the framework doesn't pick a few "representative" inputs. It generates thousands of variations with one goal: find one input that breaks the rule.

The engine deliberately targets inputs you would never write by hand:

- Empty strings and strings of length 1
- Strings full of emojis, control characters, and null bytes
- Strings at maximum allowed length
- Numbers at their exact boundaries (0, -1, `MAX_INT`, `MIN_INT`)
- `null` / `None` / `undefined` values
- Lists with zero items, one item, or thousands of items
- Timestamps at epoch, at midnight, across DST boundaries

When it finds a failure it doesn't just report "test failed." It reports the exact input that proved your rule wrong:

```
Falsifying example: test_wrong_password_always_returns_401(
    wrong_password=''
)
AssertionError: expected 401, got 200
```

An empty string accepted as a valid password — an input no example test would have tried.

After finding a failure, modern frameworks do something called **shrinking**: they automatically work backwards to find the absolute simplest input that still causes the same failure. A huge, complex JSON payload gets reduced to the one field, in the one value, that is the actual bug. You go from a haystack to the needle.

---

## Hypothesis (Python)

Hypothesis is the standard property-based testing library for Python.

### Installation
```bash
pip install hypothesis
```

### Basic structure

```python
from hypothesis import given, strategies as st
from hypothesis import settings

@given(
    amount=st.decimals(min_value=Decimal("0.01"), max_value=Decimal("10000.00"),
                       allow_nan=False, allow_infinity=False)
)
def test_transfer_conserves_total_balance(amount):
    """Property: total money in system is always conserved."""
    initial_total = from_account.balance + to_account.balance
    assume(amount <= from_account.balance)  # only test valid transfers

    transfer(amount, from_account, to_account)

    assert from_account.balance + to_account.balance == initial_total
```

### Strategies — the input generators

| Strategy | Generates |
|---|---|
| `st.integers(min_value=0, max_value=100)` | Random integers in range |
| `st.decimals(min_value=0.01, max_value=10000)` | Random decimals |
| `st.text(min_size=1, max_size=50)` | Random strings |
| `st.emails()` | Valid email addresses |
| `st.datetimes()` | Random datetime values |
| `st.one_of(st.integers(), st.text())` | Either integers or strings |
| `st.lists(st.integers(), min_size=1)` | Non-empty lists of integers |
| `st.builds(User, email=st.emails())` | Build a User object with generated email |

### Shrinking

When Hypothesis finds a failing input, it automatically **shrinks** it to the simplest possible case that still fails.

```
Hypothesis found: amount=Decimal('7234.18') fails
Hypothesis shrunk to: amount=Decimal('0.01') fails
```

This makes debugging easy — you always get the minimal failing example.

### Real example: user authentication

```python
from hypothesis import given, strategies as st, assume
import string

VALID_PASSWORD_CHARS = string.ascii_letters + string.digits + "!@#$%"

@given(
    password=st.text(
        alphabet=VALID_PASSWORD_CHARS,
        min_size=8,
        max_size=128
    )
)
def test_any_valid_password_can_be_stored_and_verified(password):
    """Property: any password meeting length/character rules can be stored and verified."""
    user = create_user(email="test@example.com", password=password)
    assert verify_password(password, user.hashed_password) is True

@given(
    email=st.emails(),
    wrong_password=st.text(min_size=1)
)
def test_wrong_password_always_returns_401(email, wrong_password):
    """Property: any incorrect password always returns 401, never 200."""
    user = create_user(email=email, password="correct-password-abc123")
    assume(wrong_password != "correct-password-abc123")

    response = client.post("/auth/login", json={"email": email, "password": wrong_password})

    assert response.status_code == 401
    assert "token" not in response.json()
```

---

## FastCheck (TypeScript / JavaScript)

FastCheck is the Hypothesis equivalent for TypeScript and JavaScript.

### Installation
```bash
npm install --save-dev fast-check
```

### Basic structure

```typescript
import * as fc from 'fast-check';

test('transfer conserves total balance', () => {
  fc.assert(
    fc.property(
      fc.float({ min: 0.01, max: 10000, noNaN: true }),
      (amount) => {
        fc.pre(amount <= fromAccount.balance); // only test valid transfers

        const initialTotal = fromAccount.balance + toAccount.balance;
        transfer(amount, fromAccount, toAccount);

        return Math.abs((fromAccount.balance + toAccount.balance) - initialTotal) < 0.001;
      }
    )
  );
});
```

### Arbitraries — the input generators

| Arbitrary | Generates |
|---|---|
| `fc.integer({ min: 0, max: 100 })` | Random integers in range |
| `fc.float({ min: 0.01, noNaN: true })` | Random floats |
| `fc.string({ minLength: 1, maxLength: 50 })` | Random strings |
| `fc.emailAddress()` | Valid email addresses |
| `fc.date()` | Random Date objects |
| `fc.oneof(fc.integer(), fc.string())` | Either integers or strings |
| `fc.array(fc.integer(), { minLength: 1 })` | Non-empty arrays of integers |
| `fc.record({ email: fc.emailAddress(), name: fc.string() })` | Objects with generated fields |

### Real example: user authentication

```typescript
import * as fc from 'fast-check';

test('any valid password can be stored and verified', () => {
  fc.assert(
    fc.property(
      fc.string({ minLength: 8, maxLength: 128 }),
      async (password) => {
        fc.pre(/^[a-zA-Z0-9!@#$%]{8,128}$/.test(password));

        const user = await createUser({ email: 'test@example.com', password });
        const result = await verifyPassword(password, user.hashedPassword);

        return result === true;
      }
    )
  );
});

test('wrong password always returns 401, never exposes token', () => {
  fc.assert(
    fc.property(
      fc.emailAddress(),
      fc.string({ minLength: 1 }),
      async (email, wrongPassword) => {
        fc.pre(wrongPassword !== 'correct-password-abc123');

        await createUser({ email, password: 'correct-password-abc123' });
        const response = await client.post('/auth/login', { email, password: wrongPassword });

        return response.status === 401 && !('token' in response.body);
      }
    )
  );
});
```

---

## When to Use Property-Based Testing

Not every test should be property-based. Use it where it adds the most value:

| Good fit | Why |
|---|---|
| Financial calculations | Numeric precision and edge cases are hard to enumerate |
| Serialization / deserialization | `parse(serialize(x)) === x` for all x |
| Authentication and authorization | Security must hold for all valid and invalid inputs |
| Data validation logic | Boundary conditions are the most common source of bugs |
| Sorting and ordering algorithms | Properties like stability and idempotency |
| API request parsing | Malformed inputs must be handled safely for all shapes |

| Poor fit | Why |
|---|---|
| UI rendering | Visual correctness is hard to express as a property |
| Third-party integrations | Generating inputs for external APIs is impractical |
| One-off business logic | If there's only one valid input, use an example test |

---

## Deriving Properties From Scenarios

Every `Then` clause in a scenario is a candidate for a property.

**Scenario:**
```
Given  a user with sufficient balance
When   they transfer any valid amount
Then   the total money in the system is unchanged
  And  the sender's balance decreases by exactly the transfer amount
  And  the receiver's balance increases by exactly the transfer amount
```

**Properties derived:**
```python
# From "total money is unchanged"
@given(amount=valid_transfer_amounts())
def test_total_balance_conserved(amount): ...

# From "sender balance decreases by exactly transfer amount"
@given(amount=valid_transfer_amounts())
def test_sender_balance_debited_exactly(amount): ...

# From "receiver balance increases by exactly transfer amount"
@given(amount=valid_transfer_amounts())
def test_receiver_balance_credited_exactly(amount): ...
```

One scenario can produce multiple properties. Each `Then` clause that contains "always", "any", "never", or "exactly" is a strong signal for a property-based test.

---

## The Six-Step Recipe

A repeatable process from requirement to proven code:

| Step | What to do |
|---|---|
| **1. Write EARS criteria** | State each requirement as a clear EARS clause |
| **2. Group into invariants** | Cluster related clauses by theme (security, validation, data integrity) — each cluster is a candidate invariant |
| **3. Define generators** | Tell the framework how to produce valid inputs: `st.decimals(min_value=0.01)`, `@Provide Arbitrary<OffsetDateTime> pastTimestamps()` |
| **4. Write the property** | Implement the property using Hypothesis or FastCheck — state the rule, not the examples |
| **5. Enable shrinking and replay** | Confirm that failures produce a minimal counterexample and a seed you can replay deterministically |
| **6. Gate in CI** | Run property tests in CI. Make it impossible to merge code that violates an invariant |

---

## CI as Mandatory Gate

Property tests are only as strong as the enforcement behind them. A property test run locally but not in CI is just documentation that can drift.

The rule: **every invariant is a merge gate.**

```yaml
# .github/workflows/ci.yml
- name: Property tests
  run: pytest tests/ -k "property" --hypothesis-seed=0
  # Failing this step blocks the PR — no exceptions
```

This is the difference between having stated your system's truths and having proven them. The CI gate is what turns a property into a guarantee.

**When a property test fails in CI:** The framework's shrinking gives you the minimal counterexample. Use live-system observation to understand what the system actually does with that input. See [`mcp/README.md` — MCP + Property Tests](../mcp/README.md) for the full diagnostic loop.
