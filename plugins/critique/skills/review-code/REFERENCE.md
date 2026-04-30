# Example Output

This reference shows the expected format and level of detail for a code review report.

---

## Example Report

```
---
Code Review for src/order

4 files reviewed, 4 issues found (1 P1, 2 P2, 1 P3).

### 1. [P2] OrderService has three unrelated reasons to change
**Location:** src/order/service.go:87

PlaceOrder validates input, calculates pricing with discount rules, and writes to the database in one 90-line function. A pricing policy change forces edits to the same function as a validation rule change. Extract validateOrder, priceOrder, and saveOrder so each has a single reason to change.

### 2. [P2] HTTP handler embeds discount business logic
**Location:** src/order/handler.go:23

The handler parses the HTTP request, then inline-calculates volume discounts before calling the service. Discount rules belong in the domain layer so they can be tested without an HTTP context and reused from other entry points (CLI, queue consumer).

### 3. [P1] Repository directly instantiates database connection
**Location:** src/order/repo.go:14

NewOrderRepo calls sql.Open internally, making it impossible to test with a fake database or swap drivers. Accept a *sql.DB (or an interface) as a parameter so callers control the connection lifecycle.

### 4. [P3] Handler name doesn't reveal intent
**Location:** src/order/handler.go:61

`processOrder` is indistinguishable from `handleOrder` or `doOrder`. Since this handler specifically processes checkout submissions, rename to `handleCheckout` to convey the specific action.
---
```

---

## Format Rules

### Findings use H3 headers with priority tag, Location, and explanation

```
### 1. [P2] OrderService has three unrelated reasons to change
**Location:** src/order/service.go:87

Explanation of the design problem and rationale for the fix.
```

### Only show issues found — no passing rows

Do not include items that passed review. Start with "N files reviewed, M issues found (severity breakdown)."

### No tables

Do not include summary tables or issue tables. Findings are the only output.

### Truncation footer when the cap kicks in

When findings exceed the reporting cap (see SKILL.md → Reporting Cap), end the report with a single-line footer:

```
Note: 7 additional findings omitted (4 P2, 3 P3) — re-run after addressing these to surface what remains.
```

The footer is omitted when all findings fit under the cap. The header count reflects the *reported* findings; the footer count reflects the *omitted* tail.
