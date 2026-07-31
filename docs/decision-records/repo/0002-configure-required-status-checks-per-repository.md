# ADR-repo/0002: Configure Required Status Checks Per-Repository

| Field | Value |
| --- | --- |
| Status | Accepted |
| Date | 2026-07-12 |
| Authors | Nick Warila (@NWarila) |
| Decision-maker | Nick Warila (sole portfolio maintainer) |
| Consulted | GitHub ruleset platform limits; framework injection logic; a fleet-wide survey of pull-request check-context names. |
| Informed | Maintainers adding `required_checks` to repository inventory. |
| Reversibility | High |
| Review-by | N/A (Accepted) |

## TL;DR

Required status checks are configured **per repository** through the inventory
`required_checks` key, which the framework injects into that repo's Pull Request
Gate ruleset. Each list names only the check contexts that repo actually reports
on pull requests. There is deliberately **no** single account-wide rule and no
shared default: the account is a GitHub **User** (personal) account, which has no
multi-repo ruleset primitive, and the fleet has **no check-context name shared by
all repositories**, so any shared default would block some repositories' pull
requests. A framework-level default remains a documented future option, gated on
first standardizing CI context names.

## Context and Problem Statement

The framework supports an opt-in per-repo `required_checks` list in the runner
inventory (`terraform/public/*.yml`). When present on a repo that carries a
`pull_request` rule, the framework injects those strings as a
`required_status_checks` rule on that repo's **Pull Request Gate** ruleset
(non-strict; `do_not_enforce_on_create = true`). `github-terraform-runner` and
`ubi9-base-micro` use it today.

The natural question is whether this can be a **generic, account-wide rule**
instead of a list repeated per repository. It cannot, for two independent
reasons, and a third that constrains any future work:

1. **Platform.** The `NWarila` account is a GitHub **User** account. Rulesets
   that target many repositories at once are an **Organization** feature (and
   multi-repository targeting is Enterprise-gated). A User account can only apply
   **per-repository** rulesets. There is no account-wide required-checks
   primitive. (`nwarila-platform` is an Organization and could use an org
   ruleset, but that does nothing for the personal-account repositories where the
   base images and templates live.)

2. **Semantics.** Required status checks match by **literal context name** — no
   wildcard, no "require whatever ran." A required context a repository never
   emits leaves every pull request there at "waiting for status to be reported"
   and blocks merge indefinitely. Across the 14 managed repositories there is
   **no context name shared, with identical spelling, by all or nearly all** of
   them: the strongest common name (`actionlint`) appears on 9 of 14 and would
   deadlock the other 5, and conceptually-identical checks carry divergent
   literal spellings (`Markdown lint` vs `markdownlint`; `dependency review` vs
   `dependency-review`; three different CodeQL spellings). The repositories fall
   into families — a coherent framework/template cluster that shares a real set,
   a UBI9 group whose members share almost nothing, and several singletons —
   rather than one uniform set.

3. **Framework shape.** The framework injects `required_checks` per repository
   only; there is no `default_required_checks` variable, no per-type map, and no
   organization-ruleset resource.

## Decision Drivers

- Never require a context a repository does not emit (that permanently blocks its
  pull requests).
- Keep each repository's gate list explicit and reviewable.
- Prefer a policy that works on a personal (User) account today, not one that
  assumes org-level machinery.
- Leave a clean path to reduce repetition later without a risky fleet-wide change.

## Considered Options

1. Per-repository `required_checks` lists (each names only what that repo emits).
2. A framework `default_required_checks` unioned into every repository's Pull
   Request Gate ruleset.
3. An organization-level ruleset applied across repositories.

## Decision Outcome

Chosen option: **Option 1, per-repository `required_checks` lists.**

Each repository lists exactly the check contexts it reports on pull requests, and
only those. For `ubi9-base-micro` that is the twelve contexts that report on
every normally scheduled pull-request run (`actionlint`, `analyze Python tools`,
`build and hardening`, `CodeQL`, `dependency review`, `pre-commit`,
`python / required`, `repo contract`, `reproducibility gate (amd64)`,
`reproducibility gate (arm64)`, `slsa generator tag integrity`, `zizmor`). The
`python / required` reducer reports on every normally scheduled pull-request run,
while its upstream Python evidence jobs run only when change detection selects
Python-related paths. Publish-only jobs that skip on pull requests are excluded
so they cannot deadlock a merge.

## Pros and Cons of the Options

### Option 1: Per-repository lists (chosen)

- Good, because each list can name only contexts the repository provably emits,
  so no pull request can deadlock.
- Good, because it works on a User account with no org machinery.
- Good, because it is trivially reversible per repository.
- Bad, because repositories that share checks repeat those names across inventory
  files.

### Option 2: Framework `default_required_checks`

- Good, because a shared set would be written once and fan out to every Pull
  Request Gate ruleset (a small locals change: a new variable unioned into the
  normalized `required_checks` before injection, with a per-repo opt-out).
- Bad, because the apply reconciles **all** repositories at once, so a default
  naming a context some repository does not emit would wedge many repositories'
  pull requests in a single apply.
- Bad, because today **no** context name is emitted by every repository, so there
  is no safe payload for such a default without first standardizing names.

### Option 3: Organization-level ruleset

- Good, because org rulesets can carry required status checks across many repos.
- Bad, because it does not exist for a User account, so it cannot cover the
  personal-account repositories at all.

## Confirmation

This decision is confirmed when:

- each repository's live Pull Request Gate ruleset carries exactly the contexts
  in its inventory `required_checks` (and none that the repository fails to emit);
  and
- no organization ruleset or shared default is introduced without first meeting
  the standardization prerequisite in Consequences.

## Consequences

- Gate lists are explicit and safe; the cost is repetition across repositories
  that share checks.
- A future de-duplication step is available and cheap in code: add a framework
  `default_required_checks` variable unioned into every Pull Request Gate ruleset
  (with a per-repo opt-out). It is **deferred and gated on first standardizing CI
  context names across the fleet** — ideally converging each repository on a
  single aggregate gate context — so that any shared default names only contexts
  every targeted repository provably emits. Even after standardization a shared
  default is realistic per **family** (the framework/template cluster) rather than
  across the whole account.
- Related operational note: the framework's secure-by-default `allow_forking =
  false` for public personal-account repositories is **not persisted by the GitHub
  API** (the update returns success but the setting stays enabled), so those
  repositories show a recurring no-op `allow_forking` change on every plan. This
  is independent of required checks and does not affect them.

## Assumptions

- `NWarila` remains a User (personal) account managed by this runner.
- `nwarila-platform/github-terraform-framework` remains the owner of the ruleset
  injection logic.

## Supersedes

None.

## Superseded by

None.

## Related ADRs

- [ADR-repo/0001](0001-thin-runner-deployer-scope.md)

## Compliance Notes

This ADR records how required status checks are configured across the account's
repositories. It does not change which repositories are managed; `required_checks`
edits remain ordinary repo-data pull requests under `terraform/`.
