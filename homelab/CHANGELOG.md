# Changelog

Sancho-Hermes-Agent releases.

## Unreleased

### Added

- Initial homelab/ overlay scaffolding
- Dockerfile installs Hermes from local fork via `uv pip install .`
- k8s manifests refactored for `agents-shared` namespace + floating
  (no nodeSelector) + Longhorn-backed (longhorn-single state,
  longhorn-rwx graphs)
- config/{SOUL,TOOLS}.md + hermes.toml for personal-life agent
- GitHub Actions: build.yml + upstream-sync.yml
- Submodule of l3ocifer/homelab at homelab/shared/

### Migrated from homelab repo

- Previously: argocd/apps/_agents/sancho/ in homelab repo, pinned
  to `alef`, hostPath graph mounts, local-path state PVC. Migrated
  fully to this repo's homelab/k8s/ for floating + Longhorn.
