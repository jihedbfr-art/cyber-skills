# Test the token boundary of an HTTP MCP server

[Version française](mcp-token-boundaries.fr.md)

Scope: MCP authorization specification 2025-11-25. The server must accept access tokens intended for itself. An OIDC ID token is not a substitute. A valid signature alone does not establish the intended recipient. See [MCP authorization](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization#token-audience-binding-and-validation).

## Prepare an authorized test

Use a disposable environment and tokens issued by its authorization server. Record the expected resource identifier, issuer and protocol version. Obtain a correctly signed token for another resource; editing `aud` in an existing JWT invalidates its signature and tests a different failure.

| Case | Expected evidence |
|---|---|
| Valid token intended for this MCP server | Initialization succeeds; tool permissions checked separately |
| Same trusted issuer, token intended only for another resource | Initialization rejected with HTTP 401 |
| Expired token or unexpected issuer | Initialization rejected; record which validation failed |
| Inbound token reaches a downstream mock unchanged | Boundary violation; retain only a boolean comparison result |

## Probe a negative case without printing the token

This Bash template requires `MCP_ENDPOINT` and `TEST_ACCESS_TOKEN` in the environment. Use the wrong-audience test token, an authorized endpoint and the protocol version supported by that target. The example targets 2025-11-25. It sends configuration on stdin and prints only HTTP status; do not add verbose tracing or persist the request headers.

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

A timeout or unreachable server is not a successful rejection. Establish the valid-token control using an MCP client first. Keep only status, case label, build reference and the validation result in the report.

## Check the downstream boundary and fix

Use a mock API that compares its received authorization value with the test token internally and reports only equality. The inbound MCP token must not be forwarded unchanged. Configure a separate downstream credential with the intended audience and permissions; token exchange is one possible design, not implemented by this procedure. See the [token-passthrough guidance](https://modelcontextprotocol.io/docs/2025-11-25/tutorials/security/security_best_practices#token-passthrough).

Enforce recipient validation in the resource server before dispatch, then test roles at the tool boundary. These are independent checks. This procedure is an assessment template; it does not claim that MCP Guard currently passes audience or downstream-token tests.

---

<div align="center">
  <img src="../../../assets/brand/jihedailabs-logo.svg" alt="JihedAiLabs" width="120">
  <br>
  <sub>A <a href="https://github.com/jihedbfr-art"><b>JihedAiLabs</b></a> project</sub>
</div>
