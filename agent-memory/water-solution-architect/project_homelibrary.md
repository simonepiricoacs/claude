---
name: project-homelibrary
description: HomeLibrary (Biblioteca Casalinga) — embeddable Water module designed 2026-05-23
metadata:
  type: project
---

HomeLibrary is a single-tenant, single-copy domestic book-catalog Water module covering Authors, Books, and Loans (one open loan per book, enforced via partial unique index).

**Why:** Greenfield Water module to demonstrate cross-aggregate cascade between Book and Loan (markAsLost forces loan closure; loan closure with condition=LOST flips Book.status to LOST). Cascade must happen through `*SystemApi` (privileged path) to avoid permission-double-check and prevent circular dependency between `BookApi` and `LoanApi`.

**How to apply:**
- Module layout: 3 sub-modules only (`-model`, `-api`, `-service`). NO `-application` — embeddable library.
- Roles fixed: `homelibrary_viewer`, `homelibrary_librarian`, `homelibrary_admin`.
- JWT: external issuer; app validates only — never embed login endpoint.
- Runtime: multi-runtime (Spring Boot + OSGi + Quarkus). No runtime-specific code in `-service`.
- Cross-aggregate cascade rule: `BookServiceImpl` depends on `LoanSystemApi`; `LoanServiceImpl` depends on `BookSystemApi`. Each impl exposes its own `*SystemApi` via `@FrameworkComponent(services={...})`. The two `*SystemApi` interfaces sit in `-api` and break the dependency cycle because both impls depend only on interfaces.
- REST base path family: `/api/homelibrary/{authors|books|loans}` — never include `/water` prefix (CXF base context). See [[feedback-rest-path-prefix]].
- Custom permission actions to register at `@OnActivate`: `Book.lost`, `Book.archive`, `Loan.close`. Standard CRUD auto-registered by Water.
- Loan invariant enforced at DB level: partial unique index on `(book_id) WHERE return_date IS NULL`.
- Optimistic locking via `@Version` on Book and Loan.

Module is in `/Users/aristide-cittadino/Documents/Workspace/AcSoftware/Water-framwork/source/HomeLibrary/`. Functional spec at `HomeLibrary/doc/analysis/home-library-functional-analysis.md`.
