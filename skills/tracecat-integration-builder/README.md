# tracecat-integration-builder

Intègre n'importe quelle technologie dans Tracecat à partir d'une URL.

## Comment ça marche

1. L'utilisateur donne une URL (docs, API reference, page produit)
2. Le skill recherche l'API, comprend l'auth, identifie les endpoints utiles
3. Il construit l'intégration complète : secret, actions, edges, positionnement
4. Il valide, teste et déploie

## Quand utiliser

- "Intègre-moi VirusTotal" + URL de la doc
- "On veut connecter notre Wazuh" + URL
- "Ajoute CrowdStrike à Tracecat" + URL
- N'importe quel service avec une API REST ou un webhook

## Ce qui est géré automatiquement

| Aspect | Comment |
|--------|---------|
| Recherche API | WebFetch de la doc, recherche d'OpenAPI spec |
| Auth | Détecte le type (API key, OAuth2, Basic, JWT) et configure les secrets |
| Actions | Crée via MCP avec inputs YAML corrects |
| Edges & Layout | Connecte et positionne tous les nœuds |
| Validation | Validate + dry run avant deploy |

## 3 chemins possibles

- **Path A** : OpenAPI spec existe → Tracecat OpenAPI Converter (automatique)
- **Path B** : Pas de spec → Build manuel avec `core.http_request` + secrets (le plus courant)
- **Path C** : Service push-based → Webhook trigger Tracecat

## Files

| File | Description |
|------|-------------|
| `SKILL.md` | Guide complet : process, auth patterns, layout rules, SOAR templates |
| `COMMON_MISTAKES.md` | 12 erreurs courantes |
| `EXAMPLES.md` | 5 exemples : VirusTotal, Wazuh, Splunk, CrowdStrike, TheHive |
| `README.md` | This file |
