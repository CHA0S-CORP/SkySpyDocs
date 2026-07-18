---
title: Ownership & Shell-Risk Screening
hidden: false
---

# 🕵️ Ownership & Shell-Risk Screening

> Flag opaque aircraft ownership — shell LLCs, trust registrations, registered-agent addresses — and screen owner names against sanctions/PEP/watchlists.

---

## 📖 Overview

Many aircraft are registered through structures that obscure the beneficial owner: single-purpose LLCs, ownership trusts (Bank of Utah, Wilmington Trust), registered-agent addresses (CT Corp, CSC), or rapid chains of transfers. SkySpy v3 computes a **shell-company risk score** per airframe from FAA registry signals and optionally screens the owner name against **OpenSanctions** (sanctions, PEPs, watchlists).

Results attach to the airframe record (`shell_score`, `is_shell_suspected`, `owner_type`, `ownership_flags`), feed the [airframe dossier](./airframe-intelligence), and render on the aircraft detail screen with a risk badge.

> ⚖️ **Signals, not conclusions.** A high score means the ownership *structure* is opaque or matches known laundering patterns — not that the aircraft or owner is doing anything illegal. Treat it as a lead, not a verdict.

---

## 🧮 Shell-Company Risk Score

`analyze_registration()` combines weighted factors into a `shell_company_score` (0.0–1.0) and a `risk_level` (`high` ≥ 0.7, `medium` ≥ 0.4, else `low`).

| Factor | Weight | Signal |
|:-------|:------:|:-------|
| `sanctions_hit` | 0.75 | Owner matched a sanctions/PEP record (dominates scoring) |
| `registered_agent_address` | 0.25 | Address is a corporate agent (C/O CT Corp, CSC, …) |
| `multiple_transfers` | 0.20 | 3+ transfers in 3 years, or <90 days between transfers |
| `generic_llc_name` | 0.15 | Generic pattern like "ABC AVIATION LLC" |
| `trust_ownership` | 0.15 | Known trust registrant (Bank of Utah, Wells Fargo, Wilmington Trust) |
| `po_box_address` | 0.10 | P.O. Box / PMB / mailbox address |
| `llc_no_web_presence` | 0.15 | Reserved (currently inert) |

### FAA registry signals used

| FAA MASTER field | Mapped to | Use |
|:-----------------|:----------|:----|
| `TYPE-REGISTRANT` | `faa_registrant_type` | Authoritative owner classification (individual / corp / co-owned / LLC / gov …) |
| `STREET` + `STREET2` | `owner_address` | Registered-agent vs PO-box heuristics |
| `FRACT OWNER` | `fractional_owner` | Fractional-ownership flag (NetJets-style) |
| `TYPE AIRCRAFT` | `faa_is_rotorcraft` | Rotorcraft (codes 6/9) — a law-enforcement indicator |

---

## 🌐 OpenSanctions Screening

When enabled, owner names are matched against the hosted **OpenSanctions** API. A match with score ≥ 0.70 (or an explicit match flag) contributes the dominant `sanctions_hit` factor.

**Endpoint:** `POST {OPENSANCTIONS_API_URL}/match/{OPENSANCTIONS_DATASET}` with a `LegalEntity` query on the owner name. Hits/misses cache for 24h; network failures cache for 10 min.

```env
OPENSANCTIONS_ENABLED=True
OPENSANCTIONS_API_URL=https://api.opensanctions.org
OPENSANCTIONS_API_KEY=your-key          # free for non-commercial use
OPENSANCTIONS_DATASET=default
```

> 📘 Off by default and a no-op without an API key. The `details.sanctions` object (caption, score, topics, datasets) is preserved on the result for review.

---

## 📤 Output Shape

The enrichment writes a compact record onto the airframe (`AircraftInfo`) and a full reviewable row (`RegistrationAnalysis`):

```json
{
  "owner_type": "trust",
  "is_shell_suspected": true,
  "shell_score": 0.62,
  "ownership_flags": {
    "risk_level": "medium",
    "factors": {
      "registered_agent_address": 0.25,
      "trust_ownership": 0.15,
      "generic_llc_name": 0.15
    },
    "details": {
      "registered_agent_detected": true,
      "trust_ownership_detected": true,
      "sanctions": null
    },
    "fractional_owner": false
  }
}
```

`is_shell_suspected` is true when the aggregate score ≥ 0.5. The full `RegistrationAnalysis` model additionally supports manual review (`reviewed_at`, `is_confirmed_le`).

---

## 🖥️ In the Dashboard

![Airframe overview with ownership](https://raw.githubusercontent.com/CHA0S-CORP/SkySpy/main/docs/screenshots/desktop/airframe-overview.png)

The **aircraft detail** screen surfaces:

- A **shell-risk badge** with score (0–100%) and `risk_level`
- Owner type (LLC / Trust / Corporation / Government / Individual)
- The weighted evidence factors that fired
- A **Law Enforcement badge** when `ownership_flags.law_enforcement` is present (category + description tooltip)

These fields also appear on the Live Map detail panel's operator/owner badge and feed `lookup_airframe` / `semantic_airframe_search` in the [AI Assistant](./ai-assistant).

---

## ⚙️ Configuration

| Variable | Default | Description |
|:---------|:--------|:------------|
| `OPENSANCTIONS_ENABLED` | `False` | Enable owner-name screening |
| `OPENSANCTIONS_API_URL` | `https://api.opensanctions.org` | API base |
| `OPENSANCTIONS_API_KEY` | *(empty)* | API key — required for any match |
| `OPENSANCTIONS_DATASET` | `default` | Collection to match against |

> 💡 Shell-risk scoring from FAA signals runs even without OpenSanctions — the sanctions factor simply stays zero. Enable OpenSanctions to add the watchlist dimension.

---

## 📚 Related

- [Airframe Intelligence & RAG](./airframe-intelligence) — consumes ownership risk in dossiers
- [AI Assistant](./ai-assistant) — `lookup_airframe`, `semantic_airframe_search`
- [Cannonball Mode](./cannonball-mode) — law-enforcement / surveillance detection
