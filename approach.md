# APPROACH.md

## 1. Initial Project Check

Before changing the code, I first checked how the project was working.

### Commands I ran:

- `python -m pip install -r requirements-dev.txt` – Installed the required packages.
- `python -m pytest` – Ran the existing tests.
- `python -m main` – Checked whether the application runs.

### Initial test results:

- 67 tests failed.
- 147 tests passed.
- 1 warning appeared.
- Total: 214 tests.

### Application output:

- Points example: 29
- Repeat warnings: []

These results helped me understand that some parts of the project were working, but several problems needed to be fixed.

## 2. Understanding the Project

Before making changes, I studied the important files.

### Files I checked:

- `service.py`: Contains the main functions for calculating points, processing purchases and refunds, and detecting repeat transactions.
- `models.py`: Creates a new customer account with a points balance, transaction records, processed event IDs and a ledger.
- `fixtures.py` and `data/records.json`: Contain sample data used for testing.
- `tests/test_acceptance_matrix.py` and `tests/test_contract.py`: Contain tests that explain how the reward system should behave.

### How the main features work:

**Points Calculation:**

The system calculates reward points based on the purchase amount. It also applies bonus points and any maximum points limit.

**Purchase and Refund Processing:**

The system checks each event before processing it.

- For a successful purchase, it awards points and records the transaction.
- For a refund, it calculates how many points should be reversed.
- It prevents duplicate events from being processed.
- It makes sure the points balance never becomes negative.

**Transaction Alerts:**

The system checks purchases made by the same account. If multiple transactions have the same date and amount and exceed the given limit, it creates an alert for review.

I understood these features by reading the problem statement and comparing the requirements with the failed test results.

## 3. Problems Found and How I Fixed Them

### Problem 1: Incorrect Points Calculation

**Problem:**

The system used rounding to calculate points. Because of this, a purchase of 9999 paise could incorrectly receive 1 point instead of 0.

Another problem was that a maximum points limit of zero was ignored.

**Why it happened:**

The original code used `round()` instead of integer floor division. It also checked the points limit in a way that treated zero as if no limit was provided.

**How I fixed it:**

- Replaced rounding with integer floor division (`//`).
- Added a proper check for the points limit.
- Added validation for negative or invalid amounts and rules.

### Problem 2: Duplicate Purchase Events

**Problem:**

When the same purchase event was received multiple times, the system added its ID to the list again and could create duplicate ledger entries.

**Why it happened:**

The system did not stop an already processed event early enough.

**How I fixed it:**

- Added a check at the beginning of event processing.
- If the event ID already exists, the system stops processing it.
- Only accepted purchase and refund events are added to the processed event list.

### Problem 3: Incorrect Refund Handling

**Problem:**

The system could accept negative refunds, refunds greater than the original purchase, and invalid inputs such as `True`, `False` or text values.

Multiple partial refunds could also exceed the original purchase amount.

**Why it happened:**

The original code did not properly validate refund amounts or check the total refunded amount.

**How I fixed it:**

- Added checks for valid positive integer refund amounts.
- Rejected Boolean values and other invalid inputs.
- Checked that the total refund does not exceed the original purchase amount.
- Prevented the points balance from becoming negative.

## 4. Important Implementation Decisions

### Why I made these changes:

- Separated points calculation and input validation to make the code easier to understand.
- Added checks to prevent Boolean values from being accepted as numbers.
- Used `deepcopy()` to work with a separate copy of the account instead of changing the original account.
- Added checks for missing event IDs, transaction IDs and invalid input data.

### Other approaches I considered:

- **Changing the original account directly:** I avoided this because the project requires the original account data to remain unchanged.
- **Raising errors for invalid refunds:** I avoided this because invalid refund events should be safely ignored without damaging account information.

### Assumptions:

- Unknown or invalid events should not change account balances or transaction records.
- Repeat transaction alerts should be created only when the number of matching transactions is greater than the given limit.

## 5. Testing and Verification

I added a new file called `tests/test_regression.py` with 8 tests to check the problems I fixed.

### Commands and results:

**Command:** `python -m pytest`

Result:
- 222 tests passed.
- 1 warning.
- 0 failed tests.
- Execution time: 0.33 seconds.

**Command:** `python -m main`

Application output:
- Points example: 27
- Repeat warnings: []

### Important cases I tested:

1. **Purchase amount boundaries:** Checked amounts such as 9999, 10000, 19999 and 20000 paise to make sure points are calculated correctly.
2. **Zero points limit:** Checked that a zero limit awards zero points.
3. **Duplicate purchases:** Checked that replaying the same purchase does not create duplicate records or award points again.
4. **Partial refunds:** Checked that multiple partial refunds are accepted only when their total does not exceed the original purchase amount.
5. **Negative balance:** Checked that refund processing never makes the points balance negative.

## 6. Remaining Problems or Limitations

All 222 tests passed in my reported test run.

No remaining test failures were reported.


