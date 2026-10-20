## 0.4.1 (2026-10-21)

### Bug fixes

- Outbound HTTPS (OIDC discovery, JWKS, GitHub API) now trusts the system CA store instead of the `webpki-roots` set compiled into the binary, so CA distrust such as the [eMudhra intermediate compromise](https://emudhra.we-have-a-problem.com/) follows OS CA updates. [#1](https://github.com/sorah/sts-cat/pull/1)
- rustls-webpki rolled to 0.103.16, which rejects certificate paths through the compromised eMudhra intermediates. [#1](https://github.com/sorah/sts-cat/pull/1)

## 0.4.0 (2026-05-12)

### Enhancements

- `--policy-ignored-repo {org}/{repo}` option was added to ignore repo-level policies from certain repositories.

## 0.3.0 (2026-03-29)

### Enhancements

- Trust policy gains `max_token_lifetime` option to reject OIDC tokens whose lifetime (`exp - nbf` or `exp - iat`) exceeds a specified number of seconds

## 0.2.0 (2026-03-29)

### Enhancements

- Tracing
    - Log `remote_addr` and `xff` (X-Forwarded-For) on all exchange events (`exchange_denied`, `exchange_authorized`, `exchange_success`)
    - Add `tower_http::trace::TraceLayer` with request span logging at INFO level, including `method`, `uri`, `remote_addr`, and `xff`
    - Add `#[tracing::instrument]` spans to internal I/O methods (OIDC discovery, JWKS fetch, GitHub API calls, KMS signing)
    - Lambda binary: use `lambda_http::tracing` subscriber instead of custom `init_tracing`; set `AWS_LAMBDA_LOG_FORMAT=JSON` when `--log-json` is given
    - Lambda binary: inject `source_ip` from Lambda request context (API Gateway V2/V1, ALB) into `X-Forwarded-For` header

## 0.1.0 (2026-03-29)

- Initial release.
