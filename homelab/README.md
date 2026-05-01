# Sancho — Hermes-Agent personal-life agent

This is **Leo's fork of [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)**,
extended to run as `Sancho` (the agent that handles Leo's personal
life: family, custody, calendar, kids' care, email, iMessage, home
automation) inside [Leo's homelab K3s cluster](https://github.com/l3ocifer/homelab).

## Layout

```
sancho-hermes-agent/                 ← repo root (this fork)
├── (upstream hermes-agent source)
│   ├── agent/
│   ├── acp_adapter/
│   ├── acp_registry/
│   ├── pyproject.toml
│   └── ...
└── homelab/                          ← everything we add
    ├── Dockerfile                    ← Python 3.13 + pip install . from local
    ├── k8s/                          ← kustomize tree
    ├── config/                       ← SOUL.md, TOOLS.md, hermes.toml
    ├── shared/                       ← submodule → l3ocifer/homelab
    ├── .github/workflows/
    ├── PATCHES.md, CHANGELOG.md, README.md
```

## Sancho's persona, in 30 seconds

Patient, observant, deeply protective of the kids. Calendar coordinator
between custody schedules, school events, doctor appointments. Email
triage. Home Assistant control. Tells Leo what matters and what
doesn't. See `config/SOUL.md`.

## Required env vars

Provided by `sancho-secrets` SealedSecret in `agents-shared`
namespace. See `config/hermes.toml` for the full reference.

## License

Hermes-Agent upstream: MIT (see [LICENSE](../LICENSE)).
Homelab additions in `homelab/`: same.
Persona text in `config/SOUL.md` is Leo Paska's IP.
