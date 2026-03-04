---
title: Hyper Tracker
description: AI-powered security task tracker for vulnerability management, threat intelligence, and incident response
version: 1.0.0
author: Security Operations Team
tags: [security, ai, automation]
dependencies:
  - openclaw-cli>=2.0
  - python>=3.9
  - sqlite3
  - numpy>=1.24.0
  - scikit-learn>=1.3.0
  - transformers>=4.30.0
  - torch>=2.0.0
  - fastapi>=0.100.0
  - uvicorn[standard]>=0.22.0
  - pydantic>=2.0.0
  - jira>=3.5.0
  - requests>=2.30.0
  - python-slugify>=8.0.0
---

# Hyper Tracker

AI-enhanced security tracking system that automatically ingests, categorizes, prioritizes, and tracks security-related tasks, events, and incidents with integrated threat intelligence and predictive analytics.

## Purpose

Hyper Tracker serves as the central nervous system for security operations:

1. **Vulnerability Management**: Auto-categorize CVEs, assign risk scores using AI analysis of exploit availability, CVSS metrics EPSS scores, and asset criticality. Example: CVE-2024-3094 gets risk score 92/100 because AI detected weaponized exploit in dark web forums 3 days before public disclosure.

2. **Threat Intelligence Tracking**: Track IOCs, TTPs, and adversary behaviors with automated enrichment from VirusTotal, AlienVault OTX, and commercial feeds. Auto-correlates new IOCs with existing incidents using embedding similarity.

3. **Incident Response Coordination**: Timeline-based IR tracking with automated task assignment, evidence chain preservation, and SLA monitoring. Example: When a ransomware alert fires, Hyper Tracker automatically creates incident, assigns to tier-2 team, and starts 1-hour containment SLA countdown.

4. **Compliance Mapping**: Auto-map events to controls (NIST 800-53, ISO 27001, PCI-DSS). Example: Every "access revoked" event automatically tagged as PCI-DSS 8.1.4 evidence with retention date.

5. **Predictive Analytics**: Use LSTM on historical incident data to predict:
   - Which unpatched vulns will be exploited next (95% precision for critical CVEs)
   - Likely attacker next moves based on MITRE ATT&CK progression
   - Analyst burnout risk based on workload patterns

6. **Automated Ticketing**: Bidirectional sync with JIRA, ServiceNow, GitHub Issues. Comments in ticket update Hyper Tracker; analysis updates push to ticket via webhook.

Real-world use case:
> SOC analyst receives 500 daily events. Hyper Tracker reduces to 15 high-risk items by AI triage. Analyst investigates incident INC-2024-0892 (suspected credential stuffing). Hyper Tracker shows:
> - Related IOCs: 47 IPs match "Proxy Bot" campaign (confidence 94%)
> - Predictive: 87% chance attacker will pivot to SSO next
> - Recommended response: block IPs at WAF, enable MFA for 3 high-risk users, review SSO logs
> - Auto-created JIRA ticket with all evidence attached
> - SLA timer started: containment due in 2 hours (current: 23 minutes elapsed)

## Scope

Hyper Tracker provides production-grade commands for security teams:

### Core Event Operations

```
hyper-tracker add \
  --type <vulnerability|incident|ioc|compliance|task|threat> \
  --source <nmap|qualys|nessus|openvas|siem|manual|crowdstrike|sentinel|guardduty|custom> \
  --severity <critical|high|medium|low|info|unknown> \
  [--confidence 0.0-1.0] \
  --title "Brief description (max 120 chars)" \
  --details "Extended description with technical specifics" \
  [--cve CVE-2024-XXXX] \
  [--cwe CWE-XXXX] \
  [--cvss CVSS_VECTOR] \
  [--epss-score 0.0-1.0] \
  [--asset-ids ID1,ID2,...] \
  [--iocs IOC1,IOC2,...] \
  [--mitre-attack TXXXX.XXX] \
  [--due-date YYYY-MM-DD] \
  [--assignee email@domain.com] \
  [--tags tag1,tag2,...] \
  [--attachment /path/to/file]
```

```
hyper-tracker search \
  [--query "full-text search"] \
  [--type TYPE] \
  [--severity SEVERITY] \
  [--status <open|investigating|contained|resolved|closed|rejected>] \
  [--assignee EMAIL] \
  [--asset-id ID] \
  [--ioc VALUE] \
  [--cve CVE-2024-XXXX] \
  [--mitre ATTACK_ID] \
  [--date-start YYYY-MM-DD] \
  [--date-end YYYY-MM-DD] \
  [--priority <P0|P1|P2|P3>] \
  [--risk-score-min N] \
  [--ai-confidence-min 0.0-1.0] \
  [--limit N] \
  [--offset N] \
  [--sort <created|severity|risk|due_date>] \
  [--format <json|table|csv|jira>]
```

Examples:
```bash
# Find critical vulns on production assets with available exploit
hyper-tracker search \
  --type vulnerability \
  --severity critical \
  --asset-tag environment:production \
  --ai-confidence-min 0.8 \
  --format table

# Find incidents related to specific IOC
hyper-tracker search \
  --ioc "185.220.101.77" \
  --status "!resolved" \
  --sort created \
  --limit 20

# Export all open PCI-relevant events
hyper-tracker search \
  --tag pci-dss \
  --status "!closed" \
  --format csv \
  --output /tmp/pci-open-events.csv
```

```
hyper-tracker update \
  <event-id> \
  [--status STATUS] \
  [--severity SEVERITY] \
  [--priority P0|P1|P2|P3] \
  [--assignee EMAIL] \
  [--add-comment "Comment text (markdown supported)"] \
  [--add-evidence /path/to/file] \
  [--add-ioc IOC_VALUE] \
  [--remove-ioc IOC_VALUE] \
  [--add-tag TAG] \
  [--remove-tag TAG] \
  [--resolution "How it was resolved"] \
  [--resolution-by EMAIL] \
  [--resolution-date YYYY-MM-DD]
```

```
hyper-tracker show \
  <event-id> \
  [--full] \
  [--timeline] \
  [--ai-analysis] \
  [--evidence] \
  [--format <json|table>]
```

### AI & Enrichment

```
hyper-tracker analyze \
  <event-id> \
  [--re-run] \
  [--model <vulnerability|incident|ioc|ttp|all>] \
  [--wait] \
  [--timeout SECONDS]
```

Triggers async AI enrichment. With `--wait`, blocks until complete and shows results.

```
hyper-tracker enrich-ioc \
  <ioc-value> \
  [--sources <all|virusTotal|alienvault|otx|custom>] \
  [--force]
```

```
hyper-tracker predict \
  <event-id> \
  [--scenario <exploitation|lateral-movement|data-exfiltration>]
```

Uses AI to predict likely attack progression. Output includes probability, timeline, recommended defenses.

```
hyper-tracker recommend \
  <event-id> \
  [--type <containment|remediation|investigation>] \
  [--limit N]
```

Generates step-by-step playbook recommendations.

### Alerting & Automation

```
hyper-tracker alert add \
  --name "Alert name" \
  --condition 'json-logic expression' \
  --channel <slack|email|pagerduty|teams|webhook|splunk> \
  --recipient TARGET \
  [--template TEMPLATE_NAME] \
  [--throttle < per_minute|per_hour|daily >] \
  [--cooldown MINUTES] \
  [--severity-threshold <critical|high|medium>]
```

Condition examples:
```json
--condition '{"and": [{"==": [{"var": "type"}, "vulnerability"]}, {">": [{"var": "risk_score"}, 80]}, {"==": [{"var": "asset.environment"}, "production"]}]}'
```

```
hyper-tracker webhook add \
  <name> \
  --url https://service/hook \
  --secret SHARED_SECRET \
  --events <incident.created|vulnerability.updated|ioc.matched|alert.fired> \
  [--format json] \
  [--retry N] \
  [--timeout SECONDS]
```

```
hyper-tracker rule create \
  --name "Auto-assign SSH vulns" \
  --trigger "vulnerability.created && cve && 'CVE-2024-XXXX' in tags" \
  --action "assign = 'ssh-team@company.com'; add_tag = 'ssh-team-pending'"
```

```
hyper-tracker schedule add \
  --name "Daily threat briefing" \
  --cron "0 8 * * *" \
  --command 'report daily-briefing --format html --email security-team@company.com'
```

### Reporting & Export

```
hyper-tracker report \
  <report-type> \
  [--output FILE] \
  [--format <html|pdf|json|csv|jsonl|markdown>] \
  [--date-range <LAST_7_DAYS|LAST_30_DAYS|CUSTOM>] \
  [--date-start YYYY-MM-DD] \
  [--date-end YYYY-MM-DD] \
  [--filters FILTER_JSON] \
  [--template TEMPLATE_NAME] \
  [--email RECIPIENTS]
```

Report types:
- `daily-briefing`: SOC morning digest with overnight events
- `vulnerability-trends`: Charts of new vs fixed vulns by severity
- `incident-summary`: All open incidents with SLA status
- `compliance-gap`: Mapping to frameworks with gaps
- `mttr-mttd`: Mean time to respond/detect with trends
- `threat-landscape`: Top IOCs, campaigns, TTPs
- `asset-risk-heatmap`: Asset inventory colored by risk
- `ai-performance`: Model accuracy, false positive rates

```
hyper-tracker export \
  [--table <events|assets|iocs|incidents|alerts|audit>] \
  [--output FILE] \
  [--format <json|csv|parquet|feather|sqlite>] \
  [--compression <gzip|bzip2|zstd>] \
  [--filters FILTER_JSON] \
  [--fields field1,field2,...] \
  [--partition-by date]
```

```
hyper-tracker import \
  --source FILE \
  --format <json|csv|nessus|qualys|jira|servicenow|csv> \
  [--map-fields CONFIG_JSON] \
  [--dry-run] \
  [--skip-duplicates] \
  [--update-existing] \
  [--batch-size N]
```

### Asset & IOC Management

```
hyper-tracker asset add \
  --asset-id UNIQUE_ID \
  --hostname FQDN \
  --ip IP_ADDRESS \
  [--mac MAC] \
  [--os OS_NAME] \
  [--criticality <low|medium|high|critical>] \
  [--environment <dev|staging|production|lab>] \
  [--owner EMAIL] \
  [--team TEAM_NAME] \
  [--tags tag1,tag2,...] \
  [--cmdb-id EXTERNAL_ID]
```

```
hyper-tracker asset tag \
  <asset-id> \
  add TAG[:VALUE] \
  [--expire DATE]
```

```
hyper-tracker ioc add \
  --value IOC_VALUE \
  --type <ip|domain|url|hash|email|user-agent|custom> \
  [--source source_name] \
  [--confidence 0.0-1.0] \
  [--threat-type <malware|c2|phishing|botnet|apt>] \
  [--campaign CAMPAIGN_NAME] \
  [--first-seen DATE] \
  [--last-seen DATE] \
  [--feed-id EXTERNAL_ID]
```

```
hyper-tracker ioc check \
  <value> \
  [--sources all] \
  [--format full]
```

### System Administration

```
hyper-tracker init \
  [--config CONFIG_FILE] \
  [--db-path PATH] \
  [--ai-model-dir PATH] \
  [--log-level <DEBUG|INFO|WARNING|ERROR>] \
  [--skip-samples] \
  [--force]
```

```
hyper-tracker config \
  set <key> <value> \
  [--scope <global|ai|alerts|integrations|database|dashboard>] \
  [--file CONFIG_FILE]

hyper-tracker config \
  get <key> \
  [--scope SCOPE]

hyper-tracker config \
  list \
  [--scope SCOPE] \
  [--show-secrets]
```

```
hyper-tracker status
```

Shows comprehensive system health:
```
System Status: HEALTHY
Database: /var/lib/hyper-tracker/data.db (12.4GB, 45,231 events)
AI Models: vulnerability (v2.1, loaded ✓), incident (v1.4, loaded ✓)
Last ingestion: 2 minutes ago (Qualys scan)
Queue depth: events: 15, alerts: 3
Active webhooks: 4 (all healthy)
Dashboard: http://localhost:8080 (auth: enabled)
Disk usage: 45% (75GB free of 137GB)
Last backup: 2024-03-15 02:00 UTC (success)
```

```
hyper-tracker model list \
  [--show-accuracy] \
  [--show-training-data] \
  [--format table]

hyper-tracker model download \
  <model-name> \
  --source <huggingface|s3|local|url> \
  --model-id HF_MODEL_ID \
  [--api-token TOKEN] \
  [--cache-dir PATH]

hyper-tracker model train \
  <model-name> \
  [--dataset PATH] \
  [--epochs N] \
  [--validation-split 0.0-1.0] \
  [--learning-rate 0.0001] \
  [--batch-size N] \
  [--backup-before] \
  [--dry-run]

hyper-tracker model evaluate \
  <model-name> \
  [--test-set PATH] \
  [--metrics precision|recall|f1|accuracy|all]
```

```
hyper-tracker backup \
  [--output FILE] \
  [--encrypt] \
  [--compress] \
  [--include-logs] \
  [--include-models]

hyper-tracker restore \
  --backup FILE \
  [--dry-run] \
  [--merge-strategy <overwrite|skip|merge|prompt>] \
  [--skip-models] \
  [--verify]
```

```
hyper-tracker audit \
  [--event-id ID] \
  [--user EMAIL] \
  [--action <create|update|delete|login|config_change>] \
  [--date-start DATE] \
  [--date-end DATE] \
  [--format json|table]
```

Shows immutable audit trail of all system changes.

```
hyper-tracker healthcheck
```
Exit code 0 if healthy, non-zero if issues. Suitable for monitoring systems.

```
hyper-tracker maintenance \
  vacuum \
  [--db-path DB] \
  [--aggressive]

hyper-tracker maintenance \
  archive \
  [--older-than DAYS] \
  [--dry-run] \
  [--verify]

hyper-tracker maintenance \
  reindex \
  [--table events] \
  [--concurrently]
```

## Detailed Work Process

### 1. Initialization Flow

```bash
hyper-tracker init --config ./hyper-tracker.yaml --db-path /var/lib/hyper-tracker/data.db
```

`hyper-tracker.yaml` example:
```yaml
database:
  type: sqlite
  path: /var/lib/hyper-tracker/data.db
  wal_mode: true
  journal_mode: WAL
  cache_size: 64000  # 64MB

ai:
  models_dir: ./models
  vulnerability_model: microsoft/cvebert-base-uncased
  incident_classifier: custom-trained-incident-v3
  ioc_enrichment: hybrid-virustotal-alienvault
  enable_recommendations: true
  confidence_threshold: 0.6

integrations:
  jira:
    url: https://jira.company.com
    project: SEC
    auto_create: true
    update_on_ai: true
    mapping:
      severity: priority
      risk_score: customfield_risk_score
  
  slack:
    webhook: https://hooks.slack.com/services/...
    channel: "#security-alerts"
    mention_on: ["critical", "P0"]
  
  threat_intel:
    vt_api_key: ${VIRUSTOTAL_API_KEY}
    otx_api_key: ${OTX_API_KEY}
    feeds:
      - https://api.threatintelplatform.com/v1/iocs
      - https://raw.githubusercontent.com/.../威胁情报.csv

ingest:
  sources:
    qualys:
      schedule: "0 */6 * * *"
      api_url: https://qualysapi.qualys.com
      api_token: ${QUALYS_TOKEN}
      tag_new: qualys
      auto_create_ticket: true
    
    siem:
      webhook_path: /webhook/siem
      auth_token: ${SIEM_WEBHOOK_SECRET}
      format: json-array
  
  batch_size: 100
  max_workers: 10

alerts:
  rate_limit:
    critical: 5/hour
    high: 20/hour
    medium: 50/hour
  
  throttle_default: 15/min
  cooldown_seconds: 300

retention:
  raw_events_days: 365
  audit_log_days: 2555  # 7 years
  logs_days: 90

dashboard:
  enabled: true
  host: 0.0.0.0
  port: 8080
  ssl: false
  auth:
    method: basic
    users_file: /etc/hyper-tracker/users

logging:
  level: INFO
  format: json
  max_size_mb: 100
  backup_count: 10
```

Init process creates:
```
/home/user/.config/hyper-tracker/config.yaml
/var/lib/hyper-tracker/data.db (SQLite with WAL)
/var/lib/hyper-tracker/models/
├── vulnerability/
│   ├── config.json
│   ├── pytorch_model.bin
│   └── tokenizer/
├── incident/
│   └── ...
/var/log/hyper-tracker/
├── hyper-tracker.log (JSON lines)
├── ingest.log
├── ai.log
└── audit.log
/var/lib/hyper-tracker/backups/
```

### 2. Ingestion Pipeline Deep Dive

**Qualys integration**:
```bash
# Runs every 6 hours via cron
hyper-tracker ingest run --source qualys --since-last
```

Process:
1. API call to Qualys: `GET /api/2.0/fo/vuln/`
2. Parse 10,000+ vulns per scan
3. For each vulnerability:
   - Deduplicate: hashed title + CVE + asset ID
   - Asset lookup: map QID hostname to internal asset-id via CMDB sync table
   - AI risk scoring: load vulnerability model, encode CVE description + asset context, predict exploit probability
   - Generate risk score: `base_severity * 0.3 + ai_exploit_score * 0.5 + asset_criticality * 0.2`
   - Create or update event
4. Batch insert 1000 events per DB transaction
5. Trigger alerts for risk_score > 80
6. Auto-create JIRA tickets if configured
7. Log ingestion metrics: `{ "source": "qualys", "total": 12453, "new": 2341, "updated": 10112, "duration_sec": 342 }`

**SIEM webhook** (real-time):
```bash
curl -X POST http://localhost:8080/webhook/siem \
  -H "Authorization: Bearer ${SIEM_SECRET}" \
  -H "Content-Type: application/json" \
  -d '[
    {
      "timestamp": "2024-03-15T14:23:00Z",
      "event_type": "authentication_failure",
      "source_ip": "185.220.101.77",
      "user": "admin",
      "asset": "vpn-gateway-01",
      "count": 15,
      "rule": "SSH Brute Force"
    }
  ]'
```

Processing:
1. Verify token
2. Parse JSON array (up to 1000 events per request)
3. Each event:
   - Type: map SIEM rule to Hyper Tracker type
   - IOC extraction: "source_ip" → add to IOC table if not exists
   - Asset normalization: "asset" → asset-id lookup (fuzzy match)
   - AI analysis: if authentication_failure > 10/min same IP+user → auto-create incident
   - Correlation: check if IP matches existing IOC → link event to incident
4. Response: `{"processed": 45, "created_incidents": 2, "alerts_fired": 1}`

### 3. AI Enrichment Deep Dive

**Vulnerability model** (CVE BERT fine-tuned):
```python
# Simplified architecture
input: [CLS] + CVE description tokens + [SEP] + asset criticality + environment + exposure level
output: 
  - exploit_probability (sigmoid)
  - weaponized (sigmoid > 0.7)
  - adjusted_severity (softmax over 5 classes)
  - recommended_actions (sequence generation with BART head)

Training data: 50,000 CVEs with ground truth:
  - Did NVD include exploit reference?
  - Was there a Metasploit module within 30 days?
  - Did customers get exploited (telemetry from EDR)?
  - What was actual patch compliance rate?
```

Example AI analysis output:
```json
{
  "event_id": "evt_5f3a1b9c",
  "ai_analysis": {
    "model": "vulnerability-v2.1",
    "model_version": "2024-03-10",
    "confidence": 0.87,
    "inference_time_ms": 142,
    "inputs": {
      "cve_description": "OpenSSH through 8.7 allows ...",
      "asset_criticality": "medium",
      "environment": "DMZ",
      "exposure": "internet-facing"
    },
    "outputs": {
      "adjusted_severity": "high",
      "exploit_prediction": {
        "probability": 0.89,
        "public_exploit_exists": true,
        "weaponized": true,
        "expected_exploit_timeframe": "0-7 days"
      },
      "recommendations": [
        "Apply patch within 72 hours (weaponized exploit observed)",
        "Monitor for suspicious authentication patterns",
        "Implement network segmentation for affected systems"
      ],
      "related_cves": [
        {"cve": "CVE-2024-XXXX", "similarity": 0.87},
        {"cve": "CVE-2023-YYYY", "similarity": 0.82}
      ],
      "mitre_ttp": "T1190"
    }
  }
}
```

**Incident classification model**:
- Input: incident title + description + IOCs + timeline
- Output: incident type (data_exfiltration, ransomware, account_takeover, malware, phishing, insider_threat, other)
- Also predicts likely attacker profile (script_kiddie, cybercrime, apt, insider)

**IOC enrichment**:
```bash
hyper-tracker enrich-ioc 185.220.101.77 --sources all
```

Output:
```json
{
  "ioc": "185.220.101.77",
  "type": "ip",
  "enrichments": {
    "virustotal": {
      "detection_ratio": "45/75",
      "last_analysis_date": "2024-03-15T10:23:00Z",
      "categories": ["c2", "malware"],
      "tags": ["cobalt-spider", "ransomware"]
    },
    "alienvault": {
      "pulse_count": 3,
      "last_pulse": "2024-03-14",
      "malware_families": ["Cobalt Strike"],
      "attack_techniques": ["T1132.001", "T1573"]
    },
    "greynoise": {
      "noise": true,
      "classification": "malicious",
      "last_seen": "2024-03-15T08:45:00Z",
      "tor": false
    }
  },
  "risk_score": 94,
  "campaign_association": {
    "campaign": "Cobalt Spider December 2023",
    "confidence": 0.92,
    "first_seen": "2023-12-01",
    "last_seen": "2024-01-15"
  }
}
```

### 4. Alerting Rules in Practice

**Rule 1: Critical prod vuln with weaponized exploit**
```bash
hyper-tracker alert add \
  --name "Critical prod vuln - weaponized" \
  --condition '{"and": [{"==": [{"var": "type"}, "vulnerability"]}, {">": [{"var": "risk_score"}, 85]}, {"==": [{"var": "asset.environment"}, "production"]}, {"==": [{"var": "ai_analysis.outputs.exploit_prediction.weaponized"}, true]}]}' \
  --channel slack \
  --recipient "#sec-critical" \
  --template "critical-vuln" \
  --throttle per_hour \
  --cooldown 30
```

Fires for CVE-2024-3094 on web-prod-01:
```
🚨 CRITICAL VULNERABILITY - WEAPONIZED
CVE: CVE-2024-3094
Asset: web-prod-01 (10.10.1.101) • Production • Criticality: HIGH
CVSS: 9.8 • AI Risk: 94/100 • Weaponized: YES
Description: Remote code execution in Apache Struts
Recommendation: IMMEDIATE PATCH REQUIRED - Exploit observed in wild
AI Confidence: 93%
Associated Incidents: 0 (new)
Created: 2024-03-15T14:32:00Z
[View in Dashboard](http://hyper-tracker:8080/events/evt_abc123)
[@here] @security-team
```

**Rule 2: New IOC matches campaign**
```bash
hyper-tracker alert add \
  --name "IOC Campaign Match" \
  --condition '{"and": [{"==": [{"var": "type"}, "ioc"]}, {">": [{"var": "campaign_association.confidence"}, 0.8]}]}' \
  --channel email \
  --recipient threat-intel@company.com \
  --template "ioc-campaign" \
  --throttle daily
```

Email template:
```
Subject: [Hyper Tracker] New IOC linked to campaign: {{campaign_association.campaign}}

New indicator detected: {{ioc_value}} ({{type}})
Campaign: {{campaign_association.campaign}} ({{campaign_association.confidence * 100}}% confidence)
First seen: {{first_seen}}
Threat type: {{threat_type}}
Enrichment: {{enrichments.virustotal.detection_ratio}} detections on VT
Action: Review related incidents and consider proactive blocking.

[View details]({{dashboard_url}})
```

### 5. Report Generation Examples

**Daily Briefing** (`hyper-tracker report daily-briefing`):

```
SECURITY DAILY BRIEFING - March 15, 2024

SUMMARY
--------
New events: 47 (↑12% vs yesterday)
Open incidents: 23 (↑3)
Critical vulns: 5 (↓2)
SLA compliance: 94.2% (P0: 100%, P1: 96%, P2: 91%)

TOP PRIORITY ITEMS
------------------
1. [P0] INC-2024-0892 - Ransomware activity on finance-db-01
   Created: 14:23 | Owner: jsmith@ | SLA: 37 min left
   AI Prediction: 87% chance of data exfiltration within 4h
   Action: Isolate host, preserve memory, notify legal

2. [P0] evt_abc123 - CVE-2024-3094 (Apache Struts) on web-prod-01
   Risk score: 94 | Exploit: weaponized | Asset: Production
   Action: Patch within 2h or take offline

3. [P1] INC-2024-0890 - Suspicious privileged access
   Created: 09:15 | Owner: mjones@ | SLA: 4h 45m left
   User: support-admin - Access from unusual location (Russia)
   Action: Interview user, check authentication logs

OVERNIGHT INCIDENTS
-------------------
• INC-2024-0885 - Brute force on SSH gateway (blocked by fail2ban)
  IOCs: 185.220.101.77, 185.220.101.78
  AI: Linked to "Cobalt Spider" campaign (94% confidence)
  Status: Resolved at 03:45

• INC-2024-0888 - False positive DLP alert (test file)
  Status: Closed as false positive

THREAT INTELLIGENCE
-------------------
New high-confidence IOCs added: 12
├─ 8 IPs (Cobalt Spider C2)
├─ 3 domains (phishing)
└─ 1 SHA256 (Emotet variant)

Campaign activity: "Cobalt Spider" remains active (32 incidents YTD)
Most targeted: Finance (12), Engineering (8), HR (5)

VULNERABILITY TRENDS
--------------------
New CVEs added: 34
High/New: 8 | Medium: 19 | Low: 7

Top 3 critical:
CVE-2024-3094 (Apache Struts) - 5 assets affected, risk: 94
CVE-2024-3004 (Fortinet) - 2 assets, risk: 91
CVE-2024-2987 (Microsoft) - 1 asset, risk: 88

Patch compliance (last 30 days):
Critical: 91% (↓3%) | High: 87% | Medium: 94%

UPCOMING
--------
• PCI-DSS external pen test due: April 15 (60 days)
• Quarterly access review starts Monday (800 users)
• System maintenance: March 22, 02:00-06:00 UTC

---
Dashboard: http://hyper-tracker:8080
Questions? #security-team
```

**Compliance Gap Report** (`hyper-tracker report compliance-gap --framework NIST-800-53`):

```
NIST 800-53 Rev. 5 Compliance Gap Analysis
As of: March 15, 2024
Period: Q1 2024 (Jan 1 - Mar 31)

OVERALL COMPLIANCE: 89/100

CONTROL FAMILY: ACCESS CONTROL (AC)
-----------------------------------
AC-2 (Account Management): COMPLIANT
  ✓ All user accounts inventoried
  ✓ Inactive accounts disabled (>90 days)
  ✓ Privileged accounts audited quarterly
  
AC-3 (Access Enforcement): MINOR GAP (87/100)
  ⚠ 3 roles with excessive privileges identified
  ⚠ No evidence of least privilege review for 2 applications
  
AC-6 (Least Privilege): GAP (72/100)
  ✗ 15 service accounts with admin rights (should be 0)
  ✗ No regular privilege escalation review process documented
  Action Items:
    1. Review all admin accounts by Mar 29
    2. Implement quarterly privilege review (due Apr 15)
    3. Implement PAM solution (project: Q2 2024)

CONTROL FAMILY: INCIDENT RESPONSE (IR)
--------------------------------------
IR-4 (Incident Handling): COMPLIANT (95/100)
  ✓ All security incidents tracked in Hyper Tracker
  ✓ Response playbooks exist for top 10 incident types
  ✓ Exercises conducted quarterly
  
IR-5 (Incident Monitoring): MINOR GAP (88/100)
  ⚠ 24/7 monitoring only for production (dev/staging gaps)
  Action: Extend monitoring coverage by Apr 30

CONTROL FAMILY: RISK ASSESSMENT (RA)
------------------------------------
RA-3 (Risk Assessment): COMPLIANT
  ✓ Annual risk assessment completed
  ✓ Risk register maintained in GRC tool
  ✓ Risk treatment plans tracked

COMPLIANCE TREND
----------------
Q4 2023: 86/100
Q1 2024: 89/100 ↑3 points
Projected Q2 2024: 91/100 (if open gaps addressed)

RISK-BASED PRIORITY (by impact × likelihood)
---------------------------------------------
1. AC-6 Least Privilege gap (Risk: 8.4/10) → ATTACK SURFACE: HIGH
2. IR-5 Monitoring gap (Risk: 6.2/10) → DETECTION GAP: HIGH
3. CA-7 Continuous Monitoring (Risk: 5.1/10) → MEDIUM

---
Full evidence package: /var/reports/compliance/nist80053-q1-2024/
Evidence collection automated via Hyper Tracker tags.
```

### 6. Predictive Analysis Example

**Predicting next attacker moves**:
```bash
hyper-tracker predict inc_9a2b3c4d --scenario lateral-movement
```

Output:
```json
{
  "incident_id": "inc_9a2b3c4d",
  "scenario": "lateral_movement",
  "predictions": [
    {
      "step": 1,
      "probability": 0.87,
      "technique": "T1078 - Valid Accounts",
      "target": "domain controllers",
      "timeline": "Within 24-48 hours",
      "trigger": " attacker already has domain admin creds from initial breach",
      "recommended_defenses": [
        "Enable PIM for all admin accounts (pending)",
        "Deploy LAPS on all workstations",
        "Block remote PowerShell from non-admin systems"
      ]
    },
    {
      "step": 2,
      "probability": 0.65,
      "technique": "T1055 - Process Injection",
      "target": "critical_sev_servers",
      "timeline": "48-72 hours after step 1",
      "trigger": "if credential theft successful",
      "recommended_defenses": [
        "Deploy process allowlisting on critical servers",
        "Enable Sysmon with process access monitoring"
      ]
    }
  ],
  "model_confidence": 0.76,
  "training_data": "45 similar incidents from 2023-2024",
  "last_updated": "2024-03-10"
}
```

### 7. Bulk Operations

**Import 5000 legacy events**:
```bash
hyper-tracker import \
  --source ./legacy-incidents-2023.csv \
  --format csv \
  --map-fields '{"type": "incident_type", "severity": "priority", "title": "subject", "details": "description", "create_date": "opened_at", "close_date": "resolved_at", "assignee": "owner"}' \
  --dry-run
```

Dry-run output:
```
DRY RUN - No changes will be made
Source: ./legacy-incidents-2023.csv (5,231 rows)
Mapping: 7/8 fields mapped successfully
├─ incidents: 5,231 (estimated)
├─ new assets discovered: 156
├─ duplicate detection: 342 events already exist (would skip)
└─ validation errors: 0

Sample mapping:
  legacy severity "P1" → Hyper Tracker severity "high"
  legacy status "Closed" → Hyper Tracker status "resolved"
  
Would create:
  - 4,889 incidents
  - 156 new assets
  - Skip 342 duplicates
Estimated duration: 8-12 minutes
```

Actual run with batch processing:
```bash
hyper-tracker import \
  --source ./legacy-incidents-2023.csv \
  --format csv \
  --map-fields ./mapping.json \
  --batch-size 500 \
  --skip-duplicates \
  --update-existing
```

Progress: `[====>                  ] 45% (2341/5231) - 12m elapsed, 15m remaining`

Completion:
```
✓ Import completed successfully
  • Events created: 4,889
  • Events updated: 342
  • New assets added: 156
  • Duplicates skipped: 342
  • Validation errors: 0
  • Duration: 14m 23s
  
Post-import: AI enrichment queued for 4,889 events
  (estimated completion: 2 hours)
  
Generated report: /tmp/import-summary-20240315.html
```

### 8. Dashboard Usage

Dashboard at `http://hyper-tracker:8080` (basic auth per user):

**Tab: Overview**
- KPI cards: Open incidents, critical vulns, MTTR, new events 24h
- Trend charts: events by severity (7-day), SLA compliance
- Heatmap: risk by asset group

**Tab: Events**
- Searchable table with all fields
- Inline edits: drag-and-drop assignee, status dropdown
- Bulk actions: change status, add tags, re-run AI analysis
- Export button (CSV/JSON)

**Tab: Assets**
- Tree view: environment → team → assets
- Asset details modal: vulns, incidents, IOCs, risk score trend
- Edit asset: criticality, owner, tags

**Tab: IOCs**
- Watchlist table: all known IOCs with last seen, threat type
- Match history: which events matched this IOC
- Enrichment panel: VT/OTX data inline

**Tab: Analytics**
- MTTR/MITTD by severity, asset type, team
- AI confusion matrix (actual vs predicted severity)
- False positive rate by analyst
- Top attack techniques (MITRE mapping)

**Tab: System**
- Queue depths, ingestion latency
- AI model performance
- Storage usage, backup status

## Golden Rules

1. **Never Auto-Close Without Verification**: Events can only be marked `resolved` with mandatory `resolution` field. AI may suggest closure but human must confirm. For vulnerabilities: require `patched_version` or `mitigation_applied` evidence. For incidents: require `postmortem_link` or `lessons_learned_attached`.

2. **Immutable Audit Trail**: All event updates append to `audit_log` - never UPDATE or DELETE. Show changes as diffs. Retain audit log 7 years for compliance. The only delete operation is `hyper-tracker rollback --event-id X --reason "duplicate"` within 5 minutes of creation, which logs deletion in audit.

3. **AI Confidence Thresholds**: Never act on AI recommendations with confidence < 0.7 without analyst review. Display confidence prominently in UI and CLI. High-stakes decisions (P0 incident response) require analyst override field if deviating from AI.

4. **Data Minimization for IOCs**: Do not store raw PII in IOC details. Hash email addresses, mask IPs in internal logs (show last /24 only). Encrypt database at rest if containing EU/PHI data.

5. **Rate Limiting & Throttling**: All external API calls (VirusTotal, JIRA) must respect rate limits. Hyper Tracker default: VT 4 req/min, JIRA 100 req/min. Implement exponential backoff. Monitored via `hyper-tracker status`.

6. **Model Version Pinning**: Never auto-upgrade AI models. Pin specific model versions in config:
   ```
   ai:
     models:
       vulnerability: microsoft/cvebert-1.0
       incident: custom-incident-v3.1
   ```
   Test new models in staging before production deployment.

7. **Evidence Chain Integrity**: All evidence files stored with SHA256 checksum recorded. Prevent tampering: evidence directory is append-only (chattr +a). File modifications logged to audit.

8. **SLA Enforcement**:
   - P0 incidents: acknowledge within 15m, contain within 1h, resolve within 4h
   - P1: acknowledge 1h, contain 4h, resolve 24h
   - P2: acknowledge 4h, contain 24h, resolve 7d
   SLA timers start at creation, pause only with `status: investigating` (requires comment explaining delay). SLA breaches trigger immediate alert to management.

9. **Segregation of Duties**: Hyper Tracker `admin` role cannot also be `analyst`. Admin manages system config, models, integrations. Analyst creates/updates events, runs AI analysis. Dual-role forbidden in production.

10. **Backup Before Destructive Operations**: Always run `hyper-tracker backup` before:
    - Bulk imports/updates
    - Model retraining (which may overwrite production model)
    - Database maintenance (VACUUM, ARCHIVE)
    - Config changes with `--scope global`

## Examples with Real Inputs/Outputs

### Example 1: Vulnerability ingestion and AI analysis

**Input** (manual addition of new CVE):
```bash
hyper-tracker add \
  --type vulnerability \
  --source manual \
  --severity unknown \
  --title "OpenSSH CVE-2024-3094 remote code execution" \
  --details "A vulnerability in OpenSSH's ssh-keygen allows writing beyond allocated buffer when processing certain certificate keys. Attackers can craft malicious keys to achieve RCE on systems that validate keys." \
  --cve CVE-2024-3094 \
  --cvss "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H" \
  --asset-ids web-prod-01,web-prod-02,web-staging-03 \
  --tags cve2024,openssh,remote-code-execution
```

**Output**:
```
✓ Event created: evt_f3a1b9c2
Assets: 3 found (2 production, 1 staging)
AI analysis queued (job ID: ai_9z8y7x6w)
Will notify when complete (typically 30-90 seconds)
Run: hyper-tracker show evt_f3a1b9c2 --ai-analysis to view results
```

**Later** (after AI processing):
```bash
hyper-tracker show evt_f3a1b9c2 --full
```

Output (key fields):
```json
{
  "event_id": "evt_f3a1b9c2",
  "type": "vulnerability",
  "cve": "CVE-2024-3094",
  "title": "OpenSSH CVE-2024-3094 remote code execution",
  "severity": "high",  // AI adjusted from 'unknown'
  "risk_score": 94,
  "priority": "P0",
  "assets": [
    {
      "asset_id": "web-prod-01",
      "hostname": "web-prod-01.company.com",
      "ip": "10.10.1.101",
      "criticality": "critical",
      "environment": "production",
      "exposure": "internet-facing"
    },
    {
      "asset_id": "web-prod-02",
      "hostname": "web-prod-02.company.com",
      "ip": "10.10.1.102",
      "criticality": "critical",
      "environment": "production"
    },
    {
      "asset_id": "web-staging-03",
      "hostname": "web-staging-03.company.com",
      "ip": "10.10.2.103",
      "criticality": "medium",
      "environment": "staging"
    }
  ],
  "ai_analysis": {
    "model": "vulnerability-v2.1",
    "confidence": 0.93,
    "inference_time_ms": 156,
    "outputs": {
      "adjusted_severity": "critical",  // AI confidence: 0.93
      "exploit_prediction": {
        "probability": 0.96,
        "public_exploit_exists": true,
        "weaponized": true,
        "expected_exploit_timeframe": "0-3 days (active exploitation observed on dark web forums)"
      },
      "recommendations": [
        "IMMEDIATE: Patch all OpenSSH instances to 9.4p1 or later. This vulnerability is being actively exploited.",
        "Consider temporary mitigation: disable ssh-keygen certificate validation if not required (not recommended for production)",
        "Monitor for suspicious authentication patterns",
        "Review logs for evidence of exploitation attempts"
      ],
      "related_cves": [
        {"cve": "CVE-2023-38408", "similarity": 0.89, "notes": "Similar OpenSSH parsing vuln"},
        {"cve": "CVE-2024-22001", "similarity": 0.85, "notes": "Recent SSH vuln, different component"}
      ],
      "mitre_ttp": "T1190"
    }
  },
  "alerts_fired": [
    {
      "alert_name": "Critical prod vuln - weaponized",
      "channel": "slack",
      "timestamp": "2024-03-15T14:35:12Z",
      "recipient": "#sec-critical"
    }
  ],
  "ticket_created": {
    "system": "jira",
    "ticket_id": "SEC-12456",
    "url": "https://jira.company.com/browse/SEC-12456"
  },
  "created_at": "2024-03-15T14:32:00Z",
  "updated_at": "2024-03-15T14:35:15Z",
  "audit_log": [
    {"timestamp": "2024-03-15T14:32:00Z", "user": "analyst@company.com", "action": "create", "details": "Event created from CLI"},
    {"timestamp": "2024-03-15T14:35:12Z", "user": "system", "action": "ai_enrichment", "details": "AI analysis completed, risk_score=94"},
    {"timestamp": "2024-03-15T14:35:13Z", "user": "system", "action": "alert_fired", "details": "Alert 'Critical prod vuln - weaponized' to slack"},
    {"timestamp": "2024-03-15T14:35:15Z", "user": "system", "action": "ticket_created", "details": "JIRA SEC-12456 created"}
  ]
}
```

**JIRA ticket created automatically**:
```
Project: SEC
Issue Type: Vulnerability
Priority: Highest
Summary: [P0] CVE-2024-3094 - Remote Code Execution in OpenSSH on web-prod-01, web-prod-02 (Production Web Servers)
Environment: Production
Affected Assets:
  • web-prod-01 (10.10.1.101) - Critical
  • web-prod-02 (10.10.1.102) - Critical
  • web-staging-03 (staging - lower priority)

CVSS Score: 9.8 (CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)

AI Hyper Tracker Analysis:
  • Risk Score: 94/100
  • Weaponized Exploit: YES (active exploitation observed in wild)
  • Exploit Probability: 96% within next 3 days
  • Model Confidence: 93%
  • Recommended Action: PATCH IMMEDIATELY

Description:
OpenSSH's ssh-keygen contains a buffer overflow vulnerability when processing specially crafted certificate keys. Remote attackers can achieve code execution on systems that validate attacker-controlled keys (typical in authentication workflows).

Affected Versions:
  • OpenSSH < 9.4p1
  • OpenSSH 9.4p1 and later are not vulnerable

Recommendations (from AI):
1. IMMEDIATE: Patch to OpenSSH 9.4p1 or later on web-prod-01 and web-prod-02 within 24 hours.
2. Consider temporary mitigation if patching delayed: disable certificate validation (not recommended).
3. Monitor authentication logs for exploitation attempts.
4. web-staging-03 can wait until next maintenance window (lower criticality).

Patch Window:
  Production servers: Available Sundays 02:00-06:00 UTC. For emergency, override available with manager approval.

Asset Owners:
  web-prod team: web-team@company.com
  Notified: Yes (auto-assigned)

Evidence:
  • CVE description: https://nvd.nist.gov/vuln/detail/CVE-2024-3094
  • Metasploit module: https://github.com/rapid7/metasploit-framework/pull/12345
  • AI analysis: https://hyper-tracker:8080/events/evt_f3a1b9c2?view=ai

---
[Auto-generated by Hyper Tracker]
[Track: evt_f3a1b9c2]
```

**Slack alert received in #sec-critical**:
```
🚨 CRITICAL VULNERABILITY - WEAPONIZED

CVE: CVE-2024-3094
Asset: web-prod-01 (10.10.1.101) • Production • Criticality: HIGH
CVSS: 9.8 • AI Risk: 94/100 • Weaponized: YES (active exploitation)
Title: OpenSSH ssh-keygen buffer overflow RCE
AI Confidence: 93%

Recommendation: IMMEDIATE PATCH REQUIRED - Exploit observed in wild

[JIRA Ticket](https://jira.company.com/browse/SEC-12456) | [Dashboard](http://hyper-tracker:8080/events/evt_f3a1b9c2)

Created: 2 minutes ago
[@here] @security-team @web-team
```

**Analyst response workflow**:

```bash
# 1. Acknowledge by assigning to self and adding comment
hyper-tracker update evt_f3a1b9c2 \
  --assignee senior-analyst@company.com \
  --add-comment "Acknowledged. Will coordinate patch with web-team. Checking if any exploit attempts observed in our logs."

# 2. Search for related IOC matches
hyper-tracker search --ioc "185.220.101.77" --limit 10
# (shows 2 incidents from Cobalt Spider campaign)

# 3. Run AI prediction to assess likelihood of exploitation
hyper-tracker predict evt_f3a1b9c2 --scenario exploitation
# Returns: "exploit_probability: 0.96 (weaponized)"

# 4. Check if any assets already compromised
hyper-tracker search --asset-id web-prod-01 --status "!resolved" --type incident
# (returns 0 - good)

# 5. Change SLA status (from P0 to investigating, pauses SLA timer)
hyper-tracker update evt_f3a1b9c2 --status investigating --add-comment "Investigation: no evidence of exploitation yet. Log review in progress."

# 6. After patch applied
hyper-tracker update evt_f3a1b9c2 \
  --status resolved \
  --resolution "OpenSSH upgraded to 9.4p1 on both web-prod-01 and web-prod-02 during emergency maintenance window. Verified by checking `sshd -T | grep version` and running test scan." \
  --resolution-by senior-analyst@company.com \
  --add-evidence /tmp/patch-verification.txt

# 7. Close linked JIRA ticket automatically (if configured)
# (happens via webhook)
```

### Example 2: Incident response - suspected ransomware

**Trigger**: SIEM alert for unusual SMB activity

```bash
# Webhook received and auto-created incident
hyper-tracker show inc_9a2b3c4d --timeline --full
```

Output:
```json
{
  "incident_id": "inc_9a2b3c4d",
  "type": "incident",
  "severity": "critical",
  "priority": "P0",
  "title": "Suspicious SMB activity and encryption patterns on finance-db-01",
  "status": "open",
  "assignee": "tier2-team@company.com",
  "sla": {
    "acknowledge_by": "2024-03-15T15:20:00Z (37m remaining)",
    "contain_by": "2024-03-15T16:20:00Z (1h 37m remaining)",
    "resolve_by": "2024-03-16T14:23:00Z (23h 37m remaining)",
    "status": "running"  // not paused
  },
  "timeline": [
    {"timestamp": "2024-03-15T14:23:00Z", "user": "system", "action": "incident_created", "details": "Auto-created from SIEM alert rule 'SMB Unusual Encryption'"},
    {"timestamp": "2024-03-15T14:23:05Z", "user": "system", "action": "ioc_enriched", "details": "IOC 185.220.101.77 matched 'Cobalt Spider' campaign (94% confidence)"},
    {"timestamp": "2024-03-15T14:23:10Z", "user": "system", "action": "ai_analysis", "details": "AI predicted ransomware with 87% confidence. Recommended: isolate immediately, preserve memory, notify legal."},
    {"timestamp": "2024-03-15T14:23:15Z", "user": "system", "action": "alert_fired", "details": "Alert sent to #incident-response"},
    {"timestamp": "2024-03-15T14:24:00Z", "user": "system", "action": "assignment", "details": "Auto-assigned to tier2-team@company.com based on asset ownership"},
    {"timestamp": "2024-03-15T14:25:30Z", "user": "tier2-analyst@company.com", "action": "comment", "details": "Acknowledged. Initiating containment procedures. Isolating finance-db-01 from network."}
  ],
  "ai_analysis": {
    "incident_type": "ransomware",
    "confidence": 0.87,
    "attacker_profile": "cybercrime (ransomware-as-a-service)",
    "likely_ransomware_family": "LockBit 3.0",
    "attack_chain": [
      {"stage": "initial_access", "technique": "T1133 - External Remote Services", "confidence": 0.82},
      {"stage": "persistence", "technique": "T1136 - Create Account", "confidence": 0.76},
      {"stage": "lateral_movement", "technique": "T1021 - Remote Services (SMB)", "confidence": 0.89},
      {"stage": "impact", "technique": "T1486 - Data Encryption", "confidence": 0.94}
    ],
    "iocs_linked": [
      {"value": "185.220.101.77", "type": "ip", "confidence": 0.94, "campaign": "Cobalt Spider"},
      {"value": "e6a40a3d8f1c9b2e5d7f6a8b9c0d1e2f", "type": "file_hash", "confidence": 0.91}
    ],
    "recommended_actions": [
      "IMMEDIATE: Isolate finance-db-01 from network (already done)",
      "Preserve memory dump for forensic analysis",
      "Review backup integrity - ensure backups not accessible from production network",
      "Do NOT pay ransom - contact law enforcement (FBI Cyber)",
      "Check for lateral movement to other financial systems",
      "Enable MFA on all admin accounts if not already"
    ],
    "similar_incidents": [
      {"incident_id": "inc_7x8y9z", "date": "2024-01-15", "outcome": "Contained, no ransom paid, data restored from backup", "confidence": 0.89},
      {"incident_id": "inc_5a6b7c", "date": "2023-11-02", "outcome": "Ransom paid $50k (not recommended)", "confidence": 0.76}
    ]
  },
  "evidence": [
    {"filename": "network-isolation-proof.txt", "added": "2024-03-15T14:25:00Z", "sha256": "a1b2c3..."},
    {"filename": "memory-dump.raw", "added": "2024-03-15T14:45:00Z", "sha256": "d4e5f6..."}
  ]
}
```

**Analyst workflow with playbook**:

```bash
# 1. View recommended actions in detail
hyper-tracker recommend inc_9a2b3c4d --type containment --limit 10
```

Output:
```json
{
  "incident_id": "inc_9a2b3c4d",
  "recommendations": [
    {
      "step": 1,
      "action": "Isolate compromised system from network",
      "details": "Remove finance-db-01 from network segment. Use network switch port disable or host-based firewall. preserve evidence.",
      "commands": [
        "On network switch: disable port Gi1/0/24",
        "On Windows host: netsh advfirewall set allprofiles firewallpolicy blockinbound,blockoutbound"
      ],
      "sla_timer": "immediate (SLA: 1 hour)",
      "evidence_required": "Network isolation confirmation (screenshot or ticket)"
    },
    {
      "step": 2,
      "action": "Preserve volatile evidence",
      "details": "Before powering off, collect memory dump and running process list. Use FTK Imager or DumpIt.",
      "commands": [
        "Memory: dumpit.exe -o memory.raw",
        "Process list: pslist > processes.txt",
        "Network connections: netstat -ano > connections.txt"
      ],
      "sla_timer": "within 30 minutes",
      "evidence_required": ".raw memory file, process list, network connections"
    },
    {
      "step": 3,
      "action": "Check backup integrity",
      "details": "Verify backups for affected systems are not accessible from production network and are free from malware.",
      "commands": [
        "Review backup file timestamps - any modifications in last 24h?",
        "Mount backup read-only and scan with antivirus",
        "Test restore of critical database to isolated environment"
      ],
      "sla_timer": "within 2 hours",
      "evidence_required": "Backup scan report, restore test log"
    },
    {
      "step": 4,
      "action": "Scope lateral movement",
      "details": "Check for authentication from finance-db-01 to other systems. Look for SMB, RDP, SSH connections.",
      "queries": [
        "SIEM: source IP = finance-db-01 AND dest_port in (445,3389,22)",
        "Azure AD: sign-ins from compromised user accounts",
        "Check for new admin accounts created in last 24h"
      ],
      "sla_timer": "within 4 hours",
      "evidence_required": "Log query results, list of potentially compromised accounts"
    }
  ]
}
```

**Analyst completes steps**:

```bash
# 2. Update incident with evidence and progress
hyper-tracker update inc_9a2b3c4d \
  --status investigating \
  --add-comment "Step 1 complete: finance-db-01 isolated at network switch port Gi1/0/24. See evidence network-isolation-proof.txt. Memory dump collected (memory.raw, sha256: d4e5f6...)." \
  --add-evidence /evidence/network-isolation-proof.txt \
  --add-evidence /evidence/memory-dump.raw

# 3. Run AI prediction for next attacker moves
hyper-tracker predict inc_9a2b3c4d --scenario lateral-movement
```

Output:
```json
{
  "predictions": [
    {
      "step": 1,
      "probability": 0.87,
      "technique": "T1078 - Valid Accounts",
      "target": "domain controllers",
      "timeline": "24-48 hours",
      "trigger": "Attacker already has domain credentials from finance-db-01",
      "recommended_defenses": [
        "Enable PIM for all privileged accounts (not yet deployed)",
        "Implement LAPS on all workstations",
        "Monitor for authentication from unusual locations"
      ]
    }
  ]
}
```

**Creating evidence log**:

```bash
# 4. After containing and resolving
hyper-tracker update inc_9a2b3c4d \
  --status resolved \
  --resolution "Ransomware contained. No ransom paid. Attacker gained initial access via compromised service account (password reuse). Service account disabled. System rebuilt from clean image. Data restored from offline backup. No evidence of data exfiltration. Postmortem: https://confluence.company.com/incidents/inc_9a2b3c4d" \
  --resolution-by tier2-analyst@company.com \
  --resolution-date 2024-03-15T18:30:00Z \
  --add-tag ransomware,lockbit,resolved-no-ransom \
  --add-comment "Lessons learned: Implement PAM, reduce service account usage, enforce MFA for admin access. Action items: 1) Deploy PIM by Q2, 2) Audit all service accounts, 3) Enable MFA for all admins by Apr 15."
```

**SLA results**:
- Acknowledge: 7 minutes (within 15m) ✓
- Contain: 4 hours (within 1h SLA) ✗ (breached)
- Reason: "Memory dump collection took longer than expected (2 hours). Will update playbook to parallelize tasks."
- Resolve: 4h 7m total (within 24h) ✓

**Alert sent to management**:
```
[SLACK #security-management]
⚠️ INCIDENT BREACHED SLA
Incident: inc_9a2b3c4d - Ransomware - finance-db-01
SLA: Containment (1 hour) - Breached by I:03:07
Current status: Resolved (no ransom paid, data restored)
Owner: tier2-analyst@company.com
Breach reason: Memory dump collection took longer than expected
Postmortem: https://confluence.company.com/incidents/inc_9a2b3c4d
```

### Example 3: IOC enrichment and threat intelligence

**Scenario**: New firewall block log shows IP 185.220.101.77 trying to connect to multiple servers. Analyst checks IP:

```bash
hyper-tracker enrich-ioc 185.220.101.77 --sources all --force
```

Output:
```json
{
  "ioc": "185.220.101.77",
  "type": "ip",
  "enrichments": {
    "virustotal": {
      "api_version": "v3",
      "response_code": 1,
      "last_analysis_date": "2024-03-15T10:23:00Z",
      "stats": {
        "harmless": 0,
        "malicious": 45,
        "suspicious": 2,
        "undetected": 0,
        "timeout": 28
      },
      "detection_ratio": "45/75",
      "categories": ["c2", "malware"],
      "tags": ["cobalt-spider", "ransomware", "c2-server"],
      "last_analysis_stats": {"harmless":0,"malicious":45,"suspicious":2,"undetected":0,"timeout":28}
    },
    "alienvault_otx": {
      "pulse_count": 3,
      "pulses": [
        {"id": "5a1b2c3d4e5f", "name": "Cobalt Spider December 2023", "created": "2023-12-01"},
        {"id": "6b7c8d9e0f1a", "name": "Emerging C2 Infrastructure", "created": "2024-01-15"},
        {"id": "7c8d9e0f1a2b", "name": "LockBit Ransomware Infrastructure", "created": "2024-03-10"}
      ],
      "malware_families": ["Cobalt Strike", "LockBit"],
      "attack_techniques": ["T1055", "T1218", "T1132.001", "T1573"],
      "last_pulse": "2024-03-10"
    },
    "greynoise": {
      "seen": true,
      "classification": "malicious",
      "last_seen": "2024-03-15T08:45:00Z",
      "first_seen": "2023-11-15",
      "noise": true,
      "tor": false,
      "category": "malicious"
    },
    "abuseipdb": {
      "confidence_score": 95,
      "total_reports": 1247,
      "last_reported_at": "2024-03-15T06:30:00Z",
      "countries": ["RU", "DE", "NL"],
      "usage_type": "data center/web hosting"
    }
  },
  "hyper_tracker_correlation": {
    "events_linked": 14,
    "incidents_containing": 3,
    "first_seen_in_system": "2023-12-02T14:23:00Z",
    "last_seen_in_system": "2024-03-15T08:45:00Z",
    "campaign_association": {
      "campaign": "Cobalt Spider",
      "confidence": 0.94,
      "first_seen": "2023-12-01",
      "last_seen": "2024-03-14",
      "related_iocs": 47,
      "incidents_attributed": 32
    }
  }
}
```

**Actions taken**:

```bash
# 1. Check which incidents already tracked
hyper-tracker search --ioc 185.220.101.77 --format table

# 2. Search for all assets that saw traffic from this IP
hyper-tracker search --query "185.220.101.77" --format table

# 3. If new and malicious, block firewall rule (using integration)
hyper-tracker alert add \
  --name "Firewall block automatically" \
  --condition '{"and": [{"==": [{"var": "ioc"}, "185.220.101.77"]}, {">": [{"var": "risk_score"}, 90]}]}' \
  --channel webhook \
  --recipient https://firewall.company.com/api/block \
  --template '{"action":"block_ip","ip":"{{ioc}}","duration":"permanent","reason":"{{ai_analysis.campaign_association.campaign}} - confidence {{ai_analysis.campaign_association.confidence * 100}}%"}'

# (Output: webhook fired successfully, firewall acknowledged)

# 4. Add IOC to watchlist if not present
hyper-tracker ioc add \
  --value 185.220.101.77 \
  --type ip \
  --source virustotal \
  --confidence 0.95 \
  --threat-type c2 \
  --campaign "Cobalt Spider" \
  --first-seen 2023-11-15
```

**Firewall webhook payload**:
```json
{
  "action": "block_ip",
  "ip": "185.220.101.77",
  "duration": "permanent",
  "reason": "Cobalt Spider campaign - confidence 94%",
  "source": "Hyper Tracker",
  "timestamp": "2024-03-15T14:45:00Z",
  "ioc_linked_incidents": 3,
  "recommendation": "Monitor for related IOCs and lateral movement attempts"
}
```

**Firewall response**:
```
{"status": "success", "rule_id": "HT-BLOCK-20240315-184500-18522010177", "message": "IP added to permanent block list. Will persist across reboots."}
```

**Auto-comment added to all incidents containing this IOC**:

```bash
# (automated by webhook action)
hyper-tracker update inc_7x8y9z \
  --add-comment "2024-03-15 14:45: IP 185.220.101.77 blocked at firewall perimeter. Additional context: linked to LockBit activity per OTX pulse 7c8d9e0f1a2b."
```

### Example 4: Compliance evidence collection

**PCI-DSS Requirement 8.2.3**: "Review system user access at least quarterly."

```bash
# 1. Search all user access review events
hyper-tracker search \
  --type compliance \
  --tag pci-dss-8.2.3 \
  --date-start 2024-01-01 \
  --date-end 2024-03-31 \
  --format json
```

Returns:
```json
[
  {
    "event_id": "comp_abc123",
    "type": "compliance",
    "title": "Q1 2024 PCI-DSS 8.2.3 User Access Review",
    "status": "completed",
    "compliance_framework": "PCI-DSS",
    "control": "8.2.3",
    "period": "2024-01-01 to 2024-03-31",
    "assets_covered": 856,
    "users_reviewed": 1247,
    "exceptions": 3,
    "evidence_attached": [
      "/evidence/pci-8.2.3-q1-2024-summary.pdf",
      "/evidence/user-review-spreadsheet.xlsx"
    ],
    "reviewer": "compliance-officer@company.com",
    "completion_date": "2024-03-10",
    "tags": ["pci-dss", "access-review", "quarterly"]
  }
]

# 2. Generate compliance gap report for auditor
hyper-tracker report compliance-gap \
  --framework PCI-DSS \
  --date-range "2024-01-01 to 2024-03-31" \
  --output /tmp/pci-dss-q1-2024-report.pdf \
  --format pdf
```

Report snippet:
```
PCI-DSS v4.0 COMPLIANCE STATUS - Q1 2024
Generated: 2024-03-15 14:30:00 UTC
Period: January 1, 2024 - March 31, 2024

REQUIREMENT 8: IDENTIFY AND AUTHENTicate ACCESS TO SYSTEM COMPONENTS
--------------------------------------------------------------------------------

8.2.3 - Quarterly user access review
Status: COMPLIANT ✓
Evidence: comp_abc123 (completed 2024-03-10)
Coverage: 856 systems, 1247 users reviewed
Exceptions: 3 (all approved, documented)
Reporter: compliance-officer@company.com

Evidence trail:
  • Review conducted via Hyper Tracker compliance event
  • Access review spreadsheet uploaded as evidence (signed PDF)
  • Automated verification: Access matrix matched against HR records
  • Exceptions: 2 terminated employees still had access (corrected within 24h), 1 contractor extension approved

Automated control testing passed:
  ✓ All user accounts reviewed within quarter
  ✓ Review documented with approver signature
  ✓ Exceptions tracked and remediated

REQUIREMENT 6: DEVELOP AND MAINTAIN SECURE SYSTEMS AND APPLICATIONS
--------------------------------------------------------------------------------

6.2 - Ensure that all system components and software are protected from known vulnerabilities
Status: PARTIALLY COMPLIANT (87/100) ⚠

Critical vulnerabilities open > 30 days: 3 (↓2 vs last quarter)
   CVE-2024-3094 - web-prod-01 (age: 15 days) - P0
   CVE-2024-2987 - payment-db-01 (age: 42 days) - P1
   CVE-2023-38408 - ldap-server-01 (age: 67 days) - P1

Compliance gap:
  • 2 high-risk vulnerabilities exceeding 30-day remediation window
  • Payment-db-01 (asset criticality: CRITICAL) overdue by 12 days

Required action:
  1. CVE-2024-2987: Patch by March 27 (11 days remaining)
  2. CVE-2023-38408: Schedule emergency maintenance (overdue 37 days)

Attested by: CISO - pending
```

### Example 5: Bulk import and deduplication

**Scenario**: Migrating from legacy ticketing system with 10,000 incidents.

```bash
# Dry-run to validate mapping
hyper-tracker import \
  --source ./legacy-incidents-2023.jsonl \
  --format jsonl \
  --map-fields '{"type": "incident_type", "severity": "priority", "title": "subject", "details": "description", "create_date": "opened_at", "close_date": "resolved_at", "assignee": "owner_email", "tags": "tags"}' \
  --dry-run
```

Dry-run report:
```
DRY RUN - No data will be modified
Source: ./legacy-incidents-2023.jsonl (9,847 lines)
Estimated events: 9,847
Mapping validation:
  ✓ type mapping OK (legacy: ['sec_incident','info_event','malware'] → ['incident','incident','incident'])
  ✓ severity mapping OK (legacy P1-5 → Hyper Tracker high/medium/low)
  ✓ date parsing OK (ISO 8601 format detected)
  ⚠ 142 events missing 'assignee' → will default to 'security-team@'
  
Duplicate detection:
  • 1,234 events already exist (by CVE hash or title+asset fuzzy match)
  • Would skip: 1,234 (12.5%)
  • Would create: 8,613 (87.5%)

New assets discovered:
  • 89 assets not in current inventory
  • Auto-creation: YES (--create-assets flag not set, use --auto-create-assets to enable)

Confidence: HIGH - ready to proceed

Estimated duration: 15-20 minutes
```

Actual import with auto-create assets:

```bash
hyper-tracker import \
  --source ./legacy-incidents-2023.jsonl \
  --format jsonl \
  --map-fields ./mapping.json \
  --batch-size 1000 \
  --skip-duplicates \
  --auto-create-assets \
  --update-existing
```

Progress output (real-time):
```
Importing: 100% |██████████████████████████| 9847/9847 [15m 42s elapsed, 0s remaining]

✓ Import completed successfully
  • Events processed: 9,847
  • Events created: 8,613 (new)
  • Events skipped (duplicates): 1,234
  • Events updated (existing): 0
  • New assets created: 89
  • Validation errors: 0

AI enrichment queued for 8,613 events
  Job ID: ai_import_20240315_abc123
  Estimated completion: 3 hours (10 workers)
  Monitor: hyper-tracker model status

Post-import recommendations:
  1. Run AI enrichment: Already queued (will complete automatically)
  2. Generate import summary report: hyper-tracker report import-summary --session ai_import_20240315_abc123
  3. Review new assets: hyper-tracker asset list --tag auto-created-import-20240315
  
Logs: /var/log/hyper-tracker/import_20240315_abc123.log
```

**Automatic AI enrichment processing**:

```bash
# Check AI job status
hyper-tracker model status
```

Output:
```
AI Workers: 10 active
Queue:
  • vulnerability events: 4,231 (est. 1h 15m)
  • incident events: 4,382 (est. 2h 30m)
  
Currently processing: vulnerability model (batch 432/4231)
Last completed batch: 431 events in 89s (4.8 events/sec)

Expected completion:
  • All events enriched by: 2024-03-15 17:30 UTC
```

After enrichment, analyst reviews a sample:

```bash
hyper-tracker search --tag auto-created-import-20240315 --limit 5 --format table
```

Table output:
```
ID              TYPE      SEVERITY  TITLE                                    ASSETS  AI_RISK  STATUS
evt_x1y2z3a     incident  high      Suspicious outbound traffic detected   2       78       open
evt_b4c5d6e     vuln      medium    Apache Log4j CVE-2021-44228            1       65       resolved
evt_f7g8h9i     incident  critical  Ransomware on file-server-01           1       96       closed
evt_j0k1l2m     vuln      unknown   WordPress plugin XSS                  1       45       open
evt_n3o4p5q     incident  low       Phishing email reported                0       22       closed
```

**Analyst adds missing context to legacy events**:

```bash
# Add MITRE ATT&CK technique to all phishing incidents from import
hyper-tracker search --tag auto-created-import-20240315 --type incident --query "phishing" --format json \
  | jq -r '.[] | .event_id' \
  | xargs -I {} hyper-tracker update {} --add-tag technique:T1566.002
```

### Example 6: Model training and evaluation

**Problem**: AI model underestimates risk for cloud-native vulnerabilities (containers, serverless).

```bash
# 1. Export training data from last year's incidents
hyper-tracker export \
  --table events \
  --filters '{"type":"vulnerability","created_at":{"$gte":"2023-01-01","$lte":"2023-12-31"}}' \
  --output /tmp/training_vulns_2023.jsonl \
  --format jsonl
```

**2. Label dataset with outcome (was this vuln exploited?)**

Convert to training format:
```python
# scripts/prepare_training.py
import json
with open('/tmp/training_vulns_2023.jsonl') as f_in, open('/tmp/train.jsonl', 'w') as f_out:
    for line in f_in:
        event = json.loads(line)
        # Ground truth: exploited if linked incident with ransomware/apt
        exploited = 1 if any(
            inc.get('ai_analysis',{}).get('incident_type') in ['ransomware','apt','malware']
            for inc in event.get('linked_incidents', [])
        ) else 0
        
        training_example = {
            "cve": event.get('cve'),
            "description": event.get('details', '')[:500],
            "asset_criticality": event.get('asset', {}).get('criticality', 'medium'),
            "environment": event.get('asset', {}).get('environment', 'production'),
            "cvss": event.get('cvss'),
            "epss_score": event.get('epss_score'),
            "exploited": exploited,
            "days_to_exploit": event.get('days_to_exploit')  # from linked incident
        }
        f_out.write(json.dumps(training_example) + '\n')
```

**3. Train new model**:

```bash
hyper-tracker model train \
  vulnerability \
  --dataset /tmp/train.jsonl \
  --epochs 5 \
  --validation-split 0.2 \
  --learning-rate 2e-5 \
  --batch-size 32 \
  --backup-before \
  --dry-run
```

Dry-run output:
```
Model: vulnerability
Dataset: /tmp/train.jsonl (45,231 examples)
Training examples: 36,185
Validation examples: 9,046
Epochs: 5
Batch size: 32
Learning rate: 2e-5
Backup current model: YES (to /var/lib/hyper-tracker/models/backup/vulnerability-20240315-143022/)

Estimated duration: 2-3 hours (GPU: auto-detect)
Estimated disk space: 500MB for new model
 
DRY RUN - No changes will be made
To execute training, remove --dry-run flag

Proposed changes:
  • New model: vulnerability-custom-20240315 (size: ~500MB)
  • Will replace: vulnerability (current: v2.1)
  • Backup retained: YES (30 days)
  • Validation required: YES (accuracy > 0.85)
```

Actual training (with GPU):
```bash
hyper-tracker model train \
  vulnerability \
  --dataset /tmp/train.jsonl \
  --epochs 5 \
  --backup-before
```

Progress output:
```
Starting training: vulnerability-custom-20240315
✓ Dataset loaded: 45,231 examples (36,185 train, 9,046 val)
✓ Model initialized: microsoft/cvebert-base-uncased
✓ Loss: CrossEntropy, Optimizer: AdamW
───────────────────────────────────────────────────
Epoch 1/5 [█████████░░░░░░░░░░] 65% - loss: 0.4234 - val_loss: 0.3876 - val_acc: 0.8234
Epoch 2/5 [███████████░░░░░░] 45% - loss: 0.3123 - val_loss: 0.2987 - val_acc: 0.8541
Epoch 3/5 [████████████░░░░] 23% - loss: 0.2567 - val_loss: 0.2734 - val_acc: 0.8678
───────────────────────────────────────────────────
Training completed: 5/5 epochs
Final metrics:
  • Training accuracy: 0.8912
  • Validation accuracy: 0.8734
  • F1 score (exploited): 0.8245
  
Model saved: /var/lib/hyper-tracker/models/vulnerability-custom-20240315/
Backup created: /var/lib/hyper-tracker/models/backup/vulnerability-v2.1-20240315-143022/
```

**4. Evaluate model on holdout test set**:

```bash
hyper-tracker model evaluate \
  vulnerability-custom-20240315 \
  --test-set /tmp/test_set_2024.jsonl \
  --metrics all
```

Output:
```
Model: vulnerability-custom-20240315
Test set: 11,307 examples

METRICS
--------
Overall Accuracy: 0.8691
Precision (exploited): 0.8342
Recall (exploited): 0.8123
F1-Score (exploited): 0.8231
Precision (not exploited): 0.9012
Recall (not exploited): 0.9234
F1-Score (not exploited): 0.9121

Confusion Matrix:
                 Predicted
                 exploited  not_exploited
Actual exploited     2,104         486
      not_exploited    589       8,128

AUC-ROC: 0.9123

BY SEVERITY STRATUM
-------------------
Critical CVEs: precision=0.89, recall=0.87, f1=0.88 (3,241 examples)
High CVEs: precision=0.82, recall=0.79, f1=0.80 (4,102 examples)
Medium/Low: precision=0.76, recall=0.71, f1=0.73 (3,964 examples)

MODEL QUALITY: ACCEPTABLE FOR PRODUCTION ✓
Recommendation: Deploy to 10% canary first, monitor false positive rate.
```

**5. Promote model to production**:

```bash
hyper-tracker model promote \
  vulnerability-custom-20240315 \
  --canary-percent 10 \
  --monitor-period 7d \
  --rollback-on-fp-rate >0.15
```

Output:
```
Model promotion initiated:
  • Current production: vulnerability-v2.1 (100% traffic)
  • New model: vulnerability-custom-20240315
  • Canary: 10% of new events will use new model
  • Rollback trigger: false positive rate > 15%
  • Monitor period: 7 days
  
Dashboard: /hyper-tracker/admin/models?promotion=abc123
Notifications: #security-ml (alerts), security-team@company.com (daily digest)

Next steps:
  1. Monitor false positive rate daily
  2. If FP rate < 10% after 3 days, gradually increase to 25%, 50%, 100%
  3. If FP rate > 15% at any point, automatic rollback triggered
  4. Success metric: maintain >85% precision while reducing false negatives by 20%
```

### Example 7: SLA monitoring and breach handling

**Regular SLA status check**:

```bash
hyper-tracker sla status --format table --show-breaches
```

Output:
```
INCIDENT SLA STATUS - Real-time
┌─────────────────────┬─────────────┬─────────────┬─────────────┬──────────────┐
│ Incident            │ Priority    │ Ack Deadline│ Con Deadline│ Res Deadline │
├─────────────────────┼─────────────┼─────────────┼─────────────┼──────────────┤
│ inc_9a2b3c4d        │ P0 (critical)│ 14:20 (7m) │ 15:20 (1h 7m)│ Tomorrow 14:23│
│ inc_9a2b3c4e        │ P0 (critical)│ 14:25 (2m) │ 15:25 (1h 2m)│ Tomorrow 09:15│
│ inc_9a2b3c4f        │ P1 (high)   │ 16:00 (2h 35m)│ 20:00 (6h 35m)│ 2024-03-18  │
└─────────────────────┴─────────────┴─────────────┴─────────────┴──────────────┘

SLA COMPLIANCE (Last 7 days)
├─ P0 acknowledge: 95% (19/20) ⚠ 1 breach
├─ P0 containment: 80% (16/20) ⚠ 4 breaches
├─ P0 resolution: 70% (14/20) ⚠ 6 breaches
├─ P1 acknowledge: 98% (49/50) ✓
├─ P1 containment: 92% (46/50) ⚠ 4 breaches
└─ Overall: 89% ✓

CURRENT BREACHES (3)
────────────────────
1. inc_9a2b3c4d - P0 containment breached in 1h 7m (SLA: 1h)
   Reason: Memory dump collection took longer than expected
   Owner: tier2-analyst@company.com
   Escalation: Will notify management at breach+30m

2. inc_9a2b3c4e - P0 acknowledge at 14:28 (SLA: 14:25)
   Reason: Assignment delay - waiting on on-call responder
   Owner: tier1-team@company.com

3. inc_9a2b3c4f - P1 containment breached in 6h 35m (SLA: 8h)
   Reason: Pending legal approval for system isolation
   Owner: legal-team@company.com

---
Full SLA report: /var/reports/sla/last-7d.html
```

**Automated SLA breach notification**:

```bash
hyper-tracker alert add \
  --name "SLA Breach" \
  --condition '{"==": [{"var": "sla.status"}, "breached"]}' \
  --channel slack \
  --recipient "#security-management" \
  --template "sla-breach"
```

Slack message:
```
🚨 SLA BREACH DETECTED

Incident: inc_9a2b3c4d
Type: P0 Incident - Ransomware
SLA: Containment (1 hour) - Breached by 1 hour 7 minutes
Owner: tier2-analyst@company.com
Breach reason: "Memory dump collection took longer than expected"
Current status: Investigating

Required: Manager review and postmortem within 24 hours
[View Incident](http://hyper-tracker:8080/incidents/inc_9a2b3c4d)
```

**Manager overrides SLA timer** (valid reason):

```bash
hyper-tracker update inc_9a2b3c4d \
  --add-comment "SLA pause approved by manager@company.com: legitimate reason - memory preservation took precedence over containment. SLA pause recorded from 14:45 to 15:30 (45 minutes). New containment deadline: 16:12." \
  --sla-pause 45m \
  --sla-pause-reason "Forensic memory collection priority"
```

Now SLA countdown adjusts:
```
Containment SLA: 45m paused → new deadline: 16:12
```

### Example 8: Dashboard queries and visualizations

**Security manager views dashboard**:

```
http://hyper-tracker:8080
Authentication: Basic (SSO via LDAP)
```

**Overview tab**:
```
CARD 1: Open Incidents by Priority
  P0: 3 (↑1 from yesterday)
  P1: 7 (→)
  P2: 13 (↓2)
  P3: 0

CARD 2: Critical Vulnerabilities (risk_score > 90)
  2 (↑1)
  ├─ web-prod-01: CVE-2024-3094 (94)
  └─ payment-db-01: CVE-2024-2987 (91) - 42 days overdue

CARD 3: MTTR (Last 30 days)
  P0: 2h 15m (target: <4h) ✓
  P1: 8h 42m (target: <24h) ✓
  P2: 3d 2h (target: <7d) ✗

CARD 4: New Events (24h)
  Total: 47 (↑12%)
  Vulnerabilities: 23 (↑5)
  Incidents: 12 (↑4)
  IOCs: 8 (↑2)
  Other: 4

CHART: Events by Severity (7-day trend)
  • Line chart with critical/↑, high/↔, medium/↓, low/→
  
CHART: SLA Compliance (weekly bar)
  • This week: 89% (↓2% from last week)
```

**Assets tab**:
- Tree view: Production (45 assets) → Team: Web (12), Database (8), Network (5)...
- Heatmap: color-coded by max risk_score on asset (red: >85, orange: 70-85, yellow: 50-70, green: <50)
- Click asset "web-prod-01":
  ```
  Asset: web-prod-01 (10.10.1.101)
  Criticality: HIGH
  Environment: Production
  Owner: web-team@company.com
  
  Vulnerabilities (open):
  ┌─────────────────────────────┬────────────┬────────────┬────────────┐
  │ CVE                         │ Severity   │ Risk Score │ Age (days) │
  ├─────────────────────────────┼────────────┼────────────┼────────────┤
  │ CVE-2024-3094               │ critical   │ 94         │ 2          │
  │ CVE-2024-2987               │ high       │ 87         │ 15         │
  └─────────────────────────────┴────────────┴────────────┴────────────┘

  Incidents (last 90 days): 3
    • inc_9a2b3c4d (ransomware) - resolved
    • inc_8z9y0x1w (SSH brute force) - resolved
    • inc_7v6w5u4t (data exfiltration) - resolved

  Risk Trend: ⬆️ (increased 12 points in last 30 days)
  Recommended: Prioritize CVE-2024-3094 patch within 48h, review CVE-2024-2987 for accelerated patch.
  ```

**IOCs tab**:
- Table: Value | Type | Threat | Campaign | First Seen | Last Seen | Events Linked
- Filter: Only show IOCs linked to active incidents
- Search: `campaign:"Cobalt Spider"`
- Click IOC 185.220.101.77:
  ```
  IOC: 185.220.101.77 (IP)
  Threat Type: C2 Server
  Campaign: Cobalt Spider (94% confidence)
  First seen: 2023-11-15
  Last seen: 2024-03-15T08:45:00Z
  
  Enrichment:
  ┌─────────────┬────────────────────────────────────────────────────────────┐
  │ VT detections│ 45/75 (harmless:0, malicious:45, suspicious:2)         │
  │ OTX pulses   │ 3 (Cobalt Spider, Emerging C2, LockBit)               │
  │ GreyNoise    │ Malicious, active, not Tor                           │
  │ AbuseIPDB    │ 1,247 reports, confidence 95, last: 6h ago          │
  └─────────────┴────────────────────────────────────────────────────────────┘
  
  Linked Events: 14
  ├─ inc_9a2b3c4d (ransomware) - open
  ├─ inc_8z9y0x1w (SSH brute) - closed
  └─ 12 other incidents (mostly Jan-Feb 2024)
  
  Action: Already blocked at firewall (rule ID: HT-BLOCK-20240315-184500)
  ```

### Example 9: Troubleshooting common issues

**Issue 1: AI analysis stuck in "queued" for hours**

```bash
# Check AI worker status
hyper-tracker model status
```

Output shows:
```
AI Workers: 0/10 (all stopped)
Queue: events: 342 (stuck)
ERROR: CUDA out of memory on worker-3
```

Solution:
```bash
# Check system resources
free -h  # RAM: 32GB total, 28GB used, 4GB free → low memory
nvidia-smi  # GPU memory: 24GB total, 22GB used → full

# Restart AI workers with smaller batch size
hyper-tracker config set ai.batch_size 16 --scope ai
hyper-tracker maintenance restart-workers ai

# Or pause other GPU jobs
# Stop other training jobs:
hyper-tracker model stop --job-id train_abc123

# Reduce queue workers to match GPU capacity
hyper-tracker config set ai.max_workers 5 --scope ai
hyper-tracker maintenance restart-workers ai

# Clear stuck queue (careful!)
hyper-tracker maintenance clear-queue --older-than 2h --dry-run
# Shows: Would clear 342 events (older than 2h)
hyper-tracker maintenance clear-queue --older-than 2h
# Clears oldest events (they'll need manual re-queue if important)
```

**Issue 2: Duplicate events appearing**

```bash
# Search for duplicates (same CVE + asset)
hyper-tracker search --type vulnerability --cve CVE-2024-3094 --asset-id web-prod-01 --format json | jq length
# Output: 5 duplicates!

# Check deduplication settings
hyper-tracker config get deduplication.enabled
# Output: true

# Check if same source creating duplicates
hyper-tracker search --cve CVE-2024-3094 --asset-id web-prod-01 --format json | jq -r '.[].source' | sort | uniq -c
# Output:
#   3 qualys
#   2 manual

# The qualys source ran multiple times in short period and not deduping properly.

# Fix: Verify deduplication hash function
hyper-tracker maintenance dedupe-check --cve CVE-2024-3094

# Output shows that events from same day but different times have different hashes due to timestamp field included.

# Solution: Reconfigure dedupe to ignore timestamp and source field
hyper-tracker config set deduplication.fields "type,cve,asset_ids,title" --scope database
# This ensures same CVE on same asset will dedupe regardless of ingestion time/source.

# Manual merge duplicates:
hyper-tracker dedupe merge evt_abc123 evt_def456 evt_ghi789 \
  --keep-newest \
  --reason "Duplicate from Qualys re-scan"

# Now search again: should be 1 event.
```

**Issue 3: Alerts not firing**

```bash
# 1. Check alert rule exists and is enabled
hyper-tracker alert list --format table

# 2. Check if condition matches any events
hyper-tracker alert test --name "Critical prod vuln - weaponized" --count 10

# This tests rule against last 10 events
# Output: "0 matches" - rule condition not matching expected events

# 3. Inspect rule condition
hyper-tracker alert show "Critical prod vuln - weaponized"
# Shows condition JSON

# Test condition directly:
hyper-tracker search --format json --limit 1 | jq -c '.[]' | hyper-tracker alert test-json --condition '{"and": [...]}' 
# (Use condition from rule)

# Common issue: asset.environment field name mismatch
# Check actual asset schema:
hyper-tracker asset show web-prod-01 --format json | jq '.environment'
# Output: "production" (lowercase)

# But rule condition used: {"==": [{"var": "asset.environment"}, "Production"]} (capitalized!)

# Fix: Update rule condition
hyper-tracker alert update "Critical prod vuln - weaponized" \
  --condition '{"and": [{"==": [{"var": "type"}, "vulnerability"]}, {">": [{"var": "risk_score"}, 85]}, {"==": [{"var": "asset.environment"}, "production"]}, {"==": [{"var": "ai_analysis.outputs.exploit_prediction.weaponized"}, true]}]}' \
  --dry-run
# (verify condition looks right)
hyper-tracker alert update "Critical prod vuln - weaponized" --condition <same>

# 4. Check alert throttling/cooldown
hyper-tracker status | grep throttle
# Could show: "throttle: 5 alerts sent in last hour, limit: 5/hour"

# That's why new alerts not firing! Wait for cooldown or increase limit.

# Option: reduce cooldown temporarily for testing
hyper-tracker alert update "Critical prod vuln - weaponized" --cooldown 30
```

**Issue 4: Dashboard not loading or slow**

```bash
# 1. Check dashboard service
hyper-tracker status | grep dashboard
# Output: "Dashboard: http://localhost:8080 (status: down)"

# 2. Check logs
tail -100 /var/log/hyper-tracker/dashboard.log | grep ERROR

# Common error: "Database is locked"
# Cause: WAL checkpoint not running, long-running queries holding lock.

# Fix:
hyper-tracker maintenance vacuum --aggressive
# This runs SQLite VACUUM and checkpoint

# 3. Check database size and indexing
ls -lh /var/lib/hyper-tracker/data.db*
# data.db (12GB), data.db-wal (8GB) → large WAL file indicates need for checkpoint

hyper-tracker maintenance reindex --concurrently

# 4. Check if too many concurrent dashboard users hitting unindexed queries
# Enable query logging temporarily:
hyper-tracker config set database.log_slow_queries 5 --scope database
# Logs queries taking >5s to /var/log/hyper-tracker/slow-queries.log

# Add missing indexes:
hyper-tracker maintenance add-index --table events --column created_at
hyper-tracker maintenance add-index --table events --column risk_score
```

**Issue 5: Importing fails with validation errors**

```bash
hyper-tracker import \
  --source ./data.csv \
  --format csv \
  --map-fields ./mapping.json \
  --dry-run

# Output:
Validation errors:
  • Row 234: Invalid severity 'Urgent' (allowed: critical,high,medium,low,info,unknown)
  • Row 567: Invalid date format '03/15/2024' (expected: YYYY-MM-DD or ISO 8601)
  • Row 890: Missing required field 'title' (empty string)
  • Row 1203: Asset ID 'APP-01' not found and --auto-create-assets not set

Total: 4,123 rows | Valid: 4,119 | Errors: 4
```

Fix CSV:
```bash
# 1. Pre-process CSV with sed/awk
sed -i 's/Urgent/critical/g' ./data.csv
sed -i 's#03/15/2024#2024-03-15#g' ./data.csv
awk -F, 'NR==1 || $3 != ""' ./data.csv > ./data_fixed.csv  # skip rows with empty title

# 2. Create missing assets first
cat ./data.csv | cut -d, -f5 | tail -n +2 | sort -u > ./missing-assets.txt
while read asset; do
  hyper-tracker asset add --asset-id "$asset" --hostname "$asset.company.com" --criticality medium --environment production
done < ./missing-assets.txt

# 3. Retry import
hyper-tracker import \
  --source ./data_fixed.csv \
  --format csv \
  --map-fields ./mapping.json \
  --auto-create-assets
```

## Rollback Commands

### 1. Rollback a single event creation (within 5 minutes, before AI)

```bash
hyper-tracker rollback \
  --event-id evt_XXXX \
  --reason "Created by mistake - duplicate"
```

```
✓ Event evt_XXXX deleted
Audit log entry created: "rollback: user analyst@ reason: Created by mistake - duplicate"
```

Restriction: Only works if event created <5 min ago and AI analysis not started. If AI already ran, use `update --status rejected` instead.

### 2. Revert event to previous version (from audit log)

```bash
hyper-tracker rollback \
  --event-id evt_XXXX \
  --to-version 3
```

Reverts event to state at audit log entry version 3. All intermediate updates remain in audit trail.

### 3. Undo specific field changes with surgical precision

```bash
hyper-tracker rollback \
  --event-id evt_XXXX \
  --field severity \
  --to-value high
```

Only changes specified field, keeps other updates.

### 4. Bulk rollback of import session

If bulk import was wrong (wrong mapping, duplicate storm):

```bash
hyper-tracker rollback \
  --import-session imp_20240315_abc123 \
  --reason "Wrong mapping - severity inverted" \
  --dry-run
```

Dry-run shows:
```
Would delete: 8,613 events created in session imp_20240315_abc123
Would also delete: 89 auto-created assets (unreferenced)
Would NOT delete: assets that have other events
Duration: ~5 minutes
```

Actual:
```bash
hyper-tracker rollback --import-session imp_20240315_abc123 --reason "Wrong mapping"
```

**Import session IDs** visible in logs: `/var/log/hyper-tracker/import_*.log` or in `audit_log` action=`import_session`.

### 5. Restore from backup with merge strategies

**Scenario 1**: Database corrupted, restore from yesterday's backup:

```bash
hyper-tracker restore \
  --backup /backups/hyper-tracker-2024-03-14-full.db.gz \
  --dry-run
```

```
DRY RUN
Backup: /backups/hyper-tracker-2024-03-14-full.db.gz (12.1GB)
Current DB: /var/lib/hyper-tracker/data.db (12.4GB)

Event counts:
  Backup (2024-03-14): 45,231 events
  Current (2024-03-15): 45,987 events
  Would add: 0 (nothing new in current)
  Would overwrite: 756 (events modified in last 24h)
  
Assets:
  Backup: 1,245 assets
  Current: 1,248 assets (+3 new)
  Would add: 3 new assets
  
IOCs:
  Backup: 4,521 IOCs
  Current: 4,532 IOCs (+11 new)
  Would add: 11 new IOCs (no conflicts)

Strategy (default: overwrite):
  • Events modified in last 24h from current would be LOST
  • Assets: keep newer version (based on updated_at)
  • IOCs: merge, keep highest confidence

Recommendation: Use --merge-strategy merge to preserve recent changes
```

Safe restore preserving recent work:

```bash
hyper-tracker restore \
  --backup /backups/hyper-tracker-2024-03-14-full.db.gz \
  --merge-strategy merge \
  --verify
```

`--verify` checks referential integrity after restore.

**Merge strategies**:
- `overwrite`: Backup replaces current completely (DANGEROUS)
- `skip`: Newer records in current DB are preserved, older records from backup are added
- `merge`: Three-way merge - keeps most recent version of each row by `updated_at`
- `prompt`: Ask for each conflict (only for small datasets)

**Scenario 2**: Restore single table (just events) from backup while keeping current assets:

```bash
hyper-tracker restore \
  --backup /backups/hyper-tracker-2024-03-14-full.db.gz \
  --table events \
  --merge-strategy skip \
  --dry-run
```

Would import only events created before backup date, skip events added after.

**Scenario 3**: Point-in-time recovery to before bad import:

```bash
# Find backup just before import
ls -lh /backups/ | grep 2024-03-15
# hyper-tracker-2024-03-15-14-00-full.db.gz  (14:00, before 15:30 import)

hyper-tracker restore \
  --backup /backups/hyper-tracker-2024-03-15-14-00-full.db.gz \
  --merge-strategy overwrite \
  --dry-run

# Dry-run says: would remove 8,613 imported events, restore 45,231 pre-import events

hyper-tracker restore \
  --backup /backups/hyper-tracker-2024-03-15-14-00-full.db.gz \
  --merge-strategy overwrite
```

After restore, re-queue AI enrichment for restored events if needed:
```bash
hyper-tracker ai re-queue --older-than "2024-03-15 14:00:00" --status "ai_pending"
```

### 6. Rollback AI model to previous version

If new model causes false positives:

```bash
# List models with performance
hyper-tracker model list --show-accuracy
# Shows:
# vulnerability v2.1 (prod) - accuracy 87%
# vulnerability v2.2 (canary) - accuracy 85% (FP rate 18%)

# Canary already showing worse performance, auto-rollback trigger fires:
# Message: "Model promotion failed: false positive rate 18% > threshold 15%. Rolling back to vulnerability-v2.1"

hyper-tracker model rollback \
  --model vulnerability \
  --to-version v2.1 \
  --reason "Canary FP rate exceeded threshold" \
  --force
```

Manual rollback before promotion completes:
```bash
hyper-tracker model rollback \
  --model vulnerability \
  --to-version v2.1 \
  --keep-canary \
  --reason "Production issues: false positives on cloud vulns"
```

`--keep-canary` retains v2.2 model file for investigation.

### 7. Undo bulk update (mass field change)

```bash
# Analyst accidentally set all open incidents to P0 priority
hyper-tracker update $(hyper-tracker search --status open --format json | jq -r '.[].event_id') --priority P0

# Oops - that was 247 incidents incorrectly updated.

# Rollback using audit log:
hyper-tracker audit \
  --action update \
  --user analyst@company.com \
  --date-start "$(date -d '1 hour ago' +%Y-%m-%dT%H:%M)" \
  --format json \
  | jq -r '.[] | select(.details | contains("priority")) | .event_id' \
  | sort -u \
  > /tmp/events-to-revert.txt

# Preview changes
hyper-tracker show $(head -5 /tmp/events-to-revert.txt) --full | jq '.priority'

# Bulk revert using previous value from audit
while read event_id; do
  old_priority=$(hyper-tracker audit --event-id "$event_id" --format json | jq -r '.[] | select(.action=="update" and .details | contains("priority")) | .details | capture("(?<old>p[0-3])") | .old')
  hyper-tracker update "$event_id" --priority "$old_priority" --add-comment "Rollback: incorrect mass priority update"
done < /tmp/events-to-revert.txt
```

### 8. Revert webhook/alert configuration changes

```bash
# Deleted important alert by mistake
hyper-tracker alert list --format json > ./alerts-backup.json  # (do this regularly!)

# Recreate alert from backup
jq -r '.[] | select(.name=="Critical prod vuln - weaponized")' ./alerts-backup.json \
  | hyper-tracker alert create --dry-run  # inspect
jq -r '.[] | select(.name=="Critical prod vuln - weaponized")' ./alerts-backup.json \
  | hyper-tracker alert create
```

**Configuration versioning**: Always keep `config.yaml` in git. To revert config:
```bash
git -C ~/.config/hyper-tracker log -p config.yaml
git -C ~/.config/hyper-tracker checkout HEAD~1 -- config.yaml
hyper-tracker config reload
```

### 9. Emergency: Rebuild database from events log (catastrophic failure)

If `data.db` is corrupted beyond repair but audit log intact:

```bash
# 1. Stop all services
hyper-tracker maintenance stop

# 2. Backup everything first
cp /var/lib/hyper-tracker/data.db /backups/corrupt-$(date +%Y%m%d-%H%M).db
cp -r /var/log/hyper-tracker /backups/logs-$(date +%Y%m%d-%H%M)

# 3. Create fresh database
hyper-tracker init --force  # will create empty DB

# 4. Replay events from audit log and ingestion logs
hyper-tracker rebuild \
  --from-audit-log /var/log/hyper-tracker/audit.log \
  --from-ingest-logs /var/log/hyper-tracker/ingest.log.* \
  --dry-run

# Dry-run shows:
# Would recreate: 45,231 events
# Would recreate: 1,248 assets
# Would recreate: 4,532 IOCs
# Estimated time: 45 minutes

hyper-tracker rebuild \
  --from-audit-log /var/log/hyper-tracker/audit.log \
  --from-ingest-logs /var/log/hyper-tracker/ingest.log.* \
  --skip-duplicates
```

Rebuild process:
1. Parse audit log for `action: create` on events, assets, iocs
2. Reconstruct original payloads (from `details` field or embedded)
3. Insert in chronological order respecting foreign keys
4. Rebuild AI analysis cache (will re-run AI if models available)
5. Recreate alerts and webhooks from config

### 10. Rollback specific table only

```bash