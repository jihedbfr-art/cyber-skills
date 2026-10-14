# Validate Keycloak role claims before granting authorities

[Version française](keycloak-role-claims.fr.md)

A role converter must distinguish a role name from an arbitrary JSON value. In the tested Spring toolkit, `String.valueOf` converted `123`, `true` and `null` into `ROLE_123`, `ROLE_TRUE` and `ROLE_NULL`. An access claim containing a list instead of an object raised `IllegalArgumentException` before conversion completed.

## Test with synthetic claims

Run these checks against your converter in unit tests. The inputs below are synthetic claim fragments, not signed tokens. No real access token is needed.

| Input | Expected converter result |
|---|---|
| `{"realm_access":{"roles":["viewer",123,true,null," "]}}` | Only `ROLE_VIEWER` |
| `{"resource_access":{"my-client":{"roles":["editor",456]}}}` | Only `ROLE_EDITOR` when `my-client` is configured |
| `{"realm_access":["admin"],"resource_access":"unexpected"}` | No role authority and no conversion exception |
| `{"realm_access":"unexpected","resource_access":{"my-client":{"roles":["editor"]}}}` | `ROLE_EDITOR` survives the malformed realm claim |

Do not coerce non-string values to role names. Ignore blank strings. Check that an access claim is a map before reading its `roles` list, and keep extraction limited to the configured client. This policy ignores malformed role data; an application may instead reject the token explicitly through its validation layer.

## Replay the verified tests

Use a new checkout for this procedure. The two snapshots are proposed changes, not releases.

```bash
git clone https://github.com/jihedbfr-art/spring-keycloak-toolkit.git
cd spring-keycloak-toolkit
git checkout 6026018
mvn -q -Dtest=KeycloakRealmRoleConverterTest test
git checkout 920441e
mvn -q -Dtest=KeycloakRealmRoleConverterTest test
```

The first snapshot tests invalid entries inside role lists; the second tests malformed access objects. They are separate branches. To validate both behaviors in your implementation, retain all three regression tests: `ignoresNonStringAndBlankRoleValues`, `ignoresMalformedAccessClaims` and `keepsValidResourceRolesWhenRealmClaimIsMalformed`.

## Evidence and scope

The new assertions failed before each fix. On 14 October 2026, the combined implementation passed 25 local tests, including Keycloak 26.7.1 under Docker, with Spring Boot 4.1.0 and Java 17. This verifies the exercised conversion and integration paths. It does not establish that an attacker can issue a trusted token, or replace signature, issuer, audience and lifetime validation.

Sources: [role-entry fix and test](https://github.com/jihedbfr-art/spring-keycloak-toolkit/commit/6026018), [access-claim fix](https://github.com/jihedbfr-art/spring-keycloak-toolkit/commit/7055489), [PR #10](https://github.com/jihedbfr-art/spring-keycloak-toolkit/pull/10), [PR #11](https://github.com/jihedbfr-art/spring-keycloak-toolkit/pull/11).

---

<div align="center">
  <img src="../../../assets/brand/jihedailabs-logo.svg" alt="JihedAiLabs" width="120">
  <br>
  <sub>A <a href="https://github.com/jihedbfr-art"><b>JihedAiLabs</b></a> project</sub>
</div>
