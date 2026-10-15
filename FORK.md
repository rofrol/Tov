# This fork

`rofrol/Tov` is a fork of https://github.com/richiemcilroy/Tov. It carries
fixes that are offered upstream as pull requests, so that projects which
need them can build with them before upstream merges them.

## History stays linear

`main` is upstream's `main` plus our commits on top, with no merge
commits. That keeps rebasing onto upstream a single `git rebase`.

- Every change reaches `main` as a fast-forward (`git merge --ff-only`);
  `git pull` uses `--ff-only` too. Never create a merge commit on `main`.
- Each fix starts on its own branch, `fix/NN-short-name` (NN is the upstream
  issue), cut from `upstream/main`, with one logical change and a test. That
  branch is the upstream pull request.
- This file is one of our commits on `main` too; it is never offered upstream.

## Adding a fix

```sh
git fetch upstream
git switch -c fix/NN-name upstream/main
# change, test (cargo test), commit
git push -u origin fix/NN-name            # open the upstream PR from here
git switch main
git rebase main fix/NN-name               # only if main moved past upstream/main
git switch main && git merge --ff-only fix/NN-name
git push origin main
```

If `fix/NN-name` was rebased onto `main` for the fast-forward, the PR branch
keeps its original commit cut from `upstream/main`. Push the rebased copy
under no other name.

## Upstream moved

```sh
git fetch upstream
git switch main
git rebase upstream/main                  # replays our commits on top
cargo test
git push --force-with-lease origin main
```

## Upstream merged one of our fixes

`git rebase upstream/main` drops a commit whose change upstream already has.
If upstream changed the fix while merging it (squashed or edited), resolve
the conflict in upstream's favour, or drop our commit with
`git rebase -i upstream/main`. Then delete `fix/NN-name` here and on
`origin`.

## Pinned commits

Rebasing rewrites the SHAs on `main`, so a project that pins a commit of
this fork tags it first (`git tag pin/<project>/<yyyy-mm-dd> <sha>`,
pushed to `origin`). The tag keeps the commit reachable after `main` is
rebased.

## Carried pull requests

Other people's open upstream PRs that herdr-tov needs are carried on `main`
as one cherry-picked commit each (`git cherry-pick -x`, which keeps the
author and names the PR's commit). They sit right after upstream, before our
own commits. A carried commit is dropped when upstream merges the PR (the
rebase drops it) or closes it (decide then: keep it, or drop it and anything
that needs it).

| PR | upstream head | why we carry it |
|---|---|---|
| https://github.com/richiemcilroy/Tov/pull/17 | `81b211b` | `process.stdin` as a stream with `setRawMode`; our `tty_wrap` work (issue #31) builds on it |

Considered and not carried (consult round `20261013-133024-8c18`): #18
(binary fs), #19 (narrowing) and #16 (wait until needed); #15, #25, #26,
#27 and #3 (no effect on herdr-tov yet); #5 and #6 (not code).

Merged upstream since (dropped by the rebase of 2026-10-15): #14 (carried
until then), #5, #6, #15, #16, #18, #25 and #26.

## Tags

`pin/<project>/<yyyy-mm-dd>.<n>` are lightweight tags that keep a commit
some project pins reachable after `main` is rebased. They name the project
that depends on the commit, so a tag can be deleted once no commit of that
project points at it. They are not releases of this fork.
