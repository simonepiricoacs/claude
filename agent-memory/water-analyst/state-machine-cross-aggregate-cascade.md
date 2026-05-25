---
name: state-machine-cross-aggregate-cascade
description: Pattern for cascading state transitions across two aggregates via SystemApi pairs (e.g. Book status ↔ Loan open/close)
metadata:
  type: project
---

When two aggregates have mutually-dependent state (one's status mirrors the other's lifecycle), expose a SystemApi method on each side and have each `*ServiceImpl` depend on the sibling's SystemApi. Never let the sibling carry the permission burden of the caller.

Canonical example (HomeLibrary):
- `BookSystemApi.transitionStatusInternal(bookId, BookStatus)` — invoked by `LoanServiceImpl` when a loan opens/closes.
- `LoanSystemApi.forceCloseOpenLoanForBook(bookId, ReturnReason)` — invoked by `BookServiceImpl.markAsLost`.

**Why:** Without this paired SystemApi, the caller would need `Book.update` AND `Loan.update` permissions even though it is invoking a narrow legitimate transition. Forcing the broad permission is a silent privilege escalation and breaks role separation (`librarian` who can lend should not also need to mutate book metadata).

**How to apply:** Whenever §3.1's state-machine diagram crosses an aggregate boundary, add the matching `*SystemApi.<actionInternal>(...)` methods in §3.2. Pair them: caller-side calls callee-side SystemApi, not the other way around. Always co-locate the two writes in the same transaction (single `@Transactional` boundary at the originating service method). Link to [[api-vs-systemapi-criteria]].
