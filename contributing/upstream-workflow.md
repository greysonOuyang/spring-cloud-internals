# Upstream contribution workflow

## Git topology

For each Spring Cloud fork:

- `origin`: `https://github.com/greysonOuyang/<repo>.git`
- `upstream`: `https://github.com/spring-cloud/<repo>.git`

Before new work:

```bash
git fetch upstream
git switch main
git reset --hard upstream/main
git push --force-with-lease origin main
git switch -c gh-<issue>-short-topic
```

The force update above is only for the personal fork's `main`, never an upstream branch.

## Investigation, teaching, and approval gate

1. Confirm the issue still exists on `upstream/main`.
2. Search open and closed PRs for the issue number and root-cause keywords.
3. Reproduce the problem locally with the narrowest useful runtime path or regression test.
4. Explain the reproduction to the user: inputs, execution path, expected behavior, actual behavior, and the exact exception or wrong result.
5. Trace the source path and identify the violated invariant. Explain the framework/runtime mechanism, not merely the failing line.
6. Present the candidate fix with alternatives, compatibility risks, and why the preferred design addresses the root cause.
7. Obtain explicit user approval of the proposed solution before preparing an upstream PR. Local experiments and prototype patches may be used before approval, but no PR is opened.
8. Implement the approved coherent fix and regression coverage.
9. Review the final diff with the user-facing explanation updated to match the actual implementation.
10. Run focused checks, then broader project-required checks as appropriate.
11. Only after the user has understood the problem and approved the final solution may an upstream PR be opened.

For design-sensitive issues, comment on the upstream issue before implementation when maintainer intent is ambiguous or multiple API semantics are plausible. A maintainer reply is not a universal prerequisite for every PR; it is a synchronization gate when design direction materially affects the patch.

## Commit/PR hygiene

Spring projects use DCO. Commit with:

```bash
git commit -s -m "Fix ..."
```

PR descriptions should contain:
- exact problem;
- root cause;
- why the change is correct;
- regression test evidence;
- compatibility concerns;
- `Fixes gh-<number>` when appropriate for that repository's convention.

Do not flood maintainers with speculative AI-generated alternatives. A small verified argument is more useful than a wall of confidence-shaped text.
