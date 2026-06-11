# TOOLS.md — Sancho

What's wired up, where it lives, and how to use it. Lives in `~/.hermes/ workspace/TOOLS.md` after deploy.

---

## Runtime

- **Framework**: Hermes Agent v2026.4.23 (`pip install hermes-agent`)
- **Image**: `ghcr.io/l3ocifer/hermes-sancho:v2026.4.23`
- **Pod**: `sancho/sancho` Deployment, nodeSelector `kubernetes.io/ hostname: alef`, single replica
- **State**: PVC `sancho-state` 5Gi RWO at `/home/hermes/.hermes`
- **Logseq graph**: PVC `sancho-graph` RWX at `/data/graphs/sancho`
(and read-only mounts of `frick-graph`, `frack-graph`, `leo-graph`
at `/data/graphs/{frick,frack,leo}`)
- **Gateway**: HTTPS on port 3001 → `sancho.leopaska.xyz` via Traefik
IngressRoute, Authelia in front
- **Logs**: stdout to Vector → Loki, query at
`grafana.leopaska.xyz` with `{namespace="sancho"}`

## Models


| Alias              | When to use                                                                   | Endpoint                                                    |
| ------------------ | ----------------------------------------------------------------------------- | ----------------------------------------------------------- |
| `litellm/chat`     | Default. Day-to-day conversation.                                             | `https://llm.leopaska.xyz/v1` (key in `LITELLM_API_KEY`)    |
| `litellm/long`     | When context > 64k tokens (briefings that summarize a week of activity, etc.) | same                                                        |
| `litellm/code`     | Rare — only if Leo asks for a code review/draft and Frack isn't around        | same                                                        |
| `litellm/frontier` | Opt-in for high-stakes daily synthesis (pin in `hermes.toml`)                 | same — Qwen3-Coder 480B MoE on blade (CPU-only, ~3-5 tok/s) |


Set in `hermes.toml` under `[models]`. Switch interactively with
`hermes model`.

## Channels


| Channel            | How to use it                                                                                             | Inbound                                                            | Outbound                                             |
| ------------------ | --------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ | ---------------------------------------------------- |
| **iMessage**       | Primary. Leo texts me, I reply on iMessage.                                                               | Webhook from BlueBubbles MacBook → cluster proxy → Hermes gateway. | Hermes → cluster BlueBubbles proxy → MacBook → APNs. |
| **Matrix**         | Always-on fallback for when MacBook is closed. `@sancho:leopaska.xyz`.                                    | Direct Matrix client subscription.                                 | Direct Matrix send.                                  |
| **Telegram**       | Tertiary, when Leo is overseas / iMessage unreliable. `/sancho` prefix on shared homelab bot.             | Hermes Telegram gateway.                                           | Same.                                                |
| **ntfy**           | Push notifications only — for nudges, reminders, "you said you'd leave at 4". `ntfy.leopaska.xyz/sancho`. | n/a (one-way)                                                      | HTTP POST.                                           |
| **Home Assistant** | `conversation.sancho` entity. "hey sancho, when's my next meeting".                                       | HA voice pipeline → `conversation.process` → Hermes HTTP.          | Same.                                                |


Routing: a request that comes in on iMessage gets answered on
iMessage (channel-stickiness). Cross-channel only when explicitly
asked or when iMessage is unavailable.

## Cluster services


| Service            | URL (in-cluster)                                                                        | Why                                                                                     |
| ------------------ | --------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| LiteLLM            | `http://litellm.inference.svc.cluster.local:4000/v1`                                    | All inference                                                                           |
| MCP devops         | `http://external-mcp.ironclaw.svc.cluster.local:8890`                                   | Read-only kubectl, prom queries                                                         |
| Agent Tool Service | `http://agent-tool-service.agents-shared.svc.cluster.local:8080`                        | Self-hosted web search/extract wrapper                                                  |
| SearXNG            | `http://searxng.agents-shared.svc.cluster.local:8080`                                   | No-key internal search backend                                                          |
| Postgres           | `postgres://hermes_sancho@homelab-pg-rw.databases.svc.cluster.local:5432/hermes_sancho` | Memory back-end (sealed in `sancho-secrets`)                                            |
| ntfy               | `https://ntfy.leopaska.xyz/sancho`                                                      | Push to Leo's phone                                                                     |
| Conduit            | `https://conduit.leopaska.xyz`                                                          | Matrix                                                                                  |
| BlueBubbles proxy  | `http://bluebubbles-proxy.agents-shared.svc.cluster.local:8080`                         | iMessage                                                                                |
| Vaultwarden        | `https://warden.leopaska.xyz`                                                           | Credential lookups via the `bw` CLI; persistent creds arrive via SealedSecret → envFrom |


## Skills (loaded from `~/.claude/skills/`)

The Hermes pod mounts `unified-ai-configs/skills/` and loads:


| Skill                             | Use case                                                                                                                                                                                                                                                                      |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `imsg`                            | iMessage read/list — but **send** goes through BlueBubbles proxy, not via the local `imsg` CLI which doesn't exist in the pod                                                                                                                                                 |
| `himalaya`                        | Email read/draft across all configured accounts (config in sealed `sancho-himalaya-config`)                                                                                                                                                                                   |
| `1password` (Vaultwarden adapter) | `bw get item`, `bw list items` — read-only against `https://warden.leopaska.xyz`. Session is unlocked at pod start with `BW_CLIENTID`/`BW_CLIENTSECRET` (sealed) and re-locks on idle. The skill name is historical — the implementation talks to Vaultwarden, not 1Password. |
| `obsidian`                        | Cross-graph reads (Logseq is markdown, the obsidian skill works for both)                                                                                                                                                                                                     |
| `weather`                         | Daily forecast in morning briefing                                                                                                                                                                                                                                            |
| `spotify-player` (`spogo`)        | Music control via HA bridge — handy for "put on focus music"                                                                                                                                                                                                                  |
| `slack`                           | **DISABLED** — Sancho stays out of work Slack                                                                                                                                                                                                                                 |
| `discord`                         | Optional — only on Leo's personal server, not work / customer servers                                                                                                                                                                                                         |
| `commit-helper`                   | If Leo dictates a commit message in conversation                                                                                                                                                                                                                              |
| `session-logs`                    | Cross-session search of Sancho's own past conversations                                                                                                                                                                                                                       |


## Web Search

Use the internal service first. It is self-hosted and needs no API key:

```bash
curl -s "$AGENT_TOOL_SERVICE_URL/search?q=school+calendar&limit=5"
```

For page text after search:

```bash
curl -s -X POST "$AGENT_TOOL_SERVICE_URL/extract" \
  -H 'content-type: application/json' \
  -d '{"url":"https://example.com","max_chars":6000}'
```

`SEARXNG_URL` points at the raw SearXNG instance when a skill needs the
native `/search?format=json` API directly.

## Memory layout


| Path (in pod)            | What                                                  | Owner                                                                            |
| ------------------------ | ----------------------------------------------------- | -------------------------------------------------------------------------------- |
| `/home/hermes/.hermes/`  | Hermes state — SQLite FTS5 sessions, skills, memories | Sancho (RW)                                                                      |
| `/data/graphs/sancho/`   | Sancho's Logseq graph                                 | Sancho (RW)                                                                      |
| `/data/graphs/leo/`      | Leo's PKM                                             | Sancho (R + write to `pages/world/{calendar-context,people,open-loops}.md` only) |
| `/data/graphs/frick/`    | Frick's private graph                                 | Sancho (R only)                                                                  |
| `/data/graphs/frack/`    | Frack's private graph                                 | Sancho (R only)                                                                  |
| Postgres `hermes_sancho` | Vector memory (pgvector)                              | Sancho (RW)                                                                      |


`memorySearch.extraPaths` in `hermes.toml` lists all 4 graphs (own +
3 read-only) with appropriate weighting.

## Cron schedule (in `hermes.toml`)


| Time (America/New_York) | Task                                                                                                              | Delivery                                |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
| 03:50 daily             | Memory consolidation — review yesterday's journal, distill to `pages/ai-memory/Sancho/decisions.md`, prune SQLite | none                                    |
| 06:00 daily             | Calendar refresh — pull next 24h, write to `pages/world/calendar-context.md`                                      | none                                    |
| 06:30 daily             | Inbox triage — flag unread P1+ emails, draft replies for the morning briefing                                     | none                                    |
| 07:00 daily             | **Morning briefing**                                                                                              | BlueBubbles/iMessage                    |
| 12:00 daily             | Midday check — any unanswered iMessages from the morning, calendar conflict watch                                 | BlueBubbles/iMessage if anything urgent |
| 18:00 daily             | Evening recap — what got done today, what's open, prep for tomorrow                                               | iMessage                                |
| every :20 :50           | Heartbeat — calendar conflicts, deadline approach, follow-ups due                                                 | ntfy if any P1+                         |


Stagger from Frick (:00 :30) and Frack (:10 :40) per HANDOFF.md §7.

## Quiet hours

`quiet_hours = "23:00-07:00 America/New_York"` in `hermes.toml`.

During quiet hours:

- ntfy pushes suppressed (queued for 07:00)
- iMessage outbound suppressed (queued)
- Matrix posts allowed only with explicit `:p0::` tag
- Heartbeat checks continue silently (write to journal only)
- Memory consolidation continues (silent)

## Hard-kill

Sentinel: `/data/HARDSTOP-SANCHO` (in `sancho-state` PVC)

```bash
kubectl -n sancho exec deploy/sancho -- touch /data/HARDSTOP-SANCHO
# wait for pod to exit cleanly (Hermes finishes in-flight tool call)
kubectl -n sancho get pod  # should show Completed
# to revive:
kubectl -n sancho exec deploy/sancho -- rm /data/HARDSTOP-SANCHO
kubectl -n sancho delete pod -l app=sancho
```

## Common operations

```bash
# tail Sancho's live thoughts
kubectl -n sancho logs -f deploy/sancho

# open a Hermes shell
kubectl -n sancho exec -it deploy/sancho -- hermes

# trigger morning briefing manually
kubectl -n sancho exec deploy/sancho -- hermes cron run morning-briefing

# inspect memory
kubectl -n sancho exec deploy/sancho -- hermes memory search "alice"

# update persona files (after edits in this repo)
cd ~/git/homelab && git pull
kubectl -n sancho rollout restart deploy/sancho
```

## Update protocol

To update Sancho's persona:

1. Edit `openclaw-configs/sancho/{SOUL,TOOLS}.md` in this repo
2. Commit + push
3. The `sancho-persona` ConfigMap in
  `[argocd/apps/_agents/sancho/configmap-persona.yaml](../../  argocd/apps/_agents/sancho/configmap-persona.yaml)` auto-rolls
   the deployment via ArgoCD (or `kubectl rollout restart deploy/  sancho` for an immediate flip)
4. Sancho re-reads the new SOUL.md on next session start

To update `hermes.toml` (runtime config — model, channels, cron):

1. Edit `openclaw-configs/sancho/hermes.toml` in this repo
2. Commit + push
3. ArgoCD applies the new ConfigMap; Hermes hot-reloads the config
  (or restart the pod to be safe)

## Things that are NOT here yet

- BlueBubbles relay on the MacBook — Phase 1 day 1 manual install
step (see substrate setup notes)
- `agents@leopaska.xyz` email subdomain — Phase 2
- Mac mini for 24/7 BlueBubbles relay — Phase 2 hardware
- Voice mode via HA satellite — Phase 2 once HA is wired
- Twilio number — never planned (use iMessage + Matrix + Telegram)


## Source control & GitOps (fleet convention)

- **Forgejo — `https://git.leopaska.xyz` — is the source of truth** for
  every repo: homelab, all agent repos, business apps. Clone/push via
  `origin` (`git@git-ssh.leopaska.xyz` SSH or HTTPS).
- **GitHub (`l3ocifer/*`) is a push-mirror backup only.** Never push,
  open issues, or open PRs on GitHub — mirroring from Forgejo is
  automatic and one-way.
- **All deploys are GitOps via ArgoCD** (`argocd.leopaska.xyz`):
  commit → push to Forgejo `main` (or PR) → CI builds the image →
  ArgoCD (+ Image Updater) rolls it. Never `kubectl apply` desired
  state by hand; self-heal reverts live edits. Manual
  `rollout restart` is fine when config in git already changed.
- **Issue intake:** Forgejo issues/comments are webhooked through
  agent-bus to the routed agent's inbox (`pages/inbox/`) with a
  `task_id: forgejo-<repo>-<n>`. Routing: `agent:<name>` label →
  per-repo route → repo-name prefix → vetinari (triage default).
- **Acting on issues:** use the Forgejo API with `$FORGEJO_TOKEN`
  (in this agent's k8s Secret, scopes `write:issue,write:repository`):

  ```bash
  # comment your result
  curl -s -X POST -H "Authorization: token $FORGEJO_TOKEN" \
    -H 'Content-Type: application/json' -d '{"body":"<result>"}' \
    https://git.leopaska.xyz/api/v1/repos/<owner>/<repo>/issues/<n>/comments
  # close when resolved
  curl -s -X PATCH -H "Authorization: token $FORGEJO_TOKEN" \
    -H 'Content-Type: application/json' -d '{"state":"closed"}' \
    https://git.leopaska.xyz/api/v1/repos/<owner>/<repo>/issues/<n>
  ```
- **File new work as Forgejo issues** (not GitHub, not ad-hoc notes)
  so it routes through the same intake to the right agent.
