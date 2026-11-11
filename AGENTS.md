# Checkout review POC

This repository contains a small Python checkout and SQLite order-lookup example.
Review it against the business rules below. Keep the review focused on concrete
correctness and security issues.

The finance integration contract below is an explicit additional requirement
for this POC demo. It is not inferable from the existing checkout docstring.

## Review behavior

- Review and report findings only.
- Do not modify files, commit changes, push branches, or merge pull requests.
- These instructions describe intended behavior; actual access restrictions
  and model selection are configured separately.

## Code Review Rules

### Coupon eligibility

- The SAVE10 coupon gives a 10% discount when the subtotal is at least $100.
- Exactly $100 is eligible. Below $100 is not eligible.
- Apply no discount when the coupon is absent or unrecognized.
- Flag eligibility conditions that violate these boundaries.

### Tax calculation

- Calculate the discount first.
- Tax is 10% of the amount remaining after the discount.
- Calculate the dollar total as subtotal minus discount plus tax, rounded to
  two decimal places, before converting it to the output format below.
- Flag calculations that tax the original subtotal when a discount applies.

### FIN-001: Finance integration output contract

- The finance consumer interprets checkout()'s return value as integer US cents.
  It does not perform a dollars-to-cents conversion itself.
- checkout() inputs remain US dollars. Its return value must be a Python int
  representing the final amount in cents, after applying the discount and tax.
- Returning a dollar-denominated number or a float violates this contract,
  even when its dollar calculation is otherwise correct. The consumer would
  interpret 198 as $1.98 instead of $198.00.
- Examples of valid results:
  - checkout(100, "SAVE10") returns 9900 (int).
  - checkout(200, "SAVE10") returns 19800 (int).
  - checkout(100) returns 11000 (int).
- Preserve cent precision when converting; do not truncate a value because of
  floating-point representation. Integer-cent calculations or decimal arithmetic
  are acceptable; do not require a particular implementation.
- This contract applies only to checkout()'s returned value. orders.total is
  a legacy dollar-denominated database column; do not request a schema or fixture
  migration solely because of FIN-001.
- Treat a wrong return unit/type as one integration-contract finding, separate
  from coupon-eligibility and tax-calculation defects. Cite FIN-001 when applicable.
- Do not flag a return that already satisfies this contract.

### Safe order lookup

- Treat order_id as untrusted user input.
- Bind order_id using SQLite query parameters.
- Flag user input interpolated or concatenated into SQL statements.
- An order ID must be treated as a literal value and must not change the SQL
  query's meaning. Preserve valid order lookup behavior.

### Finding quality

- Identify each independent root cause separately.
- For each finding, cite the relevant code location, explain the impact,
  and include a small input example or reproduction when useful.
- Describe a focused correction without changing the source files.
- Do not invent requirements for shipping, refunds, coupon stacking, or
  invalid-input handling that this POC does not specify.
- Prioritize the business-rule and SQL-safety issues above over formatting
  and style suggestions. Continue to report other concrete, consequential bugs.
