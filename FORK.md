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
