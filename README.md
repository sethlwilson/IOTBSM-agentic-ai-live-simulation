# IOTBSM — Agentic Trust Network (Browser Simulation)

![IOTBSM simulation: trust network canvas with organizations, agents, boundary spanners, metrics sidebar, and controls](docs/iotbsm-simulation-preview.png)

Single-file, client-side simulation of ideas from the **Inter-organizational Trust-based Security Model** described in:

- Hexmoor, H., Wilson, S., & Bhattaram, S. (2006). [*A theoretical inter-organizational trust-based security model*](https://doi.org/10.1017/S0269888906000932). *The Knowledge Engineering Review*, 21(2), 127–161.

The paper develops **soft security** for open, distributed information-sharing settings: participants use **trust** (social control) rather than only “hard” barriers such as centralized authentication. It links **inter-personal** trust (including **boundary spanners**—organizational representatives or gatekeepers) with **inter-organizational** trust, and gives a calculus for trust dynamics and policies. Core outcome metrics in that line of work include **information availability (IA)** and **security measure (SM)**—roughly, how well needed information reaches legitimate parties versus how often security is breached (e.g., unintended disclosure).

This repository is a **pedagogical, interactive visualization**: it is **not** a faithful reimplementation of every equation or algorithm in the article, but it **implements the same conceptual structure** (multi-organization network, intra-org trust edges, BS-mediated cross-org requests, inter-org trust evolution, TPM-style trust updates on failure) and adds an **agentic “per-request claim”** layer so each information request is evaluated along a path: agent → own boundary spanner → peer boundary spanner → target agent.

## Quick start

1. Clone or download this repository.
2. Open `iotbsm_simulation.html` in a modern desktop browser (Chrome, Firefox, Safari, or Edge).

No build step or server is required. If your environment blocks `file://` scripts or fonts, serve the folder locally, for example:

```bash
python3 -m http.server 8080
```

Then open `http://localhost:8080/iotbsm_simulation.html`.

## What you will see

- **Organizations** as colored rings, each with several **agents** (dots) and a subset of **boundary spanners (BS)** (diamond nodes).
- **Intra-organizational** trust as faint green edges between agents in the same org.
- **Inter-organizational** trust as cyan dashed edges between org centers (strength encodes trust).
- **BS–BS** links in orange for cross-boundary channels.
- Animated **pulses** along grant/deny paths when requests are evaluated.

Hover agents and organizations for tooltips (trust averages, fact requirement, BS role, etc.). The **event log** records joins, elections, grants, denials, and breaches.

## Simulation loop (high level)

Each **cycle**:

1. On a schedule controlled by **β** (BS election period), **boundary spanners** are re-elected from intra-org **reliability** (direct and simplified indirect trust).
2. Non-BS agents **generate facts** (from a small discrete “warehouse”); old facts **expire** after a fixed interval.
3. Random **information requests** fire: an unsatisfied agent seeks a fact from another organization via a **claim** (requesting org, target org, fact id).
4. A **claim** is accepted or rejected along: **agent → BS (trust threshold) → BS–BS (blended inter-org and interpersonal trust, weight α) → target**; some outcomes count as **breaches** when policy treats the exchange as insecure.
5. **Inter-organizational trust** is updated using a **logistic growth** form (comments in code refer to the paper’s development of trust over interactions); **interaction counts** drive that growth.
6. The network may **grow or shrink** (random org join/leave after warm-up) to illustrate scalability dynamics.

## Metrics (this implementation)

| Metric | Meaning in the UI |
|--------|-------------------|
| **Information availability (IA)** | Percentage of **successful shares** that were **intended** (legitimate flow under the simulated policy), relative to all shares. |
| **Security measure (SM)** | **Breaches** as a percentage of all shares—proxy for unintended or policy-violating disclosure in this toy model. |
| **Cycle** | Simulation tick; subtitle shows cumulative **requests**. |
| **Network** | Counts of organizations, non-BS agents, and boundary spanners. |

Sparklines show recent history of IA and SM. Definitions are aligned with the **spirit** of the paper’s IA/SM (availability vs. breach pressure), simplified for the browser demo.

## Controls and parameters

**Header**

- **PAUSE / RESUME**, **STEP** — timing control.
- **+ ORG / − ORG** — add or remove the newest organization.
- **FIRE REQUEST** — trigger one manual cross-org request.
- **TPM-1 / 2 / 3** — **trust policy model** variants for how trust decays along a failed path (proportional, uniform, initiator-centric).
- **RESET** — new random initial network (four organizations by default).

**Simulation controls (sidebar)**

| Control | Role |
|---------|------|
| **Speed** | Milliseconds between automatic cycles. |
| **Trust thresh** | Minimum trust for several gate checks (agent→BS, BS–BS, etc.). |
| **Trust decay** | Magnitude of trust reduction when TPM fires on denied/breached claims. |
| **BS Alpha (α)** | Blend of **inter-organizational** vs. **prior BS–BS** trust when instantiating cross-boundary trust (see code comment referencing the paper’s BS trust construction). |
| **BS Rate (β)** | Cycles between **boundary spanner re-elections**. |

Fixed defaults in code also include the **fraction of agents elected as BS** and **fact expiration** (`CFG` object in `iotbsm_simulation.html`).

## Files

| File | Description |
|------|-------------|
| `iotbsm_simulation.html` | Styles, UI, canvas rendering, and the full simulation engine (plain JavaScript, no dependencies beyond Google Fonts). |

## Disclaimer

This is an **educational simulation**. Numeric behavior is chosen for clarity and stability in the browser, not for empirical calibration of real organizations or production security decisions.
