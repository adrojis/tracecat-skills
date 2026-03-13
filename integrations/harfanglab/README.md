# HarfangLab EDR — Tracecat Integration

## Overview
16 action templates pour intégrer HarfangLab EDR dans Tracecat SOAR.

## Secret requis
Créer un secret `harfanglab` dans Tracecat avec :

| Clé | Description |
|-----|-------------|
| `API_KEY` | Token API HarfangLab (généré dans Console → Users → Generate token) |
| `BASE_URL` | URL de l'instance HarfangLab (ex: `https://edr.company.com:8443`) |

## Actions disponibles

### Alertes (3)
| Action | Fichier | Description |
|--------|---------|-------------|
| `tools.harfanglab.list_alerts` | `list_alerts.yaml` | Lister les alertes (filtres: status, type, severity) |
| `tools.harfanglab.get_alert_details` | `get_alert_details.yaml` | Détails d'une alerte |
| `tools.harfanglab.update_alert_status` | `update_alert_status.yaml` | Changer le statut d'alertes |

### Agents/Endpoints (4)
| Action | Fichier | Description |
|--------|---------|-------------|
| `tools.harfanglab.search_endpoints` | `search_endpoints.yaml` | Rechercher des agents |
| `tools.harfanglab.get_endpoint` | `get_endpoint.yaml` | Détails d'un agent |
| `tools.harfanglab.isolate_endpoint` | `isolate_endpoint.yaml` | Isoler un endpoint du réseau |
| `tools.harfanglab.deisolate_endpoint` | `deisolate_endpoint.yaml` | Rétablir la connectivité |

### Télémétrie (3)
| Action | Fichier | Description |
|--------|---------|-------------|
| `tools.harfanglab.search_processes` | `search_processes.yaml` | Recherche processus par hash/nom |
| `tools.harfanglab.search_network` | `search_network.yaml` | Recherche connexions réseau |
| `tools.harfanglab.search_binary` | `search_binary.yaml` | Recherche binaire par hash |

### Threat Intelligence (3)
| Action | Fichier | Description |
|--------|---------|-------------|
| `tools.harfanglab.list_iocs` | `list_iocs.yaml` | Lister les IOCs configurés |
| `tools.harfanglab.create_ioc` | `create_ioc.yaml` | Ajouter un IOC |
| `tools.harfanglab.list_threat_intel_sources` | `list_threat_intel_sources.yaml` | Lister les sources TI |

### Investigation (2)
| Action | Fichier | Description |
|--------|---------|-------------|
| `tools.harfanglab.create_job` | `create_job.yaml` | Lancer un job d'investigation |
| `tools.harfanglab.get_job_status` | `get_job_status.yaml` | Vérifier le statut d'un job |

### Response (2)
| Action | Fichier | Description |
|--------|---------|-------------|
| `tools.harfanglab.kill_process` | `kill_process.yaml` | Kill un processus distant |
| `tools.harfanglab.dump_process` | `dump_process.yaml` | Dump mémoire d'un processus |

## Authentification
- **Type** : API Key
- **Header** : `Authorization: Token <API_KEY>`
- **Port par défaut** : 8443 (HTTPS)
- Générer le token : Console HarfangLab → Administration → Users → Generate token

## Exemples de workflows

### Alert Triage
```
Webhook → list_alerts → get_alert_details → [enrichment VT/MISP] → create_case
```

### Incident Response
```
Trigger → search_endpoints (hostname) → isolate_endpoint → create_job (forensics) → notify SOC
```

### Threat Hunting
```
Schedule → search_processes (hash IOC) → search_network (C2 IP) → create_case si trouvé
```

## Sources
- API reconstituée depuis le SDK FortiSOAR : https://github.com/fortinet-fortisoar/connector-harfanglab-edr
- Cortex XSOAR integration docs : https://xsoar.pan.dev/docs/reference/integrations/hurukai
