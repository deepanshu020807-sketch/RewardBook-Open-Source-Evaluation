# TESTING.md

## 1. Test Command and Environment

- Python Version: Python 3.14.5 (Windows)
- Command Used: `python -m pytest -q`
- Test Result: 222 passed, 1 warning in 0.33 seconds.
- Total Tests: 214 original tests + 8 new regression tests.

The automated tests were used to check whether the RewardBook system works correctly.

## 2. Edge Cases Tested

An edge case is an unusual input or situation that may cause a program to behave incorrectly.

| No. | Test Case | Expected Result | Actual Result |
|---|---|---|---|
| 1 | Purchase amount is ₹99.99 | 0 points should be awarded. | 0 points |
| 2 | Maximum points limit is 0 | 0 points should be awarded. | 0 points |
| 3 | Same purchase event is received multiple times | Points should be awarded only once. | Only 2 points awarded; one ledger entry |
| 4 | Refund amount is greater than purchase amount | Refund should be rejected. | Account remained unchanged |
| 5 | Amount is True or False instead of a valid number | Input should be treated as invalid. | No points or refund processed |
| 6 | Refund would make the points balance negative | Balance should never become negative. | Balance became 0 |

## 3. Regression Tests Added

I added 8 new tests to prevent previously identified bugs from returning.

1. Incorrect Points Calculation:
   Fixed the issue where rounding could award extra points.

2. Zero Points Limit:
   Fixed the issue where a points limit of zero was ignored.

3. Duplicate Purchase Events:
   Prevented the same purchase event from being processed multiple times.

4. Duplicate Refund Events:
   Prevented the same refund event from reversing points twice.

5. Excessive Refunds:
   Prevented refunds that exceed the original purchase amount, including multiple partial refunds that exceed the total.

6. Negative Points Balance:
   Prevented the account balance from becoming negative after a refund.

7. Boolean Inputs:
   Prevented True and False from being treated as valid numbers.

8. Repeat Transaction Alerts:
   Fixed crashes caused by empty or incorrectly formatted records.

## 4. Remaining Failures or Limitations

- Remaining Failed Tests: 0.
- All 222 tests passed in the reported test run.
- Known Limitation: The project stores information in memory. If the application restarts, the stored information is lost because there is no permanent database or event storage.

This testing process helped verify the main RewardBook features and check that the identified bugs were fixed.