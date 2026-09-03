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

## Investigation before patch

1. Confirm issue still exists on `upstream/main`.
2. Search open and closed PRs for the issue number and root-cause keywords.
3. Find the narrowest existing test class that exercises the mechanism.
4. Add/derive a failing regression test.
5. Trace the source path and identify the violated invariant.
6. Only then implement the smallest coherent fix.

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
