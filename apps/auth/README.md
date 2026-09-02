# Auth (Authentik + Kyverno SSO annotation policy)

Draft manifests only — nothing applied yet. Convention: any Ingress annotated
with `use-sso-auth: "true"` gets the Authentik forward-auth annotations injected
by the Kyverno ClusterPolicy (`auth/sso-auth-policy.yaml`).

## Deploy order

1. **Authentik** (`auth/authentik.yaml`) — IdP + embedded outpost. Needs a
   Postgres user/DB on postgresql.haven first (see below), plus secrets from
   Vaultwarden/ESO.
2. **Proxy provider** — create in Authentik UI (or via API) once it's up:
   one provider per app you want header-auth'd, mode "forward_auth (single
   app)", or a domain-level provider with the embedded outpost.
3. **Kyverno** (`auth/kyverno-helm.yaml`) — policy engine.
4. **SSO policy** (`auth/sso-auth-policy.yaml`) — expands `use-sso-auth: "true"`
   into `auth-url` / `auth-signin` / `auth-response-headers` annotations.
5. Annotate ingresses: `kubectl annotate ingress <name> -n <ns> use-sso-auth=true`
   and mirror the change in this repo.

## Postgres bootstrap (one-time, on postgresql.haven)

```sql
CREATE USER authentik WITH PASSWORD '<from-vaultwarden>';
CREATE DATABASE authentik OWNER authentik;
```

## Notes

- Outpost URL used everywhere is `https://auth.haven` — add DNS + ingress for
  the outpost before annotating anything.
- Apps that should *consume* the injected `X-Authentik-*` headers (Grafana auth
  proxy, Paperless remote-user, OpenWebUI trusted headers) need their own env
  changes; those are app-side, not covered by the policy.
- API/WebSocket-heavy apps (arr stack, qBittorrent) should NOT get the
  annotation on API paths — either skip the annotation or exclude paths.

## Which apps get the annotation (scoping)

Three tiers, applied per ingress — never blanket:

1. **OIDC tier** (no annotation, wire native OIDC in the app instead):
   grafana, paperless, affine, openwebui, vaultwarden, beszel, slink.
   Proper logout, group mapping, zero API breakage.
2. **Forward-auth tier** (`use-sso-auth: "true"`): browser-only utilities with
   no real auth — it-tools, notepad, searxng, homepage, archivebox,
   stirlingpdf, file-nginx, code-config, havenllo, own apps (chacal,
   mindforge) if wanted.
3. **Hands off** (built-in auth is fine, forward-auth breaks clients):
   sonarr, radarr, prowlarr, qbittorrent, adguard, changedetection,
   uptimekuma, cloudreve (WebDAV), jellyfin (clients can't do redirects —
   only the community SSO plugin route exists).

## Pre-annotation checklist (per new app)

Forward-auth breakage is loud and immediate if you look in the right places.
Before annotating any real hostname:

1. **Ask: what talks to this app that isn't a human in a browser?**
   Mobile/desktop apps with own login, API-key consumers, push/webhook
   receivers, cronjobs curling through the ingress, WebDAV/RSS → any of
   those = hands off (tier 3) or exclude paths. "Just me in a browser" =
   safe.
2. **Canary first:** create a scratch ingress for the same service
   (`<app>-test.haven`) with the annotation; leave the original untouched
   for ~a week. Forgotten integrations keep hitting the original and only
   break the canary — instant rollback
   (`kubectl annotate ingress <n> -n <ns> use-sso-auth-`).
3. **Grep the docs** (30s): "reverse proxy", "trusted header", "API key",
   "webhook", "basic auth". A "running behind a reverse proxy" doc section
   is where landmines are documented.
4. **Header-trust hygiene:** apps consuming `X-Authentik-*` must be
   reachable ONLY through the ingress (no NodePort/exposed port, otherwise
   LAN clients can forge headers straight to the pod). Keep built-in local
   login enabled; never set SSO-only mode.

## Files

- `kyverno-helm.yaml` — Kyverno install values
- `sso-auth-policy.yaml` — ClusterPolicy: `use-sso-auth` → nginx auth annotations
- `authentik.yaml` — Authentik server/worker + outpost, Postgres PVC, ingress

## Status / when to actually deploy this (decision from 2026-08-28)

NOT deployed — deliberate. Sole user, low login frequency, everything
reachable via wg-easy, so SSO solves a problem that doesn't exist. The
`*.ivanch.me` public ingresses were reviewed instead: exposure without usage
is pure risk, and the right fix is removing exposure (VPN-only), not adding
an IdP.

Pull the trigger only if one of these becomes true:

1. A second person uses the homelab regularly (SSO value scales with users).
2. Something must be exposed that can't sit behind VPN (third-party
   webhooks/callbacks) — then public tier gets `use-sso-auth` + 2FA.
3. Login friction on the OIDC apps (grafana/paperless/affine/openwebui/
   vaultwarden/beszel/slink) actually annoys on a weekly basis.

Until then this folder is a shelf-ready draft; cost of keeping it is zero.
