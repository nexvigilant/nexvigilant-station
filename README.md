# NexVigilant Station

**151 free pharmacovigilance MCP tools. No auth required.**

Connect any AI agent to real drug safety data: FAERS adverse events (20M+ reports), drug safety signals (PRR/ROR/IC/EBGM), causality assessment (Naranjo/WHO-UMC), clinical trials, PubMed literature, DailyMed labeling, and sub-microsecond PV decision trees.

## Connect

### MCP Endpoint (Streamable HTTP)

```
https://mcp.nexvigilant.com/mcp
```

No authentication. MCP 2025-03-26 Streamable HTTP protocol. Works with Claude.ai, Claude Code, Cursor, Windsurf, and any MCP-compatible client.

### Claude.ai

Settings > Connectors > Add Custom Connector:
- **URL:** `https://mcp.nexvigilant.com/mcp`
- **Auth:** None

### Claude Code

Add to `~/.claude.json`:

```json
{
  "mcpServers": {
    "nexvigilant-station": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.nexvigilant.com/mcp", "--transport", "streamable-http"]
    }
  }
}
```

> **Note:** Use `streamable-http` transport, not SSE. SSE is not supported on Cloud Run.

### HTTP REST (any framework)

```bash
# List all tools
curl https://mcp.nexvigilant.com/tools

# Call a tool
curl -X POST https://mcp.nexvigilant.com/tools/api_fda_gov_search_adverse_events \
  -H 'Content-Type: application/json' \
  -d '{"drug_name": "semaglutide", "reaction": "pancreatitis"}'
```

## Tools (151)

### Drug Safety Signal Detection
| Tool | Description |
|------|-------------|
| `api_fda_gov_search_adverse_events` | Search FAERS adverse event reports by drug, reaction, or outcome |
| `api_fda_gov_get_event_timeline` | Adverse event reporting trends over time |
| `api_fda_gov_get_drug_counts` | Total report counts for a drug |
| `api_fda_gov_get_event_outcomes` | Outcome distribution (death, hospitalization, etc.) |
| `open_vigil_fr_compute_disproportionality` | Compute PRR, ROR, IC for a drug-event pair from FAERS |
| `open_vigil_fr_get_top_reactions` | Top adverse reactions for a drug by disproportionality |
| `open_vigil_fr_get_top_drugs` | Top drugs associated with a specific adverse event |

### Signal Computation
| Tool | Description |
|------|-------------|
| `calculate_nexvigilant_com_compute_prr` | Proportional Reporting Ratio from 2x2 table |
| `calculate_nexvigilant_com_compute_ror` | Reporting Odds Ratio from 2x2 table |
| `calculate_nexvigilant_com_compute_ic` | Information Component (BCPNN/WHO-UMC) |
| `calculate_nexvigilant_com_compute_ebgm` | Empirical Bayesian Geometric Mean (FDA GPS) |
| `calculate_nexvigilant_com_compute_disproportionality_table` | All 4 measures at once |

### Causality Assessment
| Tool | Description |
|------|-------------|
| `calculate_nexvigilant_com_assess_naranjo_causality` | Naranjo ADR probability scale (0-13) |
| `calculate_nexvigilant_com_assess_who_umc_causality` | WHO-UMC causality categories |
| `calculate_nexvigilant_com_classify_seriousness` | ICH E2A seriousness classification |

### Decision Trees (Micrograms)
| Tool | Description |
|------|-------------|
| `microgram_nexvigilant_com_run_pv_signal_to_action` | Full PV chain: signal -> causality -> regulatory action |
| `microgram_nexvigilant_com_run_prr_signal` | PRR threshold classification |
| `microgram_nexvigilant_com_run_naranjo_quick` | Fast Naranjo score routing |
| `microgram_nexvigilant_com_run_case_seriousness` | ICH E2A seriousness decision tree |
| `microgram_nexvigilant_com_list_micrograms` | Browse 460 available decision trees |
| `microgram_nexvigilant_com_list_chains` | Browse 34 chain workflows |

### Drug Information
| Tool | Description |
|------|-------------|
| `dailymed_nlm_nih_gov_get_adverse_reactions` | FDA-approved adverse reactions from drug label |
| `dailymed_nlm_nih_gov_get_boxed_warning` | Boxed warning text |
| `dailymed_nlm_nih_gov_get_drug_interactions` | Drug interaction information |
| `rxnav_nlm_nih_gov_get_rxcui` | Resolve drug name to RxNorm identifier |
| `rxnav_nlm_nih_gov_get_interactions` | Drug-drug interaction check |

### Literature & Evidence
| Tool | Description |
|------|-------------|
| `pubmed_ncbi_nlm_nih_gov_search_articles` | Search PubMed by query |
| `pubmed_ncbi_nlm_nih_gov_search_case_reports` | Find adverse event case reports |
| `pubmed_ncbi_nlm_nih_gov_search_signal_literature` | PV signal literature search |
| `clinicaltrials_gov_search_trials` | Search clinical trials |
| `clinicaltrials_gov_get_serious_adverse_events` | Trial safety data |

### Regulatory Intelligence
| Tool | Description |
|------|-------------|
| `ich_org_get_guideline` | ICH guideline lookup |
| `ich_org_get_pv_guidelines` | PV-specific ICH guidelines |
| `www_ema_europa_eu_get_safety_signals` | EMA safety signal assessments |
| `www_fda_gov_get_safety_labeling_changes` | FDA labeling changes |

[Full tool list: `curl https://mcp.nexvigilant.com/tools`]

## Transports

| Transport | Endpoint | Protocol |
|-----------|----------|----------|
| Streamable HTTP | `POST /mcp` | MCP 2025-03-26 |
| SSE | `GET /sse`, `POST /message` | MCP SSE |
| HTTP REST | `GET /tools`, `POST /tools/{name}` | REST |
| Health | `GET /health` | JSON |

## Example: Drug Safety Signal Investigation

```bash
# 1. Resolve drug identity
curl -X POST https://mcp.nexvigilant.com/tools/rxnav_nlm_nih_gov_get_rxcui \
  -H 'Content-Type: application/json' -d '{"drug_name": "semaglutide"}'

# 2. Search FAERS for adverse events
curl -X POST https://mcp.nexvigilant.com/tools/api_fda_gov_search_adverse_events \
  -H 'Content-Type: application/json' -d '{"drug_name": "semaglutide", "reaction": "pancreatitis"}'

# 3. Compute disproportionality signal
curl -X POST https://mcp.nexvigilant.com/tools/open_vigil_fr_compute_disproportionality \
  -H 'Content-Type: application/json' -d '{"drug": "semaglutide", "event": "pancreatitis"}'

# 4. Run full decision tree chain
curl -X POST https://mcp.nexvigilant.com/tools/microgram_nexvigilant_com_run_pv_signal_to_action \
  -H 'Content-Type: application/json' -d '{"prr": 6.93, "naranjo_score": 4, "is_serious": true}'
```

## Live Demo

See a complete signal investigation with 4 drug-event pairs, 8-year convergence analysis, and the conservation law frame:

[nexvigilant.com/station/demo](https://nexvigilant.com/station/demo)

## License

NexVigilant Source Available License v1.0. Personal non-commercial use permitted. Organizational use requires written permission from matthew@nexvigilant.com.
