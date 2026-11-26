# Neoguard OS (NetGuard) – Functionality Overview

## Project Vision
Neoguard OS (formerly NetGuard) is a Windows‑focused network‑security agent that operates from the system tray. It combines real‑time connection monitoring, AI‑augmented threat verdicts, self‑hardening protections, and a modular training lab for continuous improvement.

## Core Modules

| Module | File | Primary Function |
|--------|------|------------------|
| **Entry Point** | `netguard/__main__.py` | CLI/tray launcher; decides whether to run GUI or headless diagnostic mode. |
| **Connection Monitor** | `netguard/connection_monitor.py` | Enumerates all outbound/inbound sockets, enriches IPs/processes with PTR/ASN/org/datacenter flags, emits events for rule evaluation. |
| **Firewall Engine** | `netguard/firewall_engine.py` | Evaluates rules stored in SQLite (`netguard.db`) against live connection data; issues allow/block decisions requiring admin privileges. |
| **AI Sentinel LLM** | `netguard/sentinel_ai.py` | Connects to a local LM Studio server (OpenAI‑compatible) using model `netguard-llm`; generates contextual alerts, RAG‑based learning from logged events. |
| **Persistence Layer** | `netguard/database.py` | SQLite database with WAL mode; stores firewall rules, IP intel, app settings (`allow_dirs`, `autostart`, profile lockdown). All access goes through a lock‑guarded connection with busy timeout. |
| **Tamper Guard** | `netguard/tamper_guard.py` (and optional `icacls.exe`) | Protects runtime files, the SQLite config table (`sentinel_mode`), `allow_exe` entries, and launcher executables from unauthorized modification. Can lockdown ACLs for `netguard.db`, `experience.db`, and binaries. |
| **Training & Red‑Team Lab** | `netguard/training_scheduler.py`, `netgrid/incident.py` | Generates realistic network/hardware‑abuse scenarios using RFC 5737 IPs and a dry‑run firewall; grades live LLM verdicts; feeds mistakes into an `ExperienceStore` for immediate RAG learning. |
| **Throughput & Telemetry** | `netguard/throughput.py` | Exposes `python -m netguard.smoke_throughput`; exercises real host performance counters; exits 0 only when global & per‑process deltas are finite, non‑sentinel values. |
| **Forensic Exporter** | `netgrid/incident.py` | Produces incident bundles (`netguard.incident`) for audit/forensics; unit tests cover export format and content. |
| **Multi‑Theme Engine** | `netguard/theme.py` | UI theme support with dedicated unit tests (`netguard.theme`); supports light/dark/custom colour schemes. |
| **Launch & Autostart Scripts** | `launch_netguard_sentinel.bat`, `start_netguard.ps1` | Handle elevation, reboot persistence, and “Restart‑as‑Admin” flows for Windows‑tray operation. |

## Data Flow
1. **Startup** → `__main__.py` loads config, initiates `connection_monitor` (admin‑elevated if blocking needed).
2. **Monitoring** → Every connection event passes through `firewall_engine`, which checks rules from `database`.
3. **Alerting** → If a rule matches, `sentinel_ai` may generate an LLM verdict; event logged to `block_events` table.
4. **Tamper Protection** → Background thread validates integrity of `netguard.db`, runtime files, and ACLs; on violation, can auto‑remediate or raise alert.
5. **Diagnostics** → `--diagnostic N` flag snapshots the current state (`connection_monitor` stats, active rules, recent block events) to a JSON report.
6. **Training Lab** → `training_scheduler` generates synthetic traffic, runs `firewall_engine` in dry‑run mode, records LLM verdicts, updates `experience.db`.

## Configuration
- `.env` or DB settings control: `NETGUARD_LLM_BASE_URL`, `NETGUARD_LLM_MODEL`, `NETGUARD_LLM_API_KEY`, `NETGUARD_DB_PATH`, `NETGUARD_TAMPER_MODE`, `NETGUARD_AUTOSTART`.
- `.bat`/`.ps1` scripts accept flags for elevation and silent install.

## Usage Scenarios
- **Real‑time guardian** (`python -m netguard`) – system tray, automatic blocking.
- **Headless audit** (`python -m netguard --diagnostic 60`) – 60‑second snapshot + JSON report.
- **Throughput testing** (`python -m netguard.smoke_throughput`) – validate performance impact.
- **Red‑team training** – leverage the built‑in attack simulator to train your own LLM sentinel.

## Development Entry Points
- Add new rule types in `firewall_engine.py`.
- Extend `sentinel_ai` with additional LLM prompts or RAG sources.
- Implement custom monitors under `netguard/monitors/`.
- Contribute themes to `theme.py`.
