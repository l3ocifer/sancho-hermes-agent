# Upstream sync — manual resolution required

Generated: 2026-07-30T08:00:05Z
Upstream:   https://github.com/NousResearch/hermes-agent.git @ main
Upstream commit: c55159f185e0c4a18f4fdaacb666f77d39d10623
Behind by:  1545 commits

The automated 3-way merge on top of `origin/main` produced conflicts.
The merge was aborted before any conflict markers were committed, so
this branch currently contains only this notes file on top of
`origin/main` — that is by design.

## Conflicting paths

```
tests/gateway/test_bluebubbles.py
```

## How to resolve

```bash
git fetch origin "chore/upstream-sync-2026-07-30-c55159f" && git switch "chore/upstream-sync-2026-07-30-c55159f"
git remote add upstream https://github.com/NousResearch/hermes-agent.git 2>/dev/null || true
git fetch upstream main
git merge upstream/main
# resolve, then:
git rm UPSTREAM_SYNC_NOTES.md
git commit
git push --force origin "chore/upstream-sync-2026-07-30-c55159f"
```

Then update the PR body / drop draft state and merge.
