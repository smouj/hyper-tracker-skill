---
title: Hyper Tracker
description: Sistema de seguimiento de tareas de seguridad impulsado por IA para gestión de vulnerabilidades, inteligencia de amenazas y respuesta a incidentes
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

Sistema de seguimiento de seguridad mejorado con IA que ingiere, categoriza, prioriza y rastrea automáticamente tareas, eventos e incidentes relacionados con seguridad, con inteligencia de amenazas integrada y análisis predictivo.

## Propósito

Hyper Tracker sirve como sistema nervioso central para operaciones de seguridad:

1. **Gestión de Vulnerabilidades**: Categorización automática de CVEs, asignación de puntuaciones de riesgo mediante análisis IA de disponibilidad de exploits, métricas CVSS, puntuaciones EPSS y criticidad de activos. Ejemplo: CVE-2024-3094 recibe puntuación de riesgo 92/100 porque la IA detectó un exploit armamentizado en foros de la dark web 3 días antes de la divulgación pública.

2. **Seguimiento de Inteligencia de Amenazas**: Rastrea IOCs, TTPs y comportamientos de adversarios con enriquecimiento automatizado desde VirusTotal, AlienVault OTX y feeds comerciales. Correlaciona automáticamente nuevos IOCs con incidentes existentes usando similitud de embeddings.

3. **Coordinación de Respuesta a Incidentes**: Seguimiento de IR basado en líneas de tiempo con asignación automática de tareas, preservación de cadena de evidencia y monitoreo de SLA. Ejemplo: Cuando se dispara una alerta de ransomware, Hyper Tracker crea automáticamente el incidente, lo asigna al equipo tier-2 e incia la cuenta regresiva de SLA de contención de 1 hora.

4. **Mapeo de Cumplimiento**: Mapea automáticamente eventos a controles (NIST 800-53, ISO 27001, PCI-DSS). Ejemplo: Cada evento "access revoked" se etiqueta automáticamente como evidencia de PCI-DSS 8.1.4 con fecha de retención.

5. **Análisis Predictivo**: Usa LSTM en datos históricos de incidentes para predecir:
   - Qué vulnerabilidades sin parchear serán explotadas a continuación (95% de precisión para CVEs críticos)
   - Probables próximos movimientos del atacante basados en progresión MITRE ATT&CK
   - Riesgo de agotamiento del analista basado en patrones de carga de trabajo

6. **Ticketing Automatizado**: Sincronización bidireccional con JIRA, ServiceNow, GitHub Issues. Comentarios en ticket actualizan Hyper Tracker; análisis actualizaciones se envían al ticket via webhook.

Caso de uso real:
> El analista del SOC recibe 500 eventos diarios. Hyper Tracker los reduce a 15 elementos de alto riesgo mediante triaje IA. El analista investiga el incidente INC-2024-0892 (credencial stuffing sospechoso). Hyper Tracker muestra:
> - IOCs relacionados: 47 IPs coinciden con campaña "Proxy Bot" (confianza 94%)
> - Predictivo: 87% de probabilidad de que el atacante pivote a SSO a continuación
> - Respuesta recomendada: bloquear IPs en WAF, habilitar MFA para 3 usuarios de alto riesgo, revisar logs de SSO
> - Ticket JIRA creado automáticamente con toda la evidencia adjunta
> - Temporizador de SLA iniciado: contención debida en 2 horas (transcurridas: 23 minutos)

## Alcance

Hyper Tracker proporciona comandos de producción para equipos de seguridad:

### Operaciones Principales de Eventos

```
hyper-tracker add \
  --type <vulnerability|incident|ioc|compliance|task|threat> \
  --source <nmap|qualys|nessus|openvas|siem|manual|crowdstrike|sentinel|guardduty|custom> \
  --severity <critical|high|medium|low|info|unknown> \
  [--confidence 0.0-1.0] \
  --title \"Breve descripción (máx 120 caracteres)\" \
  --details \"Descripción extendida con especificaciones técnicas\" \
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
  [--query \"búsqueda de texto completo\"] \
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

Ejemplos:
```bash
# Encontrar vulnerabilidades críticas en activos de producción con exploit disponible
hyper-tracker search \
  --type vulnerability \
  --severity critical \
  --asset-tag environment:production \
  --ai-confidence-min 0.8 \
  --format table

# Encontrar incidentes relacionados con IOC específico
hyper-tracker search \
  --ioc \"185.220.101.77\" \
  --status \"!resolved\" \
  --sort created \
  --limit 20

# Exportar todos los eventos abiertos relevantes para PCI
hyper-tracker search \
  --tag pci-dss \
  --status \"!closed\" \
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
  [--add-comment \"Texto de comentario (soporta markdown)\"] \
  [--add-evidence /path/to/file] \
  [--add-ioc IOC_VALUE] \
  [--remove-ioc IOC_VALUE] \
  [--add-tag TAG] \
  [--remove-tag TAG] \
  [--resolution \"Cómo se resolvió\"] \
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

### IA y Enriquecimiento

```
hyper-tracker analyze \
  <event-id> \
  [--re-run] \
  [--model <vulnerability|incident|ioc|ttp|all>] \
  [--wait] \
  [--timeout SECONDS]
```

Dispara enriquecimiento IA asíncrono. Con `--wait`, bloquea hasta completar y muestra resultados.

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

Usa IA para predecir progresión de ataque probable. Salida incluye probabilidad, línea de tiempo, defensas recomendadas.

```
hyper-tracker recommend \
  <event-id> \
  [--type <containment|remediation|investigation>] \
  [--limit N]
```

Genera recomendaciones paso a paso de playbook.

### Alertas y Automatización

```
hyper-tracker alert add \
  --name \"Nombre de alerta\" \
  --condition 'expresión json-logic' \
  --channel <slack|email|pagerduty|teams|webhook|splunk> \
  --recipient DESTINO \
  [--template NOMBRE_PLANTILLA] \
  [--throttle < per_minute|per_hour|daily >] \
  [--cooldown MINutos] \
  [--severity-threshold <critical|high|medium>]
```

Ejemplos de condición:
```json
--condition '{"and": [{"==": [{"var": "type"}, "vulnerability"]}, {">": [{"var": "risk_score"}, 80]}, {"==": [{"var": "asset.environment"}, "production"]}]}'
```

```
hyper-tracker webhook add \
  <nombre> \
  --url https://servicio/hook \
  --secret SECRETO_COMPARTIDO \
  --events <incident.created|vulnerability.updated|ioc.matched|alert.fired> \
  [--format json] \
  [--retry N] \
  [--timeout SECONDS]
```

```
hyper-tracker rule create \
  --name \"Asignar automáticamente vulnerabilidades SSH\" \
  --trigger \"vulnerability.created && cve && 'CVE-2024-XXXX' in tags\" \
  --action \"assign = 'ssh-team@company.com'; add_tag = 'ssh-team-pending'\" 
```

```
hyper-tracker schedule add \
  --name \"Resumen diario de amenazas\" \
  --cron \"0 8 * * *\" \
  --command 'report daily-briefing --format html --email security-team@company.com'
```

### Reportes y Exportación

```
hyper-tracker report \
  <tipo-reporte> \
  [--output ARCHIVO] \
  [--format <html|pdf|json|csv|jsonl|markdown>] \
  [--date-range <LAST_7_DAYS|LAST_30_DAYS|CUSTOM>] \
  [--date-start YYYY-MM-DD] \
  [--date-end YYYY-MM-DD] \
  [--filters FILTROS_JSON] \
  [--template NOMBRE_PLANTILLA] \
  [--email RECIPIENTES]
```

Tipos de reporte:
- `daily-briefing`: Resumen matutino del SOC con eventos nocturnos
- `vulnerability-trends`: Gráficos de vulnerabilidades nuevas vs corregidas por severidad
- `incident-summary`: Todos los incidentes abiertos con estado de SLA
- `compliance-gap`: Mapeo a frameworks con brechas
- `mttr-mttd`: Tiempo medio de respuesta/detección con tendencias
- `threat-landscape`: Principales IOCs, campañas, TTPs
- `asset-risk-heatmap`: Inventario de activos coloreado por riesgo
- `ai-performance`: Precisión del modelo, tasas de falsos positivos

```
hyper-tracker export \
  [--table <events|assets|iocs|incidents|alerts|audit>] \
  [--output ARCHIVO] \
  [--format <json|csv|parquet|feather|sqlite>] \
  [--compression <gzip|bzip2|zstd>] \
  [--filters FILTROS_JSON] \
  [--fields campo1,campo2,...] \
  [--partition-by date]
```

```
hyper-tracker import \
  --source ARCHIVO \
  --format <json|csv|nessus|qualys|jira|servicenow|csv> \
  [--map-fields CONFIG_JSON] \
  [--dry-run] \
  [--skip-duplicates] \
  [--update-existing] \
  [--batch-size N]
```

### Gestión de Activos e IOCs

```
hyper-tracker asset add \
  --asset-id ID_ÚNICO \
  --hostname FQDN \
  --ip DIRECCIÓN_IP \
  [--mac MAC] \
  [--os NOMBRE_SISTEMA_OPERATIVO] \
  [--criticality <low|medium|high|critical>] \
  [--environment <dev|staging|production|lab>] \
  [--owner EMAIL] \
  [--team NOMBRE_EQUIPO] \
  [--tags etiqueta1,etiqueta2,...] \
  [--cmdb-id ID_EXTERNO]
```

```
hyper-tracker asset tag \
  <asset-id> \
  add ETIQUETA[:VALOR] \
  [--expire FECHA]
```

```
hyper-tracker ioc add \
  --value VALOR_IOC \
  --type <ip|domain|url|hash|email|user-agent|custom> \
  [--source nombre_fuente] \
  [--confidence 0.0-1.0] \
  [--threat-type <malware|c2|phishing|botnet|apt>] \
  [--campaign NOMBRE_CAMPAÑA] \
  [--first-seen FECHA] \
  [--last-seen FECHA] \
  [--feed-id ID_EXTERNO]
```

```
hyper-tracker ioc check \
  <valor> \
  [--sources all] \
  [--format full]
```

### Administración del Sistema

```
hyper-tracker init \
  [--config ARCHIVO_CONFIG] \
  [--db-path RUTA] \
  [--ai-model-dir RUTA] \
  [--log-level <DEBUG|INFO|WARNING|ERROR>] \
  [--skip-samples] \
  [--force]
```

```
hyper-tracker config \
  set <clave> <valor> \
  [--scope <global|ai|alerts|integrations|database|dashboard>] \
  [--file ARCHIVO_CONFIG]

hyper-tracker config \
  get <clave> \
  [--scope ALCANCE]

hyper-tracker config \
  list \
  [--scope ALCANCE] \
  [--show-secrets]
```

```
hyper-tracker status
```

Muestra salud completa del sistema:
```
System Status: HEALTHY
Database: /var/lib/hyper-tracker/data.db (12.4GB, 45,231 eventos)
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
  <nombre-modelo> \
  --source <huggingface|s3|local|url> \
  --model-id HF_MODEL_ID \
  [--api-token TOKEN] \
  [--cache-dir RUTA]

hyper-tracker model train \
  <nombre-modelo> \
  [--dataset RUTA] \
  [--epochs N] \
  [--validation-split 0.0-1.0] \
  [--learning-rate 0.0001] \
  [--batch-size N] \
  [--backup-before] \
  [--dry-run]

hyper-tracker model evaluate \
  <nombre-modelo> \
  [--test-set RUTA] \
  [--metrics precision|recall|f1|accuracy|all]
```

```
hyper-tracker backup \
  [--output ARCHIVO] \
  [--encrypt] \
  [--compress] \
  [--include-logs] \
  [--include-models]

hyper-tracker restore \
  --backup ARCHIVO \
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
  [--date-start FECHA] \
  [--date-end FECHA] \
  [--format json|table]
```

Muestra traza de auditoría inmutable de todos los cambios del sistema.

```
hyper-tracker healthcheck
```
Código de salida 0 si está saludable, no cero si hay problemas. Adecuado para sistemas de monitoreo.

```
hyper-tracker maintenance \
  vacuum \
  [--db-path DB] \
  [--aggressive]

hyper-tracker maintenance \
  archive \
  [--older-than DÍAS] \
  [--dry-run] \
  [--verify]

hyper-tracker maintenance \
  reindex \
  [--table eventos] \
  [--concurrently]
```

## Proceso de Trabajo Detallado

### 1. Flujo de Inicialización

```bash
hyper-tracker init --config ./hyper-tracker.yaml --db-path /var/lib/hyper-tracker/data.db
```

Ejemplo `hyper-tracker.yaml`:
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

El proceso init crea:
```
/home/user/.config/hyper-tracker/config.yaml
/var/lib/hyper-tracker/data.db (SQLite con WAL)
/var/lib/hyper-tracker/models/
├── vulnerability/
│   ├── config.json
│   ├── pytorch_model.bin
│   └── tokenizer/
├── incident/
│   └── ...
/var/log/hyper-tracker/
├── hyper-tracker.log (líneas JSON)
├── ingest.log
├── ai.log
└── audit.log
/var/lib/hyper-tracker/backups/
```

### 2. Pipeline de Ingestión en Profundidad

**Integración con Qualys**:
```bash
# Se ejecuta cada 6 horas via cron
hyper-tracker ingest run --source qualys --since-last
```

Proceso:
1. Llamada API a Qualys: `GET /api/2.0/fo/vuln/`
2. Parsear 10,000+ vulnerabilidades por escaneo
3. Para cada vulnerabilidad:
   - Desduplicar: título hash + CVE + ID de activo
   - Búsqueda de activo: mapear hostname QID a asset-id interno via tabla de sincronización CMDB
   - Puntuación de riesgo IA: cargar modelo de vulnerabilidad, codificar descripción CVE + contexto de activo, predecir probabilidad de exploit
   - Generar puntuación de riesgo: `base_severity * 0.3 + ai_exploit_score * 0.5 + asset_criticality * 0.2`
   - Crear o actualizar evento
4. Insertar en lote 1000 eventos por transacción DB
5. Disparar alertas para risk_score > 80
6. Crear tickets JIRA automáticamente si está configurado
7. Registrar métricas de ingestión: `{ "source": "qualys", "total": 12453, "new": 2341, "updated": 10112, "duration_sec": 342 }`

**Webhook SIEM** (tiempo real):
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

Procesamiento:
1. Verificar token
2. Parsear array JSON (hasta 1000 eventos por solicitud)
3. Cada evento:
   - Tipo: mapear regla SIEM a tipo Hyper Tracker
   - Extracción de IOC: "source_ip" → añadir a tabla IOC si no existe
   - Normalización de activo: "asset" → búsqueda asset-id (coincidencia difusa)
   - Análisis IA: si authentication_failure > 10/min misma IP+usuario → crear incidente automáticamente
   - Correlación: verificar si IP coincide con IOC existente → vincular evento a incidente
4. Respuesta: `{"processed": 45, "created_incidents": 2, "alerts_fired": 1}`

### 3. Enriquecimiento IA en Profundidad

**Modelo de vulnerabilidad** (CVE BERT fine-tuneado):
```python
# Arquitectura simplificada
input: [CLS] + tokens de descripción CVE + [SEP] + criticidad de activo + entorno + nivel de exposición
output: 
  - exploit_probability (sigmoid)
  - weaponized (sigmoid > 0.7)
  - adjusted_severity (softmax sobre 5 clases)
  - recommended_actions (generación de secuencia con cabeza BART)

Datos de entrenamiento: 50,000 CVEs con ground truth:
  - ¿NVD incluye referencia de exploit?
  - ¿Hubo módulo Metasploit dentro de 30 días?
  - ¿Los clientes fueron explotados (telemetría de EDR)?
  - ¿Cuál fue la tasa real de cumplimiento de parches?
```

Ejemplo de salida de análisis IA:
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

**Modelo de clasificación de incidentes**:
- Input: título de incidente + descripción + IOCs + línea de tiempo
- Output: tipo de incidente (data_exfiltration, ransomware, account_takeover, malware, phishing, insider_threat, other)
- También predice perfil de atacante probable (script_kiddie, cybercrime, apt, insider)

**Enriquecimiento de IOC**:
```bash
hyper-tracker enrich-ioc 185.220.101.77 --sources all
```

Salida:
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

### 4. Reglas de Alertas en Práctica

**Regla 1: Vulnerabilidad prod crítica con exploit armamentizado**
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

Se dispara para CVE-2024-3094 en web-prod-01:
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

**Regla 2: Nuevo IOC coincide con campaña**
```bash
hyper-tracker alert add \
  --name "IOC Campaign Match" \
  --condition '{"and": [{"==": [{"var": "type"}, "ioc"]}, {">": [{"var": "campaign_association.confidence"}, 0.8]}]}' \
  --channel email \
  --recipient threat-intel@company.com \
  --template "ioc-campaign" \
  --throttle daily
```

Plantilla de email:
```
Subject: [Hyper Tracker] Nuevo IOC vinculado a campaña: {{campaign_association.campaign}}

Nuevo indicador detectado: {{ioc_value}} ({{type}})
Campaña: {{campaign_association.campaign}} ({{campaign_association.confidence * 100}}% confianza)
First seen: {{first_seen}}
Threat type: {{threat_type}}
Enrichment: {{enrichments.virustotal.detection_ratio}} detecciones en VT
Action: Revisar incidentes relacionados y considerar bloqueo proactivo.

[Ver detalles]({{dashboard_url}})
```

### 5. Ejemplos de Generación de Reportes

**Resumen Diario** (`hyper-tracker report daily-briefing`):

```
SECURITY DAILY BRIEFING - March 15, 2024

SUMMARY
--------
Nuevos eventos: 47 (↑12% vs ayer)
Incidentes abiertos: 23 (↑3)
Vulnerabilidades críticas: 5 (↓2)
Cumplimiento SLA: 94.2% (P0: 100%, P1: 96%, P2: 91%)

TOP PRIORITY ITEMS
------------------
1. [P0] INC-2024-0892 - Actividad ransomware en finance-db-01
   Creado: 14:23 | Propietario: jsmith@ | SLA: 37 min restantes
   Predicción IA: 87% probabilidad de exfiltración de datos en 4h
   Action: Aislar host, preservar memoria, notificar legal

2. [P0] evt_abc123 - CVE-2024-3094 (Apache Struts) en web-prod-01
   Risk score: 94 | Exploit: weaponized | Asset: Production
   Action: Parchear en 2h o poner offline

3. [P1] INC-2024-0890 - Acceso privilegiado sospechoso
   Creado: 09:15 | Propietario: mjones@ | SLA: 4h 45m restantes
   Usuario: support-admin - Acceso desde ubicación inusual (Rusia)
   Action: Entrevistar usuario, revisar logs de autenticación

OVERNIGHT INCIDENTS
-------------------
• INC-2024-0885 - Fuerza bruta en SSH gateway ( bloqueado por fail2ban)
  IOCs: 185.220.101.77, 185.220.101.78
  IA: Vinculado a campaña "Cobalt Spider" (94% confianza)
  Estado: Resuelto a las 03:45

• INC-2024-0888 - Falso positivo de alerta DLP (archivo de prueba)
  Estado: Cerrado como falso positivo

THREAT INTELLIGENCE
-------------------
Nuevos IOCs de alta confianza añadidos: 12
├─ 8 IPs (C2 Cobalt Spider)
├─ 3 dominios (phishing)
└─ 1 SHA256 (variante Emotet)

Actividad de campaña: "Cobalt Spider" sigue activo (32 incidentes YTD)
Más objetivo: Finance (12), Engineering (8), HR (5)

VULNERABILITY TRENDS
--------------------
Nuevos CVEs añadidos: 34
Alto/Nuevo: 8 | Medio: 19 | Bajo: 7

Top 3 críticos:
CVE-2024-3094 (Apache Struts) - 5 activos afectados, riesgo: 94
CVE-2024-3004 (Fortinet) - 2 activos, riesgo: 91
CVE-2024-2987 (Microsoft) - 1 activo, riesgo: 88

Cumplimiento de parches (últimos 30 días):
Critical: 91% (↓3%) | High: 87% | Medium: 94%

UPCOMING
--------
• Prueba externa PCI-DSS debido: 15 de abril (60 días)
• Revisión de acceso trimestral comienza el lunes (800 usuarios)
• Mantenimiento del sistema: 22 de marzo, 02:00-06:00 UTC

---
Dashboard: http://hyper-tracker:8080
Questions? #security-team
```

**Reporte de Brecha de Cumplimiento** (`hyper-tracker report compliance-gap --framework NIST-800-53`):

```
NIST 800-53 Rev. 5 Análisis de Brecha de Cumplimiento
As of: 15 de marzo de 2024
Periodo: Q1 2024 (1 ene - 31 mar)

CUMPLIMIENTO GENERAL: 89/100

FAMILIA DE CONTROL: ACCESS CONTROL (AC)
-----------------------------------
AC-2 (Gestión de Cuentas): COMPLIANT
  ✓ Todas las cuentas de usuario inventariadas
  ✓ Cuentas inactivas deshabilitadas (>90 días)
  ✓ Cuentas privilegiadas auditadas trimestralmente
  
AC-3 (Aplicación de Acceso): BRECHA MENOR (87/100)
  ⚠ 3 roles con privilegios excesivos identificados
  ⚠ No hay evidencia de revisión de principio de mínimo privilegio para 2 aplicaciones
  
AC-6 (Principio de Mínimo Privilegio): BRECHA (72/100)
  ✗ 15 cuentas de servicio con derechos de admin (debería ser 0)
  ✗ No hay proceso documentado de revisión regular de escalada de privilegios
  Elementos de acción:
    1. Revisar todas las cuentas admin antes del 29 de marzo
    2. Implementar revisión trimestral de privilegios (debido 15 abr)
    3. Implementar solución PAM (proyecto: Q2 2024)

FAMILIA DE CONTROL: INCIDENT RESPONSE (IR)
--------------------------------------
IR-4 (Manejo de Incidentes): COMPLIANT (95/100)
  ✓ Todos los incidentes de seguridad rastreados en Hyper Tracker
  ✓ Existen playbooks de respuesta para los 10 tipos de incidente principales
  ✓ Ejercicios realizados trimestralmente
  
IR-5 (Monitoreo de Incidentes): BRECHA MENOR (88/100)
  ⚠ Monitoreo 24/7 solo para producción (brechas en dev/staging)
  Action: Extender cobertura de monitoreo antes del 30 abr

FAMILIA DE CONTROL: RISK ASSESSMENT (RA)
------------------------------------
RA-3 (Evaluación de Riesgos): COMPLIANT
  ✓ Evaluación de riesgos anual completada
  ✓ Registro de riesgos mantenido en herramienta GRC
  ✓ Planes de tratamiento de riesgo rastreados

TENDENCIA DE CUMPLIMIENTO
-----------------
Q4 2023: 86/100
Q1 2024: 89/100 ↑3 puntos
Proyectado Q2 2024: 91/100 (si se abren brechas abiertas)

PRIORIDAD BASADA EN RIESGO (por impacto × probabilidad)
---------------------------------------------
1. Brecha AC-6 Mínimo Privilegio (Riesgo: 8.4/10) → SUPERFICIE DE ATAQUE: ALTA
2. Brecha IR-5 Monitoreo (Riesgo: 6.2/10) → VACÍO DE DETECCIÓN: ALTO
3. CA-7 Monitoreo Continuo (Riesgo: 5.1/10) → MEDIO

---
Paquete de evidencia completo: /var/reports/compliance/nist80053-q1-2024/
Recopilación de evidencia automatizada via etiquetas Hyper Tracker.
```

### 6. Ejemplo de Análisis Predictivo

**Predicción de próximos movimientos del atacante**:
```bash
hyper-tracker predict inc_9a2b3c4d --scenario lateral-movement
```

Salida:
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
      "trigger": " attacker ya tiene credenciales de admin por brecha inicial",
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
  "training_data": "45 incidentes similares de 2023-2024",
  "last_updated": "2024-03-10"
}
```

### 7. Operaciones en Lote

**Importar 5000 eventos heredados**:
```bash
hyper-tracker import \
  --source ./legacy-incidents-2023.csv \
  --format csv \
  --map-fields '{"type": "incident_type", "severity": "priority", "title": "subject", "details": "description", "create_date": "opened_at", "close_date": "resolved_at", "assignee": "owner"}' \
  --dry-run
```

Salida dry-run:
```
DRY RUN - No se harán cambios
Source: ./legacy-incidents-2023.csv (5,231 filas)
Mapping: 7/8 campos mapeados exitosamente
├─ incidents: 5,231 (estimado)
├─ nuevos activos descubiertos: 156
├─ detección duplicados: 342 eventos ya existen (se saltarían)
└─ errores de validación: 0

Mapping de muestra:
  severidad heredada "P1" → severidad Hyper Tracker "high"
  estado heredado "Closed" → estado Hyper Tracker "resolved"
  
Se crearían:
  - 4,889 incidentes
  - 156 nuevos activos
  - Se saltarían 342 duplicados
Duración estimada: 8-12 minutos
```

Ejecución real con procesamiento por lotes:
```bash
hyper-tracker import \
  --source ./legacy-incidents-2023.csv \
  --format csv \
  --map-fields ./mapping.json \
  --batch-size 500 \
  --skip-duplicates \
  --update-existing
```

Progreso: `[====>                  ] 45% (2341/5231) - 12m transcurridos, 15m restantes`

Finalización:
```
✓ Importación completada exitosamente
  • Eventos creados: 4,889
  • Eventos actualizados: 342
  • Nuevos activos añadidos: 156
  • Duplicados omitidos: 342
  • Errores de validación: 0
  • Duración: 14m 23s
  
Post-import: Enriquecimiento IA en cola para 4,889 eventos
  (completación estimada: 2 horas)
  
Reporte generado: /tmp/import-summary-20240315.html
```

**Enriquecimiento IA automático procesando**:

```bash
# Verificar estado del trabajo IA
hyper-tracker model status
```

Salida:
```
AI Workers: 10 activos
Queue:
  • eventos de vulnerabilidad: 4,231 (est. 1h 15m)
  • eventos de incidente: 4,382 (est. 2h 30m)
  
Procesando actualmente: modelo de vulnerabilidad (lote 432/4231)
Último lote completado: 431 eventos en 89s (4.8 eventos/seg)

Completación esperada:
  • Todos eventos enriquecidos por: 2024-03-15 17:30 UTC
```

Después del enriquecimiento, el analista revisa una muestra:

```bash
hyper-tracker search --tag auto-created-import-20240315 --limit 5 --format table
```

Salida tabla:
```
ID              TYPE      SEVERITY  TITLE                                    ASSETS  AI_RISK  STATUS
evt_x1y2z3a     incident  high      Suspicious outbound traffic detected   2       78       open
evt_b4c5d6e     vuln      medium    Apache Log4j CVE-2021-44228            1       65       resolved
evt_f7g8h9i     incident  critical  Ransomware on file-server-01           1       96       closed
evt_j0k1l2m     vuln      unknown   WordPress plugin XSS                  1       45       open
evt_n3o4p5q     incident  low       Phishing email reported                0       22       closed
```

**El analista añade contexto faltante a eventos heredados**:

```bash
# Añadir técnica MITRE ATT&CK a todos los incidentes de phishing de la importación
hyper-tracker search --tag auto-created-import-20240315 --type incident --query "phishing" --format json \
  | jq -r '.[] | .event_id' \
  | xargs -I {} hyper-tracker update {} --add-tag technique:T1566.002
```

### 8. Uso del Dashboard

Dashboard en `http://hyper-tracker:8080` (auth básica por usuario):

**Pestaña: Overview**
- Tarjetas KPI: Incidentes abiertos, vulnerabilidades críticas, MTTR, nuevos eventos 24h
- Gráficos de tendencia: eventos por severidad (7 días), cumplimiento SLA
- Heatmap: riesgo por grupo de activos

**Pestaña: Events**
- Tabla searchable con todos los campos
- Ediciones en línea: arrastrar asignatario, dropdown de estado
- Acciones en lote: cambiar estado, añadir etiquetas, re-ejecutar análisis IA
- Botón exportar (CSV/JSON)

**Pestaña: Assets**
- Vista árbol: entorno → equipo → activos
- Modal de detalles de activo: vulnerabilidades, incidentes, IOCs, tendencia de riesgo
- Editar activo: criticidad, propietario, etiquetas

**Pestaña: IOCs**
- Tabla de watchlist: todos los IOCs conocidos con último visto, tipo de amenaza
- Historial de coincidencias: qué eventos coincidieron con este IOC
- Panel de enriquecimiento: datos VT/OTX en línea

**Pestaña: Analytics**
- MTTR/MITTD por severidad, tipo de activo, equipo
- Matriz de confusión IA (severidad actual vs predicha)
- Tasa de falsos positivos por analista
- Técnicas de ataque principales (mapeo MITRE)

**Pestaña: System**
- Profundidades de cola, latencia de ingestión
- Rendimiento de modelos IA
- Uso de almacenamiento, estado de backup

## Reglas de Oro

1. **Nunca Cerrar Automáticamente Sin Verificación**: Los eventos solo pueden marcarse `resolved` con campo `resolution` obligatorio. La IA puede sugerir cierre pero el humano debe confirmar. Para vulnerabilidades: requiere evidencia de `patched_version` o `mitigation_applied`. Para incidentes: requiere `postmortem_link` o `lessons_learned_attached`.

2. **Audit Trail Inmutable**: Todas las actualizaciones de eventos se añaden a `audit_log` - nunca UPDATE o DELETE. Mostrar cambios como diffs. Retener log de auditoría 7 años para cumplimiento. La única operación delete es `hyper-tracker rollback --event-id X --reason "duplicate"` dentro de 5 minutos de creación, que registra eliminación en auditoría.

3. **Umbrales de Confianza IA**: Nunca actuar en recomendaciones IA con confianza < 0.7 sin revisión de analista. Mostrar confianza prominentemente en UI y CLI. Decisiones de alto riesgo (incidente P0) requieren campo de anulación del analista si se desvía de IA.

4. **Minimización de Datos para IOCs**: No almacenar PII en bruto en detalles de IOC. Hash direcciones de correo, enmascarar IPs en logs internos (mostrar solo último /24). Cifrar base de datos en reposo si contiene datos EU/PHI.

5. **Limitación de Tasa y Throttling**: Todas las llamadas API externas (VirusTotal, JIRA) deben respetar límites de tasa. Hyper Tracker por defecto: VT 4 req/min, JIRA 100 req/min. Implementar retroceso exponencial. Monitoreado via `hyper-tracker status`.

6. **Pinning de Versión de Modelo**: Nunca auto-actualizar modelos IA. Pinear versiones específicas de modelo en config:
   ```
   ai:
     models:
       vulnerability: microsoft/cvebert-1.0
       incident: custom-incident-v3.1
   ```
   Probar nuevos modelos en staging antes de despliegue en producción.

7. **Integridad de Cadena de Evidencia**: Todos los archivos de evidencia almacenados con SHA256 checksum registrado. Prevenir manipulación: directorio de evidencia es solo-append (chattr +a). Modificaciones de archivos registradas en auditoría.

8. **Aplicación de SLA**:
   - Incidentes P0: reconocer en 15m, contener en 1h, resolver en 4h
   - P1: reconocer 1h, contener 4h, resolver 24h
   - P2: reconocer 4h, contener 24h, resolver 7d
   Temporizadores SLA inician en creación, se pausan solo con `status: investigating` (requiere comentario explicando demora). Incumplimientos de SLA disparan alerta inmediata a gestión.

9. **Separación de Funciones**: Rol `admin` de Hyper Tracker no puede ser también `analyst`. Admin gestiona config del sistema, modelos, integraciones. Analyst crea/actualiza eventos, ejecuta análisis IA. Rol dual prohibido en producción.

10. **Backup Antes de Operaciones Destructivas**: Siempre ejecutar `hyper-tracker backup` antes de:
    - Importaciones/actualizaciones en lote
    - Reentrenamiento de modelo (que puede sobrescribir modelo de producción)
    - Mantenimiento de base de datos (VACUUM, ARCHIVE)
    - Cambios de configuración con `--scope global`

## Ejemplos con Entradas/Salidas Reales

### Ejemplo 1: Ingestión de vulnerabilidad y análisis IA

**Entrada** (adición manual de nuevo CVE):
```bash
hyper-tracker add \
  --type vulnerability \
  --source manual \
  --severity unknown \
  --title "OpenSSH CVE-2024-3094 remote code execution" \
  --details "Una vulnerabilidad en ssh-keygen de OpenSSH permite escribir más allá del buffer asignado al procesar ciertas claves de certificado. Atacantes pueden craft claves maliciosas para lograr RCE en sistemas que validan claves." \
  --cve CVE-2024-3094 \
  --cvss "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H" \
  --asset-ids web-prod-01,web-prod-02,web-staging-03 \
  --tags cve2024,openssh,remote-code-execution
```

**Salida**:
```
✓ Evento creado: evt_f3a1b9c2
Activos: 3 encontrados (2 producción, 1 staging)
Análisis IA en cola (ID de trabajo: ai_9z8y7x6w)
Se notificará cuando completo (normalmente 30-90 segundos)
Ejecutar: hyper-tracker show evt_f3a1b9c2 --ai-analysis para ver resultados
```

**Después** (después de procesamiento IA):
```bash
hyper-tracker show evt_f3a1b9c2 --full
```

Salida (campos clave):
```json
{
  "event_id": "evt_f3a1b9c2",
  "type": "vulnerability",
  "cve": "CVE-2024-3094",
  "title": "OpenSSH CVE-2024-3094 remote code execution",
  "severity": "high",  // IA ajustada de 'unknown'
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
      "adjusted_severity": "critical",  // Confianza IA: 0.93
      "exploit_prediction": {
        "probability": 0.96,
        "public_exploit_exists": true,
        "weaponized": true,
        "expected_exploit_timeframe": "0-3 days (explotación activa observada en foros de dark web)"
      },
      "recommendations": [
        "INMEDIATO: Parchear todas las instancias OpenSSH a 9.4p1 o posterior. Esta vulnerabilidad está siendo explotada activamente.",
        "Considerar mitigación temporal: deshabilitar validación de certificados ssh-keygen si no requerida (no recomendado para producción)",
        "Monitorear patrones de autenticación sospechosos",
        "Revisar logs para evidencia de intentos de explotación"
      ],
      "related_cves": [
        {"cve": "CVE-2023-38408", "similarity": 0.89, "notes": "Vulnerabilidad de parsing SSH similar"},
        {"cve": "CVE-2024-22001", "similarity": 0.85, "notes": "Vulnerabilidad SSH reciente, componente diferente"}
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
    {"timestamp": "2024-03-15T14:32:00Z", "user": "analyst@company.com", "action": "create", "details": "Evento creado desde CLI"},
    {"timestamp": "2024-03-15T14:35:12Z", "user": "system", "action": "ai_enrichment", "details": "Análisis IA completado, risk_score=94"},
    {"timestamp": "2024-03-15T14:35:13Z", "user": "system", "action": "alert_fired", "details": "Alerta 'Critical prod vuln - weaponized' a slack"},
    {"timestamp": "2024-03-15T14:35:15Z", "user": "system", "action": "ticket_created", "details": "JIRA SEC-12456 creado"}
  ]
}
```

**Ticket JIRA creado automáticamente**:
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
  • Weaponized Exploit: YES (explotación activa observada en wild)
  • Exploit Probability: 96% within next 3 days
  • Model Confidence: 93%
  • Recommended Action: PATCH IMMEDIATELY

Description:
OpenSSH's ssh-keygen contains a buffer overflow vulnerability when processing specially crafted certificate keys. Remote attackers can achieve code execution on systems that validate attacker-controlled keys (typical in authentication workflows).

Affected Versions:
  • OpenSSH < 9.4p1
  • OpenSSH 9.4p1 and later are not vulnerable

Recommendations (from AI):
1. INMEDIATE: Patch to OpenSSH 9.4p1 or later on web-prod-01 and web-prod-02 within 24 hours.
2. Considerar mitigación temporal si parcheo retrasado: deshabilitar validación de certificados (no recomendado).
3. Monitorear logs de autenticación para intentos de explotación.
4. web-staging-03 puede esperar hasta próxima ventana de mantenimiento (críticidad inferior).

Patch Window:
  Servidores de producción: Disponibles domingos 02:00-06:00 UTC. Para emergencia, override disponible con aprobación de manager.

Asset Owners:
  web-prod team: web-team@company.com
  Notified: Yes (auto-asignado)

Evidence:
  • CVE description: https://nvd.nist.gov/vuln/detail/CVE-2024-3094
  • Metasploit module: https://github.com/rapid7/metasploit-framework/pull/12345
  • AI analysis: https://hyper-tracker:8080/events/evt_f3a1b9c2?view=ai

---
[Auto-generated by Hyper Tracker]
[Track: evt_f3a1b9c2]
```

**Alerta Slack recibida en #sec-critical**:
```
🚨 CRITICAL VULNERABILITY - WEAPONIZED

CVE: CVE-2024-3094
Asset: web-prod-01 (10.10.1.101) • Production • Criticality: HIGH
CVSS: 9.8 • AI Risk: 94/100 • Weaponized: YES (active exploitation)
Title: OpenSSH ssh-keygen buffer overflow RCE
AI Confidence: 93%

Recommendation: INMEDIATE PATCH REQUIRED - Exploit observed in wild

[JIRA Ticket](https://jira.company.com/browse/SEC-12456) | [Dashboard](http://hyper-tracker:8080/events/evt_f3a1b9c2)

Created: 2 minutes ago
[@here] @security-team @web-team
```

**Flujo de trabajo del analista**:

```bash
# 1. Reconocer asignándose a sí mismo y añadiendo comentario
hyper-tracker update evt_f3a1b9c2 \
  --assignee senior-analyst@company.com \
  --add-comment "Acknowledged. Will coordinate patch with web-team. Checking if any exploit attempts observed in our logs."

# 2. Buscar coincidencias de IOC relacionadas
hyper-tracker search --ioc "185.220.101.77" --limit 10
# (muestra 2 incidentes de campaña Cobalt Spider)

# 3. Ejecutar predicción IA para evaluar probabilidad de explotación
hyper-tracker predict evt_f3a1b9c2 --scenario exploitation
# Devuelve: "exploit_probability: 0.96 (weaponized)"

# 4. Verificar si algún activo ya comprometido
hyper-tracker search --asset-id web-prod-01 --status "!resolved" --type incident
# (retorna 0 - bien)

# 5. Cambiar estado SLA (de P0 a investigating, pausa temporizador SLA)
hyper-tracker update evt_f3a1b9c2 --status investigating --add-comment "Investigation: no evidence of exploitation yet. Log review in progress."

# 6. Después de aplicar parche
hyper-tracker update evt_f3a1b9c2 \
  --status resolved \
  --resolution "OpenSSH upgraded to 9.4p1 on both web-prod-01 and web-prod-02 during emergency maintenance window. Verified by checking `sshd -T | grep version` and running test scan." \
  --resolution-by senior-analyst@company.com \
  --add-evidence /tmp/patch-verification.txt

# 7. Cerrar ticket JIRA vinculado automáticamente (si configurado)
# (sucede via webhook)
```

### Ejemplo 2: Respuesta a incidente - ransomware sospechoso

**Trigger**: Alerta SIEM por actividad SMB inusual

```bash
# Webhook recibido y incidente creado automáticamente
hyper-tracker show inc_9a2b3c4d --timeline --full
```

Salida:
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
    "status": "running"  // no pausado
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

**Flujo de trabajo del analista con playbook**:

```bash
# 1. Ver acciones recomendadas en detalle
hyper-tracker recommend inc_9a2b3c4d --type containment --limit 10
```

Salida:
```json
{
  "incident_id": "inc_9a2b3c4d",
  "recommendations": [
    {
      "step": 1,
      "action": "Isolar sistema comprometido de red",
      "details": "Remover finance-db-01 de segmento de red. Usar deshabilitación de puerto de switch de red o firewall basado en host. preservar evidencia.",
      "commands": [
        "En switch de red: deshabilitar puerto Gi1/0/24",
        "En host Windows: netsh advfirewall set allprofiles firewallpolicy blockinbound,blockoutbound"
      ],
      "sla_timer": "inmediato (SLA: 1 hora)",
      "evidence_required": "Confirmación de aislamiento de red (captura de pantalla o ticket)"
    },
    {
      "step": 2,
      "action": "Preservar evidencia volátil",
      "details": "Antes de apagar, colectar volcado de memoria y lista de procesos en ejecución. Usar FTK Imager o DumpIt.",
      "commands": [
        "Memoria: dumpit.exe -o memory.raw",
        "Lista procesos: pslist > processes.txt",
        "Conexiones de red: netstat -ano > connections.txt"
      ],
      "sla_timer": "dentro de 30 minutos",
      "evidence_required": ".raw archivo de memoria, lista de procesos, conexiones de red"
    },
    {
      "step": 3,
      "action": "Verificar integridad de backup",
      "details": "Verificar que backups para sistemas afectados no sean accesibles desde red de producción y estén libres de malware.",
      "commands": [
        "Revisar timestamps de archivos de backup - ¿alguna modificación en últimas 24h?",
        "Montar backup solo-lectura y escanear con antivirus",
        "Probar restore de base de datos crítica a entorno aislado"
      ],
      "sla_timer": "dentro de 2 horas",
      "evidence_required": "Reporte de escaneo backup, log de prueba de restore"
    },
    {
      "step": 4,
      "action": "Alcance movimiento lateral",
      "details": "Verificar autenticación desde finance-db-01 a otros sistemas. Buscar conexiones SMB, RDP, SSH.",
      "queries": [
        "SIEM: source IP = finance-db-01 AND dest_port in (445,3389,22)",
        "Azure AD: sign-ins from compromised user accounts",
        "Check for new admin accounts created in last 24h"
      ],
      "sla_timer": "dentro de 4 horas",
      "evidence_required": "Resultados de consulta de logs, lista de cuentas potencialmente comprometidas"
    }
  ]
}
```

**El analista completa pasos**:

```bash
# 2. Actualizar incidente con evidencia y progreso
hyper-tracker update inc_9a2b3c4d \
  --status investigating \
  --add-comment "Paso 1 completo: finance-db-01 aislado en puerto de switch Gi1/0/24. Ver evidencia network-isolation-proof.txt. Volcado de memoria colectado (memory.raw, sha256: d4e5f6...)." \
  --add-evidence /evidence/network-isolation-proof.txt \
  --add-evidence /evidence/memory-dump.raw

# 3. Ejecutar predicción IA para próximos movimientos del atacante
hyper-tracker predict inc_9a2b3c4d --scenario lateral-movement
```

Salida:
```json
{
  "predictions": [
    {
      "step": 1,
      "probability": 0.87,
      "technique": "T1078 - Valid Accounts",
      "target": "domain controllers",
      "timeline": "24-48 hours",
      "trigger": "Attacker ya tiene credenciales de dominio de finance-db-01",
      "recommended_defenses": [
        "Enable PIM for all privileged accounts (not yet deployed)",
        "Implement LAPS on all workstations",
        "Monitor for authentication from unusual locations"
      ]
    }
  ]
}
```

**Creando log de evidencia**:

```bash
# 4. Después de contener y resolver
hyper-tracker update inc_9a2b3c4d \
  --status resolved \
  --resolution "Ransomware contenido. No se pagó rescate. Atacante obtuvo acceso inicial via cuenta de servicio comprometida (reutilización de contraseña). Cuenta de servicio deshabilitada. Sistema reconstruido desde imagen limpia. Datos restaurados desde backup offline. No hay evidencia de exfiltración de datos. Postmortem: https://confluence.company.com/incidents/inc_9a2b3c4d" \
  --resolution-by tier2-analyst@company.com \
  --resolution-date 2024-03-15T18:30:00Z \
  --add-tag ransomware,lockbit,resolved-no-ransom \
  --add-comment "Lecciones aprendidas: Implementar PAM, reducir uso de cuentas de servicio, forzar MFA para acceso admin. Elementos de acción: 1) Desplegar PIM para Q2, 2) Auditar todas las cuentas de servicio, 3) Habilitar MFA para todos admins antes 15 abr."
```

**Resultados de SLA**:
- Reconocimiento: 7 minutos (dentro de 15m) ✓
- Contención: 4 horas (dentro de 1h SLA) ✗ (incumplido)
- Razón: "Recolección de volcado de memoria tomó más tiempo de lo esperado (2 horas). Se actualizará playbook para paralelizar tareas."
- Resolución: 4h 7m total (dentro de 24h) ✓

**Alerta enviada a gestión**:
```
[SLACK #security-management]
⚠️ INCIDENTE INCUMPLIÓ SLA
Incidente: inc_9a2b3c4d - Ransomware - finance-db-01
SLA: Contención (1 hora) - Incumplido por 1 hora 7 minutos
Estado actual: Resuelto (no se pagó rescate, datos restaurados)
Propietario: tier2-analyst@company.com
Razón incumplimiento: Recolección de volcado de memoria tomó más tiempo de lo esperado
Postmortem: https://confluence.company.com/incidents/inc_9a2b3c4d
```

### Ejemplo 3: Enriquecimiento de IOC e inteligencia de amenazas

**Escenario**: Nuevo log de bloqueo de firewall muestra IP 185.220.101.77 intentando conectar a múltiples servidores. El analista verifica la IP:

```bash
hyper-tracker enrich-ioc 185.220.101.77 --sources all --force
```

Salida:
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

**Acciones tomadas**:

```bash
# 1. Ver qué incidentes ya están rastreados
hyper-tracker search --ioc 185.220.101.77 --format table

# 2. Buscar todos los activos que vieron tráfico desde esta IP
hyper-tracker search --query "185.220.101.77" --format table

# 3. Si es nuevo y malicioso, bloquear regla de firewall (usando integración)
hyper-tracker alert add \
  --name "Firewall block automatically" \
  --condition '{"and": [{"==": [{"var": "ioc"}, "185.220.101.77"]}, {">": [{"var": "risk_score"}, 90]}]}' \
  --channel webhook \
  --recipient https://firewall.company.com/api/block \
  --template '{"action":"block_ip","ip":"{{ioc}}","duration":"permanent","reason":"{{ai_analysis.campaign_association.campaign}} - confidence {{ai_analysis.campaign_association.confidence * 100}}%"}'

# (Salida: webhook disparado exitosamente, firewall reconocido)

# 4. Añadir IOC a watchlist si no presente
hyper-tracker ioc add \
  --value 185.220.101.77 \
  --type ip \
  --source virustotal \
  --confidence 0.95 \
  --threat-type c2 \
  --campaign "Cobalt Spider" \
  --first-seen 2023-11-15
```

**Payload de webhook de firewall**:
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

**Respuesta de firewall**:
```
{"status": "success", "rule_id": "HT-BLOCK-20240315-184500-18522010177", "message": "IP added to permanent block list. Will persist across reboots."}
```

**Comentario auto-añadido a todos los incidentes que contienen este IOC**:

```bash
# (automatizado por acción de webhook)
hyper-tracker update inc_7x8y9z \
  --add-comment "2024-03-15 14:45: IP 185.220.101.77 blocked at firewall perimeter. Additional context: linked to LockBit activity per OTX pulse 7c8d9e0f1a2b."
```

### Ejemplo 4: Recopilación de evidencia de cumplimiento

**Requisito PCI-DSS 8.2.3**: "Revisar acceso de usuarios del sistema al menos trimestralmente."

```bash
# 1. Buscar todos los eventos de revisión de acceso
hyper-tracker search \
  --type compliance \
  --tag pci-dss-8.2.3 \
  --date-start 2024-01-01 \
  --date-end 2024-03-31 \
  --format json
```

Retorna:
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

# 2. Generar reporte de brecha de cumplimiento para auditor
hyper-tracker report compliance-gap \
  --framework PCI-DSS \
  --date-range "2024-01-01 to 2024-03-31" \
  --output /tmp/pci-dss-q1-2024-report.pdf \
  --format pdf
```

Fragmento de reporte:
```
PCI-DSS v4.0 STATUS DE CUMPLIMIENTO - Q1 2024
Generated: 2024-03-15 14:30:00 UTC
Periodo: 1 enero 2024 - 31 marzo 2024

REQUISITO 8: IDENTIFY AND AUTHENTICATE ACCESS TO SYSTEM COMPONENTS
--------------------------------------------------------------------------------

8.2.3 - Revisión trimestral de acceso de usuarios
Estado: COMPLIANT ✓
Evidencia: comp_abc123 (completada 2024-03-10)
Cobertura: 856 sistemas, 1247 usuarios revisados
Excepciones: 3 (todas aprobadas, documentadas)
Reportero: compliance-officer@company.com

Trail de evidencia:
  • Revisión realizada via evento de cumplimiento Hyper Tracker
  • Spreadsheet de revisión de acceso subida como evidencia (PDF firmado)
  • Verificación automatizada: Matriz de acceso comparada con registros de RRHH
  • Excepciones: 2 empleados terminados aún tenían acceso (corregido en 24h), 1 extensión de contratista aprobada

Prueba de control automatizada pasó:
  ✓ Todas cuentas de usuario revisadas dentro del trimestre
  ✓ Revisión documentada con firma de aprobador
  ✓ Excepciones rastreadas y remediadas

REQUISITO 6: DEVELOP AND MAINTAIN SECURE SYSTEMS AND APPLICATIONS
--------------------------------------------------------------------------------

6.2 - Ensure that all system components and software are protected from known vulnerabilities
Estado: PARCIALMENTE COMPLIANT (87/100) ⚠

Vulnerabilidades críticas abiertas > 30 días: 3 (↓2 vs trimestre anterior)
   CVE-2024-3094 - web-prod-01 (edad: 15 días) - P0
   CVE-2024-2987 - payment-db-01 (edad: 42 días) - P1
   CVE-2023-38408 - ldap-server-01 (edad: 67 días) - P1

Brecha de cumplimiento:
  • 2 vulnerabilidades de alto riesgo excediendo ventana de remediación de 30 días
  • payment-db-01 (críticidad de activo: CRÍTICO) vencido por 12 días

Acción requerida:
  1. CVE-2024-2987: Parchear antes 27 marzo (11 días restantes)
  2. CVE-2023-38408: Programar mantenimiento de emergencia (vencido 37 días)

Attested by: CISO - pendiente
```

### Ejemplo 5: Importación en lote y deduplicación

**Escenario**: Migrando desde sistema de ticketing heredado con 10,000 incidentes.

```bash
# Dry-run para validar mapeo
hyper-tracker import \
  --source ./legacy-incidents-2023.jsonl \
  --format jsonl \
  --map-fields '{"type": "incident_type", "severity": "priority", "title": "subject", "details": "description", "create_date": "opened_at", "close_date": "resolved_at", "assignee": "owner_email", "tags": "tags"}' \
  --dry-run
```

Reporte dry-run:
```
DRY RUN - No se modificarán datos
Source: ./legacy-incidents-2023.jsonl (9,847 líneas)
Eventos estimados: 9,847
Validación de mapeo:
  ✓ mapeo tipo OK (heredado: ['sec_incident','info_event','malware'] → ['incident','incident','incident'])
  ✓ mapeo severidad OK (heredado P1-5 → Hyper Tracker high/medium/low)
  ✓ parseo de fecha OK (formato ISO 8601 detectado)
  ⚠ 142 eventos faltan 'assignee' → por defecto 'security-team@'
  
Detección de duplicados:
  • 1,234 eventos ya existen (por hash CVE o título+activofuzzy match)
  • Se saltarían: 1,234 (12.5%)
  • Se crearían: 8,613 (87.5%)

Nuevos activos descubiertos:
  • 89 activos no en inventario actual
  • Auto-creación: YES (flag --create-assets no configurado, usar --auto-create-assets para habilitar)

Confianza: ALTA - listo para proceder

Duración estimada: 15-20 minutos
```

Importación real con auto-crear activos:

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

Progreso en tiempo real:
```
Importando: 100% |██████████████████████████| 9847/9847 [15m 42s transcurridos, 0s restantes]

✓ Importación completada exitosamente
  • Eventos procesados: 9,847
  • Eventos creados: 8,613 (nuevos)
  • Eventos saltados (duplicados): 1,234
  • Eventos actualizados (existentes): 0
  • Nuevos activos creados: 89
  • Errores de validación: 0

Enriquecimiento IA en cola para 8,613 eventos
  ID de trabajo: ai_import_20240315_abc123
  Completación estimada: 3 horas (10 workers)
  Monitorear: hyper-tracker model status

Recomendaciones post-import:
  1. Ejecutar enriquecimiento IA: Ya en cola (completará automáticamente)
  2. Generar reporte de resumen de importación: hyper-tracker report import-summary --session ai_import_20240315_abc123
  3. Revisar nuevos activos: hyper-tracker asset list --tag auto-created-import-20240315
  
Logs: /var/log/hyper-tracker/import_20240315_abc123.log
```

**Procesamiento de enriquecimiento IA automático**:

```bash
# Verificar estado del trabajo IA
hyper-tracker model status
```

Salida:
```
AI Workers: 10 activos
Queue:
  • eventos de vulnerabilidad: 4,231 (est. 1h 15m)
  • eventos de incidente: 4,382 (est. 2h 30m)
  
Procesando actualmente: modelo de vulnerabilidad (lote 432/4231)
Último lote completado: 431 eventos en 89s (4.8 eventos/seg)

Completación esperada:
  • Todos eventos enriquecidos por: 2024-03-15 17:30 UTC
```

Después del enriquecimiento, el analista revisa una muestra:

```bash
hyper-tracker search --tag auto-created-import-20240315 --limit 5 --format table
```

Salida tabla:
```
ID              TYPE      SEVERITY  TITLE                                    ASSETS  AI_RISK  STATUS
evt_x1y2z3a     incident  high      Suspicious outbound traffic detected   2       78       open
evt_b4c5d6e     vuln      medium    Apache Log4j CVE-2021-44228            1       65       resolved
evt_f7g8h9i     incident  critical  Ransomware on file-server-01           1       96       closed
evt_j0k1l2m     vuln      unknown   WordPress plugin XSS                  1       45       open
evt_n3o4p5q     incident  low       Phishing email reported                0       22       closed
```

**El analista añade contexto faltante a eventos heredados**:

```bash
# Añadir técnica MITRE ATT&CK a todos los incidentes de phishing de la importación
hyper-tracker search --tag auto-created-import-20240315 --type incident --query "phishing" --format json \
  | jq -r '.[] | .event_id' \
  | xargs -I {} hyper-tracker update {} --add-tag technique:T1566.002
```

### Ejemplo 6: Entrenamiento y evaluación de modelo

**Problema**: El modelo IA subestima riesgo para vulnerabilidades cloud-native (contenedores, serverless).

```bash
# 1. Exportar datos de entrenamiento de incidentes del año pasado
hyper-tracker export \
  --table events \
  --filters '{"type":"vulnerability","created_at":{"$gte":"2023-01-01","$lte":"2023-12-31"}}' \
  --output /tmp/training_vulns_2023.jsonl \
  --format jsonl
```

**2. Etiquetar dataset con resultado (¿fue explotada esta vuln?)**

Convertir a formato de entrenamiento:
```python
# scripts/prepare_training.py
import json
with open('/tmp/training_vulns_2023.jsonl') as f_in, open('/tmp/train.jsonl', 'w') as f_out:
    for line in f_in:
        event = json.loads(line)
        # Ground truth: explotada si vinculada incidente con ransomware/apt
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

**3. Entrenar nuevo modelo**:

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

Salida dry-run:
```
Model: vulnerability
Dataset: /tmp/train.jsonl (45,231 ejemplos)
Ejemplos de entrenamiento: 36,185
Ejemplos de validación: 9,046
Epochs: 5
Batch size: 32
Learning rate: 2e-5
Backup modelo actual: YES (to /var/lib/hyper-tracker/models/backup/vulnerability-20240315-143022/)

Duración estimada: 2-3 horas (GPU: auto-detect)
Espacio en disco estimado: 500MB para nuevo modelo
 
DRY RUN - No se harán cambios
Para ejecutar entrenamiento, eliminar flag --dry-run

Cambios propuestos:
  • Nuevo modelo: vulnerability-custom-20240315 (tamaño: ~500MB)
  • Reemplazará: vulnerability (actual: v2.1)
  • Backup retenido: YES (30 días)
  • Validación requerida: YES (accuracy > 0.85)
```

Entrenamiento real (con GPU):
```bash
hyper-tracker model train \
  vulnerability \
  --dataset /tmp/train.jsonl \
  --epochs 5 \
  --backup-before
```

Progreso:
```
Iniciando entrenamiento: vulnerability-custom-20240315
✓ Dataset cargado: 45,231 ejemplos (36,185 train, 9,046 val)
✓ Modelo inicializado: microsoft/cvebert-base-uncased
✓ Loss: CrossEntropy, Optimizer: AdamW
───────────────────────────────────────────────────
Epoch 1/5 [█████████░░░░░░░░░░] 65% - loss: 0.4234 - val_loss: 0.3876 - val_acc: 0.8234
Epoch 2/5 [███████████░░░░░░] 45% - loss: 0.3123 - val_loss: 0.2987 - val_acc: 0.8541
Epoch 3/5 [████████████░░░░] 23% - loss: 0.2567 - val_loss: 0.2734 - val_acc: 0.8678
───────────────────────────────────────────────────
Entrenamiento completado: 5/5 epochs
Métricas finales:
  • Training accuracy: 0.8912
  • Validation accuracy: 0.8734
  • F1 score (exploited): 0.8245
  
Modelo guardado: /var/lib/hyper-tracker/models/vulnerability-custom-20240315/
Backup creado: /var/lib/hyper-tracker/models/backup/vulnerability-v2.1-20240315-143022/
```

**4. Evaluar modelo en conjunto de test de holdout**:

```bash
hyper-tracker model evaluate \
  vulnerability-custom-20240315 \
  --test-set /tmp/test_set_2024.jsonl \
  --metrics all
```

Salida:
```
Model: vulnerability-custom-20240315
Test set: 11,307 ejemplos

MÉTRICAS
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
Critical CVEs: precision=0.89, recall=0.87, f1=0.88 (3,241 ejemplos)
High CVEs: precision=0.82, recall=0.79, f1=0.80 (4,102 ejemplos)
Medium/Low: precision=0.76, recall=0.71, f1=0.73 (3,964 ejemplos)

CALIDAD DE MODELO: ACEPTABLE PARA PRODUCCIÓN ✓
Recommendation: Desplegar a 10% canary primero, monitorear tasa falso positivo.
```

**5. Promover modelo a producción**:

```bash
hyper-tracker model promote \
  vulnerability-custom-20240315 \
  --canary-percent 10 \
  --monitor-period 7d \
  --rollback-on-fp-rate >0.15
```

Salida:
```
Promoción de modelo iniciada:
  • Producción actual: vulnerability-v2.1 (100% tráfico)
  • Nuevo modelo: vulnerability-custom-20240315
  • Canary: 10% de nuevos eventos usarán nuevo modelo
  • Rollback trigger: tasa falso positivo > 15%
  • Período de monitoreo: 7 días
  
Dashboard: /hyper-tracker/admin/models?promotion=abc123
Notifications: #security-ml (alerts), security-team@company.com (daily digest)

Next steps:
  1. Monitorear tasa falso positivo diariamente
  2. Si FP rate < 10% después 3 días, aumentar gradualmente a 25%, 50%, 100%
  3. Si FP rate > 15% en cualquier punto, rollback automático disparado
  4. Métrica de éxito: mantener >85% precisión mientras reduce falsos negativos en 20%
```

### Ejemplo 7: Monitoreo de SLA y manejo de incumplimientos

**Revisión regular de estado SLA**:

```bash
hyper-tracker sla status --format table --show-breaches
```

Salida:
```
INCIDENT SLA STATUS - Tiempo real
┌─────────────────────┬─────────────┬─────────────┬─────────────┬──────────────┐
│ Incident            │ Priority    │ Ack Deadline│ Con Deadline│ Res Deadline │
├─────────────────────┼─────────────┼─────────────┼─────────────┼──────────────┤
│ inc_9a2b3c4d        │ P0 (critical)│ 14:20 (7m) │ 15:20 (1h 7m)│ Mañana 14:23│
│ inc_9a2b3c4e        │ P0 (critical)│ 14:25 (2m) │ 15:25 (1h 2m)│ Mañana 09:15│
│ inc_9a2b3c4f        │ P1 (high)   │ 16:00 (2h 35m)│ 20:00 (6h 35m)│ 2024-03-18  │
└─────────────────────┴─────────────┴─────────────┴─────────────┴──────────────┘

CUMPLIMIENTO SLA (Últimos 7 días)
├─ P0 reconocer: 95% (19/20) ⚠ 1 incumplimiento
├─ P0 contención: 80% (16/20) ⚠ 4 incumplimientos
├─ P0 resolución: 70% (14/20) ⚠ 6 incumplimientos
├─ P1 reconocer: 98% (49/50) ✓
├─ P1 contención: 92% (46/50) ⚠ 4 incumplimientos
└─ General: 89% ✓

INCUMPLIMIENTOS ACTUALES (3)
────────────────────
1. inc_9a2b3c4d - P0 contención incumplida en 1h 7m (SLA: 1h)
   Razón: Recolección de volcado de memoria tomó más tiempo de lo esperado
   Propietario: tier2-analyst@company.com
   Escalación: Notificará a gestión en incumplimiento+30m

2. inc_9a2b3c4e - P0 reconocimiento a las 14:28 (SLA: 14:25)
   Razón: Retraso en asignación - esperando respondedor de guardia
   Propietario: tier1-team@company.com

3. inc_9a2b3c4f - P1 contención incumplida en 6h 35m (SLA: 8h)
   Razón: Pendiente aprobación legal para aislamiento de sistema
   Propietario: legal-team@company.com

---
Reporte SLA completo: /var/reports/sla/last-7d.html
```

**Notificación automática de incumplimiento de SLA**:

```bash
hyper-tracker alert add \
  --name "SLA Breach" \
  --condition '{"==": [{"var": "sla.status"}, "breached"}]}' \
  --channel slack \
  --recipient "#security-management" \
  --template "sla-breach"
```

Mensaje Slack:
```
🚨 SLA BREACH DETECTED

Incident: inc_9a2b3c4d
Type: P0 Incident - Ransomware
SLA: Contención (1 hora) - Incumplido por 1 hora 7 minutos
Propietario: tier2-analyst@company.com
Razón incumplimiento: "Memory dump collection took longer than expected"
Estado actual: Investigating

Required: Manager review and postmortem within 24 hours
[Ver Incidente](http://hyper-tracker:8080/incidents/inc_9a2b3c4d)
```

**Manager anula temporizador SLA** (razón válida):

```bash
hyper-tracker update inc_9a2b3c4d \
  --add-comment "SLA pause approved by manager@company.com: legitimate reason - memory preservation took precedence over containment. SLA pause recorded from 14:45 to 15:30 (45 minutes). New containment deadline: 16:12." \
  --sla-pause 45m \
  --sla-pause-reason "Forensic memory collection priority"
```

Ahora cuenta regresiva SLA se ajusta:
```
Containment SLA: 45m paused → new deadline: 16:12
```

### Ejemplo 8: Consultas y visualizaciones de dashboard

**El gerente de seguridad ve dashboard**:

```
http://hyper-tracker:8080
Authentication: Basic (SSO via LDAP)
```

**Pestaña Overview**:
```
CARD 1: Incidentes Abiertos por Prioridad
  P0: 3 (↑1 de ayer)
  P1: 7 (→)
  P2: 13 (↓2)
  P3: 0

CARD 2: Vulnerabilidades Críticas (risk_score > 90)
  2 (↑1)
  ├─ web-prod-01: CVE-2024-3094 (94)
  └─ payment-db-01: CVE-2024-2987 (91) - 42 días vencido

CARD 3: MTTR (Últimos 30 días)
  P0: 2h 15m (objetivo: <4h) ✓
  P1: 8h 42m (objetivo: <24h) ✓
  P2: 3d 2h (objetivo: <7d) ✗

CARD 4: Nuevos Eventos (24h)
  Total: 47 (↑12%)
  Vulnerabilities: 23 (↑5)
  Incidents: 12 (↑4)
  IOCs: 8 (↑2)
  Other: 4

CHART: Events by Severity (7-day trend)
  • Gráfico de línea con critical/↑, high/↔, medium/↓, low/→
  
CHART: SLA Compliance (weekly bar)
  • Esta semana: 89% (↓2% de la semana pasada)
```

**Pestaña Assets**:
- Vista árbol: Production (45 activos) → Equipo: Web (12), Database (8), Network (5)...
- Heatmap: codificado por color por max risk_score en activo (rojo: >85, naranja: 70-85, amarillo: 50-70, verde: <50)
- Click en activo "web-prod-01":
  ```
  Asset: web-prod-01 (10.10.1.101)
  Criticalidad: HIGH
  Entorno: Production
  Propietario: web-team@company.com
  
  Vulnerabilidades (abiertas):
  ┌─────────────────────────────┬────────────┬────────────┬────────────┐
  │ CVE                         │ Severidad  │ Risk Score │ Edad (días)│
  ├─────────────────────────────┼────────────┼────────────┼────────────┤
  │ CVE-2024-3094               │ critical   │ 94         │ 2          │
  │ CVE-2024-2987               │ high       │ 87         │ 15         │
  └─────────────────────────────┴────────────┴────────────┴────────────┘

  Incidentes (últimos 90 días): 3
    • inc_9a2b3c4d (ransomware) - resuelto
    • inc_8z9y0x1w (SSH brute force) - resuelto
    • inc_7v6w5u4t (data exfiltration) - resuelto

  Tendencia de Riesgo: ⬆️ (aumentó 12 puntos en últimos 30 días)
  Recomendado: Priorizar parche CVE-2024-3094 en 48h, revisar CVE-2024-2987 para parche acelerado.
  ```

**Pestaña IOCs**:
- Tabla: Valor | Tipo | Amenaza | Campaña | Primera Visto | Último Visto | Eventos Vinculados
- Filtro: Solo mostrar IOCs vinculados a incidentes activos
- Búsqueda: `campaign:"Cobalt Spider"`
- Click IOC 185.220.101.77:
  ```
  IOC: 185.220.101.77 (IP)
  Tipo de Amenaza: C2 Server
  Campaña: Cobalt Spider (94% confianza)
  Primera visto: 2023-11-15
  Último visto: 2024-03-15T08:45:00Z
  
  Enriquecimiento:
  ┌─────────────┬────────────────────────────────────────────────────────────┐
  │ VT detections│ 45/75 (harmless:0, malicious:45, suspicious:2)         │
  │ OTX pulses   │ 3 (Cobalt Spider, Emerging C2, LockBit)               │
  │ GreyNoise    │ Malicious, active, not Tor                           │
  │ AbuseIPDB    │ 1,247 reports, confidence 95, last: 6h ago          │
  └─────────────┴────────────────────────────────────────────────────────────┘
  
  Eventos Vinculados: 14
  ├─ inc_9a2b3c4d (ransomware) - abierto
  ├─ inc_8z9y0x1w (SSH brute) - cerrado
  └─ 12 otros incidentes (mayormente ene-feb 2024)
  
  Acción: Ya bloqueado en firewall (ID de regla: HT-BLOCK-20240315-184500)
  ```

### Ejemplo 9: Solución de problemas comunes

**Problema 1: Análisis IA atascado en "queued" por horas**

```bash
# Verificar estado de workers IA
hyper-tracker model status
```

Salida muestra:
```
AI Workers: 0/10 (all stopped)
Queue: events: 342 (stuck)
ERROR: CUDA out of memory on worker-3
```

Solución:
```bash
# Verificar recursos del sistema
free -h  # RAM: 32GB total, 28GB usado, 4GB libre → memoria baja
nvidia-smi  # memoria GPU: 24GB total, 22GB usado → lleno

# Reiniciar workers IA con batch size más pequeño
hyper-tracker config set ai.batch_size 16 --scope ai
hyper-tracker maintenance restart-workers ai

# O pausar otros trabajos GPU
# Detener otros trabajos de entrenamiento:
hyper-tracker model stop --job-id train_abc123

# Reducir workers de cola para coincidir con capacidad GPU
hyper-tracker config set ai.max_workers 5 --scope ai
hyper-tracker maintenance restart-workers ai

# Limpiar cola atascada (¡cuidado!)
hyper-tracker maintenance clear-queue --older-than 2h --dry-run
# Muestra: Would clear 342 events (older than 2h)
hyper-tracker maintenance clear-queue --older-than 2h
# Limpia eventos más antiguos (necesitarán re-queue manual si importantes)
```

**Problema 2: Eventos duplicados apareciendo**

```bash
# Buscar duplicados (mismo CVE + activo)
hyper-tracker search --type vulnerability --cve CVE-2024-3094 --asset-id web-prod-01 --format json | jq length
# Salida: ¡5 duplicados!

# Verificar configuración de deduplicación
hyper-tracker config get deduplication.enabled
# Salida: true

# Verificar si misma fuente creando duplicados
hyper-tracker search --cve CVE-2024-3094 --asset-id web-prod-01 --format json | jq -r '.[].source' | sort | uniq -c
# Salida:
#   3 qualys
#   2 manual

# La fuente qualys se ejecutó múltiples veces en periodo corto y no está deduplicando correctamente.

# Fix: Verificar función hash de dedupe
hyper-tracker maintenance dedupe-check --cve CVE-2024-3094

# Salida muestra que eventos del mismo día pero diferentes horas tienen hashes diferentes debido a campo timestamp incluido.

# Solución: Reconfigurar dedupe para ignorar timestamp y campo source
hyper-tracker config set deduplication.fields "type,cve,asset_ids,title" --scope database
# Esto asegura mismo CVE en mismo activo deduplicará independientemente de tiempo de ingestión/fuente.

# Fusión manual de duplicados:
hyper-tracker dedupe merge evt_abc123 evt_def456 evt_ghi789 \
  --keep-newest \
  --reason "Duplicate from Qualys re-scan"

# Ahora buscar de nuevo: debería ser 1 evento.
```

**Problema 3: Alertas no disparándose**

```bash
# 1. Verificar que regla de alerta existe y está habilitada
hyper-tracker alert list --format table

# 2. Verificar si condición coincide con algún evento
hyper-tracker alert test --name "Critical prod vuln - weaponized" --count 10

# Esto prueba regla contra últimos 10 eventos
# Salida: "0 matches" - condición de regla no coincide con eventos esperados

# 3. Inspeccionar condición de regla
hyper-tracker alert show "Critical prod vuln - weaponized"
# Muestra condición JSON

# Probar condición directamente:
hyper-tracker search --format json --limit 1 | jq -c '.[]' | hyper-tracker alert test-json --condition '{"and": [...]}' 
# (Usar condición de regla)

# Problema común: campo asset.environment no coincide con nombre
# Verificar schema de activo actual:
hyper-tracker asset show web-prod-01 --format json | jq '.environment'
# Salida: "production" (minúsculas)

# Pero condición de regla usa: {"==": [{"var": "asset.environment"}, "Production"]} (¡con mayúsculas!)

# Fix: Actualizar condición de regla
hyper-tracker alert update "Critical prod vuln - weaponized" \
  --condition '{"and": [{"==": [{"var": "type"}, "vulnerability"]}, {">": [{"var": "risk_score"}, 85]}, {"==": [{"var": "asset.environment"}, "production"]}, {"==": [{"var": "ai_analysis.outputs.exploit_prediction.weaponized"}, true]}]}' \
  --dry-run
# (verificar que condición se vea correcta)
hyper-tracker alert update "Critical prod vuln - weaponized" --condition <same>

# 4. Verificar throttling/cooldown de alerta
hyper-tracker status | grep throttle
# Podría mostrar: "throttle: 5 alerts sent in last hour, limit: 5/hour"

# ¡Esa es la razón por la que nuevas alertas no se disparan! Esperar cooldown o aumentar límite.

# Opción: reducir cooldown temporalmente para pruebas
hyper-tracker alert update "Critical prod vuln - weaponized" --cooldown 30
```

**Problema 4: Dashboard no carga o lento**

```bash
# 1. Verificar servicio dashboard
hyper-tracker status | grep dashboard
# Salida: "Dashboard: http://localhost:8080 (status: down)"

# 2. Verificar logs
tail -100 /var/log/hyper-tracker/dashboard.log | grep ERROR

# Error común: "Database is locked"
# Causa: checkpoint WAL no se ejecuta, consultas de larga duración mantienen bloqueo.

# Fix:
hyper-tracker maintenance vacuum --aggressive
# Esto ejecuta SQLite VACUUM y checkpoint

# 3. Verificar tamaño de base de datos e indexación
ls -lh /var/lib/hyper-tracker/data.db*
# data.db (12GB), data.db-wal (8GB) → archivo WAL grande indica necesidad de checkpoint

hyper-tracker maintenance reindex --concurrently

# 4. Verificar si muchos usuarios concurrentes del dashboard ejecutan consultas no indexadas
# Habilitar logging de consultas temporalmente:
hyper-tracker config set database.log_slow_queries 5 --scope database
# Registra consultas tomando >5s a /var/log/hyper-tracker/slow-queries.log

# Añadir índices faltantes:
hyper-tracker maintenance add-index --table events --column created_at
hyper-tracker maintenance add-index --table events --column risk_score
```

**Problema 5: Importación falla con errores de validación**

```bash
hyper-tracker import \
  --source ./data.csv \
  --format csv \
  --map-fields ./mapping.json \
  --dry-run

# Salida:
Errores de validación:
  • Fila 234: Severidad inválida 'Urgent' (permitidas: critical,high,medium,low,info,unknown)
  • Fila 567: Formato de fecha inválido '03/15/2024' (esperado: YYYY-MM-DD o ISO 8601)
  • Fila 890: Campo requerido faltante 'title' (cadena vacía)
  • Fila 1203: ID de activo 'APP-01' no encontrado y --auto-create-assets no configurado

Total: 4,123 filas | Válidas: 4,119 | Errores: 4
```

Preprocesar CSV:
```bash
# 1. Pre-procesar CSV con sed/awk
sed -i 's/Urgent/critical/g' ./data.csv
sed -i 's#03/15/2024#2024-03-15#g' ./data.csv
awk -F, 'NR==1 || $3 != ""' ./data.csv > ./data_fixed.csv  # omitir filas con título vacío

# 2. Crear activos faltantes primero
cat ./data.csv | cut -d, -f5 | tail -n +2 | sort -u > ./missing-assets.txt
while read asset; do
  hyper-tracker asset add --asset-id "$asset" --hostname "$asset.company.com" --criticality medium --environment production
done < ./missing-assets.txt

# 3. Reintentar importación
hyper-tracker import \
  --source ./data_fixed.csv \
  --format csv \
  --map-fields ./mapping.json \
  --auto-create-assets
```

## Comandos de Rollback

### 1. Rollback de creación de evento único (dentro de 5 minutos, antes de IA)

```bash
hyper-tracker rollback \
  --event-id evt_XXXX \
  --reason "Creado por error - duplicado"
```

```
✓ Evento evt_XXXX eliminado
Entrada de log de auditoría creada: "rollback: user analyst@ reason: Creado por error - duplicado"
```

Restricción: Solo funciona si evento creado <5 min atrás y análisis IA no iniciado. Si IA ya ejecutó, usar `update --status rejected` en su lugar.

### 2. Revertir evento a versión anterior (desde log de auditoría)

```bash
hyper-tracker rollback \
  --event-id evt_XXXX \
  --to-version 3
```

Revertir evento a estado en entrada de log de versión 3. Todas las actualizaciones intermedias permanecen en trail de auditoría.

### 3. Deshacer cambios de campo específicos con precisión quirúrgica

```bash
hyper-tracker rollback \
  --event-id evt_XXXX \
  --field severity \
  --to-value high
```

Solo cambia campo especificado, mantiene otras actualizaciones.

### 4. Rollback en lote de sesión de importación

Si importación en lote estuvo mal (mapeo incorrecto, tormenta de duplicados):

```bash
hyper-tracker rollback \
  --import-session imp_20240315_abc123 \
  --reason "Wrong mapping - severity inverted" \
  --dry-run
```

Dry-run muestra:
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

**IDs de sesión de importación** visibles en logs: `/var/log/hyper-tracker/import_*.log` o en `audit_log` action=`import_session`.

### 5. Restaurar desde backup con estrategias de merge

**Escenario 1**: Base de datos corrupta, restaurar desde backup de ayer:

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

Restore seguro preservando trabajo reciente:

```bash
hyper-tracker restore \
  --backup /backups/hyper-tracker-2024-03-14-full.db.gz \
  --merge-strategy merge \
  --verify
```

`--verify` verifica integridad referencial después de restore.

**Estrategias de merge**:
- `overwrite`: Backup reemplaza actual completamente (PELIGROSO)
- `skip`: Registros más nuevos en DB actual se preservan, registros más antiguos del backup se añaden
- `merge`: Merge de tres vías - mantiene versión más reciente de cada fila por `updated_at`
- `prompt`: Preguntar por cada conflicto (solo para datasets pequeños)

**Escenario 2**: Restaurar tabla única (solo eventos) desde backup manteniendo activos actuales:

```bash
hyper-tracker restore \
  --backup /backups/hyper-tracker-2024-03-14-full.db.gz \
  --table events \
  --merge-strategy skip \
  --dry-run
```

Importaría solo eventos creados antes de fecha de backup, omitiría eventos añadidos después.

**Escenario 3**: Recuperación punto-en-tiempo antes de importación mala:

```bash
# Encontrar backup justo antes de importación
ls -lh /backups/ | grep 2024-03-15
# hyper-tracker-2024-03-15-14-00-full.db.gz  (14:00, antes de 15:30 import)

hyper-tracker restore \
  --backup /backups/hyper-tracker-2024-03-15-14-00-full.db.gz \
  --merge-strategy overwrite \
  --dry-run

# Dry-run dice: eliminaría 8,613 eventos importados, restaurar 45,231 eventos pre-import

hyper-tracker restore \
  --backup /backups/hyper-tracker-2024-03-15-14-00-full.db.gz \
  --merge-strategy overwrite
```

Después de restore, re-encolar enriquecimiento IA para eventos restaurados si es necesario:
```bash
hyper-tracker ai re-queue --older-than "2024-03-15 14:00:00" --status "ai_pending"
```

### 6. Rollback de modelo IA a versión anterior

Si nuevo modelo causa falsos positivos:

```bash
# Listar modelos con rendimiento
hyper-tracker model list --show-accuracy
# Muestra:
# vulnerability v2.1 (prod) - accuracy 87%
# vulnerability v2.2 (canary) - accuracy 85% (FP rate 18%)

# Canary ya mostrando peor rendimiento, disparador de rollback automático se activa:
# Mensaje: "Promoción de modelo fallida: tasa falso positivo 18% > umbral 15%. Revirtiendo a vulnerability-v2.1"

hyper-tracker model rollback \
  --model vulnerability \
  --to-version v2.1 \
  --reason "Canary FP rate exceeded threshold" \
  --force
```

Rollback manual antes que promoción complete:
```bash
hyper-tracker model rollback \
  --model vulnerability \
  --to-version v2.1 \
  --keep-canary \
  --reason "Production issues: false positives on cloud vulns"
```

`--keep-canary` retiene archivo de modelo v2.2 para investigación.

### 7. Deshacer actualización en lote (cambio de campo masivo)

```bash
# Analista accidentalmente estableció todos los incidentes abiertos a prioridad P0
hyper-tracker update $(hyper-tracker search --status open --format json | jq -r '.[].event_id') --priority P0

# Ups - ese fue 247 incidentes actualizados incorrectamente.

# Rollback usando log de auditoría:
hyper-tracker audit \
  --action update \
  --user analyst@company.com \
  --date-start "$(date -d '1 hour ago' +%Y-%m-%dT%H:%M)" \
  --format json \
  | jq -r '.[] | select(.details | contains("priority")) | .event_id' \
  | sort -u \
  > /tmp/events-to-revert.txt

# Previsualizar cambios
hyper-tracker show $(head -5 /tmp/events-to-revert.txt) --full | jq '.priority'

# Revertir en lote usando valor anterior de auditoría
while read event_id; do
  old_priority=$(hyper-tracker audit --event-id "$event_id" --format json | jq -r '.[] | select(.action=="update" and .details | contains("priority")) | .details | capture("(?<old>p[0-3])") | .old')
  hyper-tracker update "$event_id" --priority "$old_priority" --add-comment "Rollback: incorrect mass priority update"
done < /tmp/events-to-revert.txt
```

### 8. Revertir cambios de configuración webhook/alert

```bash
# Eliminó alerta importante por error
hyper-tracker alert list --format json > ./alerts-backup.json  # (¡hacer esto regularmente!)

# Recrear alerta desde backup
jq -r '.[] | select(.name=="Critical prod vuln - weaponized")' ./alerts-backup.json \
  | hyper-tracker alert create --dry-run  # inspeccionar
jq -r '.[] | select(.name=="Critical prod vuln - weaponized")' ./alerts-backup.json \
  | hyper-tracker alert create
```

**Versionado de configuración**: Mantener siempre `config.yaml` en git. Para revertir config:
```bash
git -C ~/.config/hyper-tracker log -p config.yaml
git -C ~/.config/hyper-tracker checkout HEAD~1 -- config.yaml
hyper-tracker config reload
```

### 9. Emergencia: Reconstruir base de datos desde log de eventos (fallo catastrófico)

Si `data.db` está corrupta más allá de reparación pero log de auditoría intacto:

```bash
# 1. Detener todos los servicios
hyper-tracker maintenance stop

# 2. Backup de todo primero
cp /var/lib/hyper-tracker/data.db /backups/corrupt-$(date +%Y%m%d-%H%M).db
cp -r /var/log/hyper-tracker /backups/logs-$(date +%Y%m%d-%H%M)

# 3. Crear base de datos fresca
hyper-tracker init --force  # creará DB vacía

# 4. Reproducir eventos desde log de auditoría y logs de ingestión
hyper-tracker rebuild \
  --from-audit-log /var/log/hyper-tracker/audit.log \
  --from-ingest-logs /var/log/hyper-tracker/ingest.log.* \
  --dry-run

# Dry-run muestra:
# Would recreate: 45,231 events
# Would recreate: 1,248 assets
# Would recreate: 4,532 IOCs
# Estimated time: 45 minutes

hyper-tracker rebuild \
  --from-audit-log /var/log/hyper-tracker/audit.log \
  --from-ingest-logs /var/log/hyper-tracker/ingest.log.* \
  --skip-duplicates
```

Proceso de reconstrucción:
1. Parsear log de auditoría para `action: create` en eventos, activos, iocs
2. Reconstruir payloads originales (desde campo `details` o embebido)
3. Insertar en orden cronológico respetando claves foráneas
4. Reconstruir caché de análisis IA (re-ejecutar IA si modelos disponibles)
5. Recrear alertas y webhooks desde config

### 10. Rollback de tabla específica únicamente

```bash

```

## Glosario

- **IOC**: Indicator of Compromise (indicador de compromiso)
- **CVSS**: Common Vulnerability Scoring System
- **EPSS**: Exploit Prediction Scoring System
- **MITRE ATT&CK**: Marco de táctnicas y técnicas de adversarios
- **SLA**: Service Level Agreement
- **MTTR**: Mean Time to Respond/Resolve
- **TTD**: Time to Detect
- **FP**: False Positive
- **FN**: False Negative
- **WAL**: Write-Ahead Logging (SQLite)
- **LSTM**: Long Short-Term Memory (red neuronal)
- **RAG**: Retrieval-Augmented Generation
- **CMDB**: Configuration Management Database
- **EDR**: Endpoint Detection and Response
- **SIEM**: Security Information and Event Management
- **PAM**: Privileged Access Management
- **GRC**: Governance, Risk, and Compliance

## Soporte y Contribuciones

Para reportar bugs o solicitar features: https://github.com/security-tools/hyper-tracker/issues

Para contribuir: ver `CONTRIBUTING.md` en el repositorio.

## Licencia

MIT License - ver LICENSE para detalles.
```