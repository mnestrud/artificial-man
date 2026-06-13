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

## The standards we hold to

This kit holds an integration to four external, authoritative standards — not house rules.
Everything in `AUDIT.md` and the CI gate traces back to one of these:

### 1. The Integration Quality Scale (HA's built-in tiers)

Home Assistant's official, four-tier grading system for integrations, defined in
[ADR 0022](https://github.com/home-assistant/architecture/blob/master/adr/0022-integration-quality-scale.md)
and published as a [rules index](https://developers.home-assistant.io/docs/core/integration-quality-scale/rules).
Each tier is a set of **concrete, testable rules**; a tier is achieved only when every rule
in it *and all lower tiers* is satisfied or justified N/A. You declare the tier you've
reached in `manifest.json`'s `quality_scale` field.

| Tier | Theme | Representative rules |
|---|---|---|
| 🥉 **Bronze** | baseline correctness | `config-flow`, `entity-unique-id`, `runtime-data`, `test-before-setup` |
| 🥈 **Silver** | robustness | `entity-unavailable`, `reauthentication-flow`, `config-entry-unloading`, `test-coverage` |
| 🥇 **Gold** | polish & UX | `devices`, `diagnostics`, `discovery`, entity/exception/icon translations, `reconfiguration-flow`, `repair-issues` |
| 🏆 **Platinum** | engineering excellence | `async-dependency`, `inject-websession`, `strict-typing` |

### 2. Architecture Decision Records (ADRs)

HA's canonical, versioned record of **why** the platform's rules exist — the rationale and
constraints behind integration design (config via UI not YAML, code owners, translations
2.0, "discovery requires a unique id", the quality scale itself). They're the authority you
cite when a design choice is questioned. Full index:
[home-assistant/architecture](https://github.com/home-assistant/architecture/tree/master/adr).

### 3. HACS publishing standards

The [Home Assistant Community Store](https://www.hacs.xyz/docs/publish/integration/) has its
own requirements for a repo to be installable and listed: a valid `hacs.json`, the
integration under `custom_components/<domain>/` with a complete `manifest.json`, a README, a
GitHub release/topic, and brand assets submitted to
[`home-assistant/brands`](https://github.com/home-assistant/brands). The `hacs/action`
GitHub Action validates these in CI.

### 4. hassfest

HA core's own validator for `manifest.json`, `strings.json`, `services.yaml`, and
translations — it enforces that they're well-formed and internally consistent. Run via the
`home-assistant/actions/hassfest` action. Reference:
[manifest docs](https://developers.home-assistant.io/docs/creating_integration_manifest).

**In short:** `AUDIT.md` tracks your progress against the **Quality Scale + ADRs**; the CI
workflows enforce **HACS + hassfest** (plus strict lint/type/test). The rest of this README
shows exactly where the kit *meets* these standards and where it deliberately *exceeds* them.

---

## What the quality audit + ADRs provide

`AUDIT.md` turns the Quality Scale and ADRs into a single living checklist: every rule, a status, and a one-line
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
