# CHALLENGES.md

## 1. Challenges I Faced

The biggest challenge was preventing the same transaction from being processed multiple times while keeping the transaction records correct.

I faced three main problems:

1. **Duplicate transactions:** When the same event was received again, the system added its ID to the list again and could apply reward points more than once.

2. **Boolean values treated as numbers:** In Python, `True` and `False` can behave like numbers. Because of this, the system could mistakenly accept `True` as 1 paisa instead of treating it as invalid input.

3. **Excessive refunds:** When a customer made multiple partial refunds, their total refund could become greater than the original purchase amount. The system needed to reject such refunds without changing the existing account details.

## 2. Evidence of the Problems

I found these problems while running the automated tests.

- In `test_purchase_event_replay_is_idempotent`, the same purchase ID appeared more than once in the processed events list.
- In `test_invalid_or_excess_refund_does_not_change_account`, the input `True` was incorrectly treated as 1 paisa.
- In `test_refund_amount_bounds_and_balance`, a refund of 125000 paise was accepted even though the original purchase was only 100000 paise.

These test failures helped me understand where the problems were occurring.

## 3. What I Tried

- For duplicate transactions, I first tried checking whether the event ID already existed before adding it to the list. However, checking too late allowed other calculations and changes to happen before the duplicate was detected.
- For invalid inputs, I tried using `isinstance(amount, int)`. However, this also accepts Boolean values in Python.
- For refunds, I initially checked whether a single refund was greater than the purchase amount. This did not handle situations where multiple partial refunds together exceeded the original purchase amount.

## 4. How I Solved the Problems

- **Duplicate transactions:** I added an early check at the beginning of event processing. If the event ID has already been processed, the function stops immediately.
- **Invalid number inputs:** I added an extra check to reject Boolean values such as `True` and `False`, even though Python treats them as integers.
- **Excessive refunds:** I added a check to make sure the total of all refunds does not exceed the original purchase amount. If it does, the refund is rejected without changing the account.
- **Refund points calculation:** I used integer calculations to reverse reward points proportionally. I also ensured that the points balance never becomes negative.

## 5. Testing and Proof

After making the changes, I checked the affected test cases.

- All 20 test cases for duplicate purchase events passed.
- All 13 test cases for invalid and excessive refunds passed.
- I added regression tests to make sure these problems do not return.

The new regression tests include:

1. `test_regression_event_replay_idempotency_no_list_duplication`
2. `test_regression_refund_event_replay_idempotency`
3. `test_regression_cumulative_partial_refunds_and_excess_rejection`
4. `test_regression_boolean_inputs_not_treated_as_integers`

The complete test suite reported 222 passed tests with no failures.

## 6. Future Improvements

All 222 tests passed in my reported test run.

One possible future improvement is to use a proper database or permanent event storage system instead of keeping all information in temporary memory.

Another improvement would be to use tools such as Pydantic or Python dataclasses to validate data types and prevent invalid transaction amounts or reward rules from entering the system.