# Payment state transitions and invariants

This note makes the authorize/capture lifecycle explicit for integrators and reviewers.
It is intentionally limited to the state machine described by the protocol; it does not
replace the Solidity contracts or a security audit.

## State model

| State | Meaning | Valid next states |
| --- | --- | --- |
| Authorized | Funds are reserved for a payment | Captured, Voided, Reclaimed |
| Captured | Funds have been transferred to the merchant | Refunded |
| Voided | The operator cancelled the authorization | Terminal |
| Reclaimed | The payer recovered an expired authorization | Terminal |
| Refunded | Captured funds were returned to the payer | Terminal |

Charge is the immediate path that combines authorization and capture. It should be
reasoned about as a transition to Captured, not as a second authorization.

## Invariants to preserve

- A capture must refer to an existing authorization and must not exceed its remaining amount.
- A void and a reclaim release an authorization; neither may be followed by capture.
- A refund requires captured funds and cannot be used to create a new authorization.
- Repeating a terminal operation must fail safely or remain without additional effect.
- Expiry is evaluated against the authorization window, not against a later capture request.
- The token collector remains part of the authorization boundary: its transfer result must
  agree with the amount recorded by AuthCaptureEscrow.

## Review checklist

When adding a payment flow or integration, inspect the operation and collector together:

1. Identify the authorization identifier and its remaining amount.
2. Check the caller role and the expiry condition for the requested transition.
3. Confirm that the event emitted by the contract describes the same amount and recipient.
4. Verify that a retry cannot cross a terminal state or pay twice.

The detailed operation pages are [Authorize](operations/Authorize.md),
[Capture](operations/Capture.md), [Void](operations/Void.md),
[Reclaim](operations/Reclaim.md), and [Refund](operations/Refund.md).
The contract-level behavior is implemented in src/AuthCaptureEscrow.sol and the
collector boundary is implemented in the files under src/collectors/.

This is a documentation contribution based on static source reading. It does not claim
that tests, deployment, or an audit were performed.
