# `<Integration>` — Compliance & Quality Audit

> **Template.** Replace `<Integration>` with your integration's name and work the tables
> below as you build. This is a **living document**: update each rule's status as you go,
> and keep every **ignored** item documented with a reason so the decision is auditable
> later. Snapshot date: `<YYYY-MM-DD>`.

This is a sourced compliance checklist covering the Home Assistant **Integration Quality
Scale** (all four tiers) and the HA **Architecture Decision Records (ADRs)**.

## Sources of truth

| Source | URL |
|---|---|
| Quality Scale — rules index | https://developers.home-assistant.io/docs/core/integration-quality-scale/rules |
| Quality Scale — per-rule detail | `https://developers.home-assistant.io/docs/core/integration-quality-scale/rules/{rule-slug}` |
| Architecture Decision Records (ADRs) | https://github.com/home-assistant/architecture/tree/master/adr |
| ADR 0022 — quality scale framework | https://github.com/home-assistant/architecture/blob/master/adr/0022-integration-quality-scale.md |
| Lint ruleset (local) | `pyproject.toml` `[tool.ruff.lint]` + this file |

Rule counts per the index at snapshot: **Bronze 19, Silver 10, Gold 24 (21 enumerated below
— reconcile the 3-rule delta when Gold work begins), Platinum 3.** Re-check the live index;
HA adds/renames rules over time.

## How compliance is enforced

The gate is **CI** (`.github/workflows/validate.yml`): HACS validation → hassfest → ruff →
mypy (strict) → pytest (≥95% coverage). The rule config lives in `pyproject.toml`
(mypy `strict`, the curated ruff `select` + `mccabe max-complexity = 10`, pytest
`--cov-fail-under=95`). This file is the human-readable map from those gates to the
quality-scale rules they satisfy.

## Status legend

- ✅ **done** — satisfied now
- 🔜 **design-time** — committed in the design; implement during the build
- 🛠 **polish-time** — mechanical; enforced continuously by ruff/mypy, no design decision needed
- ⬜ **todo** — not yet addressed
- 🚫 **ignored** — not applicable; reason recorded (see the consolidated register at the end)

> Set your release target up front (e.g. *Silver before first release, Gold before v1.0*),
> then drive the tables toward it. Declare the tier in `manifest.json` (`"quality_scale"`)
> only once every rule in that tier is ✅ or justified 🚫.

---

## 🥉 Bronze (19)

| Rule | Status | What it asks for |
|---|---|---|
| `action-setup` | ⬜ | Register service actions in `async_setup` (with a `services.yaml`), not lazily. |
| `appropriate-polling` | ⬜ | Coordinator sets a sensible `update_interval`; use `always_update=False` when data can be unchanged. |
| `brands` | ⬜ | Logo/icon submitted to `home-assistant/brands` (required to ship via HACS/core). |
| `common-modules` | ⬜ | Standard module layout (`coordinator.py`, `const.py`, an `entity.py` base). |
| `config-flow-test-coverage` | ⬜ | 100% test coverage on the config flow. |
| `config-flow` | ⬜ | UI config flow present (`config_flow: true`); no YAML setup. |
| `dependency-transparency` | ⬜ | Requirements are open and pinned; nothing opaque/closed-source. |
| `docs-actions` | ⬜ | Every service/action documented in the README. |
| `docs-high-level-description` | ⬜ | README explains what the integration does. |
| `docs-installation-instructions` | ⬜ | README install steps (HACS + manual). |
| `docs-removal-instructions` | ⬜ | README how-to-remove section. |
| `entity-event-setup` | ⬜ | Subscribe to events in `async_added_to_hass` / the coordinator; tear down on unload. |
| `entity-unique-id` | ⬜ | Every entity sets a stable `unique_id`. |
| `has-entity-name` | ⬜ | `_attr_has_entity_name = True` on entities. |
| `runtime-data` | ⬜ | Store per-entry state on `entry.runtime_data`, not `hass.data[DOMAIN]`. |
| `test-before-configure` | ⬜ | Validate the connection inside the config flow before creating the entry. |
| `test-before-setup` | ⬜ | Raise `ConfigEntryNotReady` / `ConfigEntryAuthFailed` on a failed first setup. |
| `unique-config-entry` | ⬜ | Prevent duplicate entries (`unique_id` or `single_config_entry`). |

## 🥈 Silver (10)

| Rule | Status | What it asks for |
|---|---|---|
| `action-exceptions` | ⬜ | Service handlers raise `ServiceValidationError`/`HomeAssistantError` on bad input or failure. |
| `config-entry-unloading` | ⬜ | `async_unload_entry` tears down platforms, subscriptions, and timers. |
| `docs-configuration-parameters` | ⬜ | README documents every config/option parameter. |
| `docs-installation-parameters` | ⬜ | README documents each setup parameter. |
| `entity-unavailable` | ⬜ | Entities go `unavailable` when data is stale/unreachable. |
| `integration-owner` | ⬜ | `codeowners` set in `manifest.json`. |
| `log-when-unavailable` | ⬜ | Log **once** on becoming unavailable and once on recovery — not every cycle. |
| `parallel-updates` | ⬜ | Set `PARALLEL_UPDATES` per platform. |
| `reauthentication-flow` | ⬜ | Implement a reauth flow for credential-based integrations. |
| `test-coverage` | ⬜ | Above-average test coverage (this kit gates ≥95%). |

## 🥇 Gold (24 per index; 21 enumerated)

| Rule | Status | What it asks for |
|---|---|---|
| `devices` | ⬜ | Group entities under a `DeviceInfo`. |
| `diagnostics` | ⬜ | Implement a redacted diagnostics dump. |
| `discovery` | ⬜ | Support automatic discovery where the protocol allows. |
| `discovery-update-info` | ⬜ | Update network info (e.g. IP) on rediscovery. |
| `docs-data-update` | ⬜ | Document how/when data is fetched or pushed. |
| `docs-examples` | ⬜ | Provide automation examples. |
| `docs-known-limitations` | ⬜ | Document known limitations. |
| `docs-supported-devices` | ⬜ | List supported devices/models. |
| `docs-supported-functions` | ⬜ | List entities/functions provided. |
| `docs-troubleshooting` | ⬜ | Troubleshooting section. |
| `docs-use-cases` | ⬜ | Use-case examples. |
| `dynamic-devices` | ⬜ | Add newly-appearing devices at runtime. |
| `entity-category` | ⬜ | Set `EntityCategory` (config/diagnostic) where appropriate. |
| `entity-device-class` | ⬜ | Set `device_class` where a natural one exists. |
| `entity-disabled-by-default` | ⬜ | Disable noisy/less-common entities by default. |
| `entity-translations` | ⬜ | Entity translations (`translation_key` + `strings.json`). |
| `exception-translations` | ⬜ | Translatable exception messages (`translation_key`). |
| `icon-translations` | ⬜ | `icons.json`; never `_attr_icon` on translated entities. |
| `reconfiguration-flow` | ⬜ | Provide a reconfigure flow. |
| `repair-issues` | ⬜ | Surface actionable problems as repair issues. |
| `stale-devices` | ⬜ | Remove devices that are gone. |

## 🏆 Platinum (3)

| Rule | Status | What it asks for |
|---|---|---|
| `async-dependency` | ⬜ | The backing library is fully async (no sync I/O). |
| `inject-websession` | ⬜ | Pass HA's shared aiohttp/httpx session into the dependency. |
| `strict-typing` | ⬜ | `mypy --strict` passes with `py.typed` and full annotations. |

---

## ADR compliance

Architecture Decision Records are HA's canonical rationale for the design rules. Map the
ones that apply to your integration; ignore the rest with a reason (next section).

| ADR | Status | What it requires |
|---|---|---|
| [0002](https://github.com/home-assistant/architecture/blob/master/adr/0002-minimum-supported-python-version.md) / 0020 — minimum supported Python | ⬜ | Target the supported Python (set `target-version`/`python_version`). |
| [0005](https://github.com/home-assistant/architecture/blob/master/adr/0005-style-formatting.md) — code formatting | ⬜ | Code is formatted (ruff/black). |
| [0008](https://github.com/home-assistant/architecture/blob/master/adr/0008-code-owners.md) — code owners | ⬜ | `manifest.codeowners` set. |
| [0009](https://github.com/home-assistant/architecture/blob/master/adr/0009-translations-2.0.md) — translations 2.0 | ⬜ | `strings.json` ↔ `translations/en.json`; entity/exception/icon translation keys. |
| [0010](https://github.com/home-assistant/architecture/blob/master/adr/0010-integration-configuration.md) — integration configuration | ⬜ | Device/service integrations configure via UI config flow, not YAML. |
| [0022](https://github.com/home-assistant/architecture/blob/master/adr/0022-integration-quality-scale.md) — quality scale | ⬜ | Adopt the scale; set `manifest.quality_scale` once a tier is met. |

### ADRs ignored as not applicable (with reason)

> Most ADRs are HA-platform/policy decisions (Docker images, hardware screening, GPIO,
> databases, webscraping…) that don't apply to a typical custom integration's code. Record
> the ones you skip and why, e.g.:

| ADR | Reason ignored |
|---|---|
| 0007 — integration config YAML structure | We expose no YAML platform config (config-flow only). |
| 0011 — discovery requires unique id | We implement no discovery. |
| `<ADR …>` | `<reason>` |

---

## Consolidated "ignored / N-A" register

Each entry is a deliberate exemption — revisit if the trigger condition changes. Fill in as
you mark rules 🚫.

| Item | Reason | Revisit when |
|---|---|---|
| `reauthentication-flow` (Silver) | _example:_ no integration-owned credentials. | You add an authenticated cloud/API source. |
| `inject-websession` (Platinum) | _example:_ no HTTP session used. | You add an HTTP-based dependency. |
| `<rule or ADR>` | `<reason>` | `<trigger>` |

## Design-time rules to bake in from the first commit of each file

Cheap now, costly to retrofit:
`runtime-data` · `entity-unique-id` · `has-entity-name` · `entity-event-setup` · typed
`runtime_data` + TypedDicts (config/state/payload) · `available` (entity-unavailable) ·
`action-exceptions` + exception `translation_key`s · `config-entry-unloading` cleanup ·
`parallel-updates` · tz-aware datetimes (DTZ) · lazy `%` logging (G/LOG) · diagnostics
redaction.
