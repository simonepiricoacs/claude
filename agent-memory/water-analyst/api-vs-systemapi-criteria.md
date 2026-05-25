---
name: api-vs-systemapi-criteria
description: Decision criteria for placing an operation in Api (permission-checked) vs SystemApi (privileged internal)
metadata:
  type: project
---

Place an operation in `*SystemApi` ONLY when ALL these hold:
1. It is called from another service in the same module (or an internal scheduler/lifecycle hook).
2. The caller already has authority to act, but does not naturally hold permissions on the target aggregate.
3. The operation is NEVER exposed via REST.
4. The operation supports a cross-aggregate transition or data-integrity reconciliation.

Otherwise place it in `*Api` and protect with `@AllowPermissions`.

**Why:** SystemApi bypasses the permission system. Over-using it creates silent privilege escalation paths. A pattern observed in HomeLibrary analysis: `BookServiceImpl.markAsLost()` needs to close any open `Loan`, but the caller holds `Book.lost` permission, not `Loan.update` — legitimate SystemApi use. Conversely, "find by ISBN" was a candidate for SystemApi but should remain on `Api` because it is also useful to REST consumers and requires `Book.find` check.

**How to apply:** When listing operations in §3.3.2, write a one-line security justification per SystemApi method. If the justification reads "convenience" or "skip check", move the method to Api.
