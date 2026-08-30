# Upstream sync — manual resolution required

Generated: 2026-08-30T08:00:09Z
Upstream:   https://github.com/NousResearch/hermes-agent.git @ main
Upstream commit: 26350357d76e4508c8df9304a3374bdc5a6f6220
Behind by:  8342 commits

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
git fetch origin "chore/upstream-sync-2026-08-30-2635035" && git switch "chore/upstream-sync-2026-08-30-2635035"
git remote add upstream https://github.com/NousResearch/hermes-agent.git 2>/dev/null || true
git fetch upstream main
git merge upstream/main
# resolve, then:
git rm UPSTREAM_SYNC_NOTES.md
git commit
git push --force origin "chore/upstream-sync-2026-08-30-2635035"
```

Then update the PR body / drop draft state and merge.
