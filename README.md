# artificial-man

A **quality-audit framework** and **CI workflows** for building high-quality Home Assistant
custom integrations. Drop these files into your integration repo to get a sourced
Bronze→Platinum compliance checklist and a strict, reproducible CI gate from day one.

**Non-goal:** this is *not* a code template — there is no example integration here. You
bring the integration; this brings the quality scaffolding around it.

What's inside:

- **[`AUDIT.md`](AUDIT.md)** — a living checklist mapping your integration to HA's quality
  scale and ADRs.
- **`.github/workflows/`** + **`pyproject.toml`** + **`requirements_test.txt`** — the CI
  gate (HACS → hassfest → ruff → mypy → pytest ≥95%) plus optional AI-in-CI review.

---

## What the quality audit + ADRs provide

Home Assistant grades integrations on a four-tier **Integration Quality Scale**
(Bronze → Silver → Gold → Platinum) — concrete, testable rules like "every entity has a
`unique_id`" or "raise `ConfigEntryNotReady` on a failed setup". The **Architecture
Decision Records (ADRs)** are HA's canonical *rationale* for the design rules (config via
UI, code owners, translations 2.0, the quality scale itself).

`AUDIT.md` turns both into a single living checklist: every rule, a status, and a one-line
note — plus an honest **"ignored / N-A" register** so each skipped rule has a recorded
reason and a revisit trigger. Example (Bronze excerpt):

| Rule | Status | Notes |
|---|---|---|
| `config-flow` | ⬜ | UI config flow present (`config_flow: true`) |
| `entity-unique-id` | ⬜ | every entity sets a stable `unique_id` |
| `runtime-data` | ⬜ | use `entry.runtime_data`, not `hass.data[DOMAIN]` |
| `test-before-setup` | ⬜ | raise `ConfigEntryNotReady` on failed init |

Status legend: ✅ done · 🔜 design-time · 🛠 polish-time · ⬜ todo · 🚫 ignored (reason recorded).

### Canonical sources

| Source | URL |
|---|---|
| Quality Scale — rules index | https://developers.home-assistant.io/docs/core/integration-quality-scale/rules |
| Quality Scale — per-rule detail | `…/integration-quality-scale/rules/{rule-slug}` |
| ADR index (all records) | https://github.com/home-assistant/architecture/tree/master/adr |
| ADR 0022 — integration quality scale | https://github.com/home-assistant/architecture/blob/master/adr/0022-integration-quality-scale.md |
| hassfest (manifest/strings validation) | https://developers.home-assistant.io/docs/creating_integration_manifest |
| HACS — publishing an integration | https://www.hacs.xyz/docs/publish/integration/ |

---

## Ruff config — above & beyond HA's baseline

The common baseline is `E`/`F`/`W`. `pyproject.toml` selects a curated strict set on top of
it — each group earns its place for an async, event-loop-driven integration:

| Group | Catches | Why it matters for HA |
|---|---|---|
| `ASYNC` | blocking calls inside `async` code | HA is async-first; one blocking call stalls the event loop |
| `DTZ` | naive (tz-less) datetimes | scheduling/time logic must be tz-aware to be correct |
| `G` / `LOG` | f-strings / eager `%` in log calls | matches HA logging guidance (`log-when-unavailable`) |
| `C90` | over-complex functions (`max-complexity = 10`) | keeps coordinator methods small and testable |
| `D` | missing docstrings (pep257) | HA expects module/class docstrings |
| `S` | security smells (bandit) | catches unsafe patterns before review |
| `T20` | stray `print()` | integrations log, never print |
| `B`, `SIM`, `RET`, `UP`, `I`, `N`, `TC`, `TID`, `PTH`, `RUF` | bugs, dead code, style, modern syntax, import hygiene | broad correctness + clean diffs |

Plus `mypy --strict` and a `--cov-fail-under=95` pytest gate.

---

## How we meet HA standards — and where we exceed them

| Gate | HA / HACS baseline | This kit | Verdict |
|---|---|---|---|
| HACS validation | required to list in HACS | `hacs/action` in CI | **Meets** |
| hassfest | required by HA (manifest/strings) | `hassfest` in CI | **Meets** |
| Lint | ruff suggested, minimal select | 20-group curated strict select + complexity cap | **Exceeds** |
| Typing | `strict-typing` is a *Platinum* rule | `mypy --strict` from commit #1 | **Exceeds** (Platinum-level early) |
| Tests | Silver: config-flow 100% + `test-before-setup` | global **≥95%** coverage gate, sockets blocked | **Exceeds** |
| Quality tracking | `quality_scale` manifest field (self-declared) | living Bronze→Platinum audit + ADR tracker + N/A register | **Exceeds** |
| Code review | — | AI-in-CI: auto PR review + `@claude` | **Bonus** |

---

## AI-in-CI (optional)

Two workflows run Claude **inside GitHub Actions**:

- **`claude-code-review.yml`** — reviews every PR (`opened`/`synchronize`) and posts a comment.
- **`claude.yml`** — responds when you write `@claude` in an issue or PR comment; it can
  push commits / open a PR.

Both are **opt-in and inert** until you add an `ANTHROPIC_API_KEY` repository secret
(*Settings → Secrets and variables → Actions*). Usage is billed to that Anthropic API
account. Delete either file if you don't want it. They use
[`anthropics/claude-code-action@v1`](https://github.com/anthropics/claude-code-action).

---

## How to adopt

1. Copy `AUDIT.md`, `.github/workflows/`, `pyproject.toml`, and `requirements_test.txt`
   into your integration repo.
2. Replace `<your_domain>` (in `pyproject.toml` and `validate.yml`) with your integration's
   domain — the folder name under `custom_components/`.
3. Pin `pytest-homeassistant-custom-component` in `requirements_test.txt` to the release
   matching your target HA version.
4. *(optional)* Add the `ANTHROPIC_API_KEY` secret to enable AI-in-CI.
5. Work the `AUDIT.md` tables as you build; set `manifest.json`'s `quality_scale` only once
   a tier is fully ✅ / justified 🚫.

> **Fresh-clone note:** because there's no example integration, the `mypy` and `pytest` jobs
> in `validate.yml` point at a `<your_domain>` placeholder and will fail until you add your
> code and tests. That's expected — CI goes green once your integration is in place.

## License

[MIT](LICENSE)
