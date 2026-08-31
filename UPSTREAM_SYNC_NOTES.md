# Upstream sync — manual resolution required

Generated: 2026-08-31T08:09:02Z
Upstream:   https://github.com/NousResearch/hermes-agent.git @ main
Upstream commit: 1cf36398135f4848a1d04b2167ffb564b7881d35
Behind by:  8419 commits

The automated 3-way merge on top of `origin/main` produced conflicts.
The merge was aborted before any conflict markers were committed, so
this branch currently contains only this notes file on top of
`origin/main` — that is by design.

## Conflicting paths

```
.dockerignore
tests/gateway/test_bluebubbles.py
```

## How to resolve

```bash
git fetch origin "chore/upstream-sync-2026-08-31-1cf3639" && git switch "chore/upstream-sync-2026-08-31-1cf3639"
git remote add upstream https://github.com/NousResearch/hermes-agent.git 2>/dev/null || true
git fetch upstream main
git merge upstream/main
# resolve, then:
git rm UPSTREAM_SYNC_NOTES.md
git commit
git push --force origin "chore/upstream-sync-2026-08-31-1cf3639"
```

Then update the PR body / drop draft state and merge.
