# Common Mistakes — Integration Builder

## 1. Ne pas rechercher l'API avant de construire
**Wrong:** Commencer à créer des actions sans comprendre l'API du service
**Fix:** Toujours fetch la doc (WebFetch), chercher les endpoints, comprendre l'auth AVANT de créer quoi que ce soit dans Tracecat

## 2. Oublier de demander les credentials à l'utilisateur
**Wrong:** Inventer ou deviner des API keys
**Fix:** Toujours demander explicitement à l'utilisateur ses credentials, puis les créer via `tracecat_create_secret`

## 3. Hardcoder des credentials dans les inputs
**Wrong:**
```yaml
headers:
  Authorization: Bearer sk-abc123456
```
**Fix:**
```yaml
headers:
  Authorization: Bearer ${{ SECRETS.service.API_KEY }}
```

## 4. Créer des actions orphelines (pas d'edges)
**Wrong:** Créer les actions sans les connecter au trigger et entre elles
**Fix:** Toujours appeler `tracecat_add_edges` après création des actions + `tracecat_move_nodes` pour le layout

## 5. Oublier le positionnement des nœuds
**Wrong:** Actions créées mais empilées à (0,0) dans l'UI
**Fix:** Toujours positionner : trigger à (500,0), première action à (500,300), espacement Y=160px

## 6. Utiliser core.http_request quand une intégration native existe
**Wrong:** Construire un appel HTTP manuel pour VirusTotal
**Fix:** Vérifier d'abord si `tools.virustotal.*` existe nativement. Utiliser `tracecat_list_templates` pour voir les intégrations disponibles

## 7. Mauvais format d'auth pour le service
**Wrong:** Utiliser Bearer token pour un service qui attend un API key en header custom
**Fix:** Toujours lire la doc d'auth du service. Exemples :
- VirusTotal : `x-apikey: <key>` (header custom)
- CrowdStrike : `Authorization: Bearer <token>` (OAuth2)
- Splunk : `Authorization: Bearer <token>` (session token)
- Wazuh : Basic auth → JWT → Bearer pour les requêtes suivantes

## 8. Ne pas gérer le refresh de token OAuth2
**Wrong:** Stocker un access_token statique pour CrowdStrike/Microsoft
**Fix:** Créer une action d'auth en première étape du workflow qui obtient un fresh token, puis utiliser `${{ ACTIONS.auth.result.access_token }}` dans les actions suivantes

## 9. Oublier verify_ssl pour les services self-hosted
**Wrong:** Appeler un Wazuh/Splunk self-hosted en HTTPS avec certificat auto-signé → erreur SSL
**Fix:** Ajouter `verify_ssl: false` dans les inputs de `core.http_request` pour les services internes

## 10. Convertir un spec OpenAPI entier sans filtrer
**Wrong:** Générer des templates pour les 500 endpoints de Microsoft Graph
**Fix:** Utiliser le config YAML avec `endpoints.include` pour ne garder que les endpoints SOAR-pertinents

## 11. Inputs en JSON au lieu de YAML string dans le MCP
**Wrong:**
```
tracecat_update_action:
  inputs: { "url": "https://...", "method": "GET" }
```
**Fix:**
```
tracecat_update_action:
  inputs: "url: https://...\nmethod: GET\nheaders:\n  Authorization: Bearer ${{ SECRETS.x.KEY }}"
```

## 12. Ne pas tester avec un dry run
**Wrong:** Déployer directement en production sans vérifier
**Fix:** Toujours : `tracecat_validate_workflow` → `tracecat_run_workflow` (test) → `tracecat_get_execution_compact` (vérifier) → puis deploy
