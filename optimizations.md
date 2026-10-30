# OPTIMIZATIONS.md

## 1. Starting Point and Goal

### What needed improvement?

- **Accurate points calculation:** The original code used decimal calculations and rounding. This could give incorrect reward points. I changed it to use whole-number calculations.
- **Duplicate events:** When the same transaction was received again, its event ID was added to the list repeatedly. This created unnecessary entries and repeated processing.
- **Repeat transaction detection:** The system needed a better way to organize and group transactions when checking for possible repeated purchases.

### What showed that improvements were needed?

The first test run showed 67 failing tests. The failures were related to incorrect points calculation, duplicate event IDs and refund handling.

## 2. Different Options Considered

### Option A: Using a Set to Check Duplicate Events

- A set can check whether an event ID already exists faster than a list.
- However, the existing project and tests require `processed_event_ids` to remain an ordered list.
- Changing it to a set could break the existing tests.
- Therefore, I kept the list.

### Option B: Stop Processing Duplicate Events Early

- Instead of processing the same event again, the system checks whether its ID has already been processed.
- If it has, the function returns immediately.
- This avoids unnecessary calculations and prevents duplicate records.
- I implemented this approach.

### Option C: Use Whole-Number Calculations

- The original code used decimal numbers and rounding.
- I replaced this with integer floor division (`//`).
- This gives accurate whole-number results and follows the required reward calculation rules.
- I also used integer calculations for refund points.

## 3. Changes Made

The following improvements were implemented:

1. Replaced rounding with integer floor division to calculate reward points accurately.
2. Added an early check to stop duplicate events from being processed again.
3. Improved repeat transaction detection by grouping transactions using a dictionary in a single pass.

### Tests used to check these changes

- `tests/test_acceptance_matrix.py`
  - `test_integer_floor_points_calculation`
  - `test_bonus_and_cap_matrix`
  - `test_purchase_event_replay_is_idempotent`

- `tests/test_regression.py`
  - `test_regression_floor_boundary_values`
  - `test_regression_event_replay_idempotency_no_list_duplication`

### Test execution time

- Before changes: Around 2.11 seconds for 214 tests. Some tests failed.
- After changes: Around 0.33 seconds for 222 tests. All tests passed.

These are the recorded test execution times, not a separate performance benchmark.

## 4. Trade-offs

### Copying Data Instead of Changing the Original

- The code uses `deepcopy(account)` to create a separate copy of account data.
- This prevents the original account information from being changed accidentally.
- However, creating a copy requires extra memory and processing.
- I kept this approach because the project requires the original account data to remain unchanged.

### Keeping a List Instead of a Set

- A set can find duplicate IDs faster.
- However, a list keeps the order in which events were processed.
- The existing project tests also expect a list.
- Therefore, I kept the list to maintain compatibility with the project.

## 5. Possible Future Improvements

### Faster Duplicate Checking

If the system handles thousands of transactions for each account, we could maintain an additional set internally to check duplicate event IDs faster while keeping the original list.

### Processing Large Transaction Data

The repeat transaction detection system currently works with transaction data loaded into memory.

In the future, it could process transactions one by one using a generator. This could reduce memory usage when handling very large amounts of transaction data.
