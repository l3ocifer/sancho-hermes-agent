# Upstream sync — manual resolution required

Generated: 2026-08-25T08:00:07Z
Upstream:   https://github.com/NousResearch/hermes-agent.git @ main
Upstream commit: 4c1f53be10d0fce1d25aee1975e5149b6c54f25a
Behind by:  7206 commits

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
git fetch origin "chore/upstream-sync-2026-08-25-4c1f53b" && git switch "chore/upstream-sync-2026-08-25-4c1f53b"
git remote add upstream https://github.com/NousResearch/hermes-agent.git 2>/dev/null || true
git fetch upstream main
git merge upstream/main
# resolve, then:
git rm UPSTREAM_SYNC_NOTES.md
git commit
git push --force origin "chore/upstream-sync-2026-08-25-4c1f53b"
```

Then update the PR body / drop draft state and merge.
