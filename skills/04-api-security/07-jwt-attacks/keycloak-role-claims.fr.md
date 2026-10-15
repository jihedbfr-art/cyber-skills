# Valider les rôles Keycloak avant de créer les autorités

[English version](keycloak-role-claims.md)

Un convertisseur de rôles doit distinguer un nom de rôle d'une valeur JSON quelconque. Dans le toolkit Spring testé, `String.valueOf` transformait `123`, `true` et `null` en `ROLE_123`, `ROLE_TRUE` et `ROLE_NULL`. Un claim d'accès contenant une liste au lieu d'un objet provoquait une `IllegalArgumentException` avant la fin de la conversion.

## Tester avec des claims synthétiques

Exécuter ces contrôles sur le convertisseur dans des tests unitaires. Les entrées ci-dessous sont des fragments de claims synthétiques, pas des jetons signés. Aucun véritable jeton d'accès n'est nécessaire.

| Entrée | Résultat attendu du convertisseur |
|---|---|
| `{"realm_access":{"roles":["viewer",123,true,null," "]}}` | Seulement `ROLE_VIEWER` |
| `{"resource_access":{"my-client":{"roles":["editor",456]}}}` | Seulement `ROLE_EDITOR` si `my-client` est configuré |
| `{"realm_access":["admin"],"resource_access":"unexpected"}` | Aucune autorité de rôle et aucune exception de conversion |
| `{"realm_access":"unexpected","resource_access":{"my-client":{"roles":["editor"]}}}` | `ROLE_EDITOR` est conservé malgré le claim realm malformé |

Ne pas convertir les valeurs non textuelles en noms de rôles. Ignorer les chaînes vides. Vérifier qu'un claim d'accès est une map avant de lire sa liste `roles`, et limiter l'extraction au client configuré. Cette politique ignore les données de rôles malformées ; une application peut choisir de rejeter explicitement le jeton dans sa couche de validation.

## Rejouer les tests vérifiés

Les deux correctifs sont fusionnés dans la branche `main` du toolkit. Dans un nouveau clone :

```bash
git clone https://github.com/jihedbfr-art/spring-keycloak-toolkit.git
cd spring-keycloak-toolkit
mvn -q -Dtest=KeycloakRealmRoleConverterTest test
```

Un correctif teste les entrées invalides des listes de rôles ; l'autre teste les objets d'accès malformés. Pour vérifier les deux comportements dans votre propre implémentation, conserver les trois tests de régression : `ignoresNonStringAndBlankRoleValues`, `ignoresMalformedAccessClaims` et `keepsValidResourceRolesWhenRealmClaimIsMalformed`.

## Preuves et portée

Les nouvelles assertions échouaient avant chaque correction. Le 15 octobre 2026, l'implémentation fusionnée a passé 27 tests locaux, dont l'intégration Keycloak 26.7.1 sous Docker, avec Spring Boot 4.1.0 et Java 17. Cela vérifie les chemins de conversion et d'intégration exercés. Ce résultat ne démontre pas qu'un attaquant peut émettre un jeton accepté, et ne remplace pas la validation de la signature, de l'issuer, de l'audience et de la durée de validité.

Sources : [correction et test des entrées de rôles](https://github.com/jihedbfr-art/spring-keycloak-toolkit/commit/6026018), [correction des claims d'accès](https://github.com/jihedbfr-art/spring-keycloak-toolkit/commit/7055489), [PR #10](https://github.com/jihedbfr-art/spring-keycloak-toolkit/pull/10), [PR #11](https://github.com/jihedbfr-art/spring-keycloak-toolkit/pull/11).

---

<div align="center">
  <img src="../../../assets/brand/jihedailabs-logo.svg" alt="JihedAiLabs" width="120">
  <br>
  <sub>Un projet <a href="https://github.com/jihedbfr-art"><b>JihedAiLabs</b></a></sub>
</div>
