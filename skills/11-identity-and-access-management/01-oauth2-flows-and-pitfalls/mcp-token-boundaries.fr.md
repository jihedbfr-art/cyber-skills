# Tester la frontière des jetons d'un serveur MCP HTTP

[English version](mcp-token-boundaries.md)

Périmètre : spécification d'autorisation MCP 2025-11-25. Le serveur doit accepter des access tokens destinés à son propre usage. Un ID token OIDC ne les remplace pas. Une signature valide ne suffit pas à établir le destinataire prévu. Voir l'[autorisation MCP](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization#token-audience-binding-and-validation).

## Préparer un test autorisé

Utiliser un environnement jetable et des jetons émis par son serveur d'autorisation. Consigner l'identifiant de ressource attendu, l'issuer et la version du protocole. Obtenir un jeton correctement signé pour une autre ressource ; modifier `aud` dans un JWT existant invalide sa signature et teste un autre défaut.

| Cas | Preuve attendue |
|---|---|
| Jeton valide destiné à ce serveur MCP | Initialisation réussie ; droits des outils vérifiés séparément |
| Même issuer de confiance, jeton destiné uniquement à une autre ressource | Initialisation refusée avec HTTP 401 |
| Jeton expiré ou issuer inattendu | Initialisation refusée ; consigner la validation qui échoue |
| Jeton entrant reçu à l'identique par une API simulée en aval | Frontière violée ; ne conserver que le résultat booléen de comparaison |

## Sonder un cas négatif sans afficher le jeton

Ce modèle Bash attend `MCP_ENDPOINT` et `TEST_ACCESS_TOKEN` dans l'environnement. Utiliser le jeton de test de mauvaise audience, un endpoint autorisé et une version de protocole prise en charge par la cible. L'exemple vise 2025-11-25. La configuration est transmise sur stdin et seul le statut HTTP est affiché ; ne pas ajouter de trace verbeuse ni conserver les headers de la requête.

```bash
set +x
: "${MCP_ENDPOINT:?set an authorized test endpoint}"
: "${TEST_ACCESS_TOKEN:?set a test access token}"
curl --silent --show-error --max-time 10 --output /dev/null --write-out '%{http_code}\n' --config - <<EOF
url = "$MCP_ENDPOINT"
request = "POST"
header = "Authorization: Bearer $TEST_ACCESS_TOKEN"
header = "Content-Type: application/json"
header = "Accept: application/json, text/event-stream"
data = "{\"jsonrpc\":\"2.0\",\"id\":1,\"method\":\"initialize\",\"params\":{\"protocolVersion\":\"2025-11-25\",\"capabilities\":{},\"clientInfo\":{\"name\":\"audience-probe\",\"version\":\"1\"}}}"
EOF
```

Un timeout ou un serveur injoignable ne constitue pas un rejet réussi. Établir d'abord le cas témoin avec un jeton valide et un client MCP. Ne conserver dans le rapport que le statut, le nom du cas, la référence du build et le résultat de validation.

## Contrôler la frontière aval et corriger

Utiliser une API simulée qui compare en interne la valeur d'autorisation reçue au jeton de test et ne retourne que leur égalité. Le jeton MCP entrant ne doit pas être transmis tel quel. Configurer un identifiant aval distinct, avec l'audience et les droits attendus ; l'échange de jetons est une conception possible, pas une fonctionnalité implémentée par cette procédure. Voir les [consignes sur la transmission des jetons](https://modelcontextprotocol.io/docs/2025-11-25/tutorials/security/security_best_practices#token-passthrough).

Imposer le contrôle du destinataire dans le resource server avant l'invocation, puis tester les rôles au niveau de l'outil. Ces contrôles sont indépendants. Cette procédure est un modèle d'évaluation ; elle n'affirme pas que MCP Guard passe actuellement les tests d'audience ou de jetons aval.

---

<div align="center">
  <img src="../../../assets/brand/jihedailabs-logo.svg" alt="JihedAiLabs" width="120">
  <br>
  <sub>Un projet <a href="https://github.com/jihedbfr-art"><b>JihedAiLabs</b></a></sub>
</div>
