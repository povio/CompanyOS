# Povio: restore the E2B sandbox provider on CompanyOS v0.15.1

This branch is Povio's overlay on upstream tag `v0.15.1` (`e512f21b18`). It is
not intended for a pull request to `kortix-ai/CompanyOS`.

Upstream removed the E2B provider in
[kortix-ai/CompanyOS#10310](https://github.com/kortix-ai/CompanyOS/pull/10310).
This patch puts that provider back from commit `ce0b0a27`, plus two Povio
additions that the deployed image uses:

- Install `e2b@2.37.0` and apply `povio/e2b@2.37.0.patch` so template file
  uploads work under Bun (use `Bun.file()` and send an empty `Content-Type`).
- New optional `E2B_SANDBOX_URL`, so all sandbox HTTP traffic can go through
  one TLS hostname instead of per-sandbox `*.<E2B_DOMAIN>` names.

The resulting API source on this branch matches the overlay baked into the
image `kortix/kortix-api:v0.15.1-e2b`.

## Environment

Set these on the Kortix API (and keep `KORTIX_AUTO_UPDATE=false` so the
official `stable` tag cannot overwrite the custom image):

| Variable | Purpose |
|---|---|
| `ALLOWED_SANDBOX_PROVIDERS=e2b` | Enable only the E2B provider. |
| `E2B_API_KEY` | E2B team API key. Required when `e2b` is allowed. Do not commit this value. |
| `E2B_DOMAIN` | Base domain without a protocol. API calls go to `https://api.<domain>`. Defaults to `e2b.dev`. For a self-hosted cluster use that cluster's base domain (for Povio: `e2b.povio.site`). |
| `E2B_SANDBOX_URL` | Optional absolute URL for sandbox ingress (no trailing slash). When set, all sandbox traffic uses this one host, with `E2b-Sandbox-Id` / `E2b-Sandbox-Port` headers. Example: `https://sandbox.e2b.povio.site`. |
| `E2B_TEMPLATE` | Optional default E2B template id if a session does not pass a snapshot. |
| `KORTIX_AUTO_UPDATE=false` | Disable the Kortix image auto-updater. |

`E2B_API_KEY` must come from the environment or a secret store. Never add it
to this repo, a Dockerfile, or a compose file that is committed.

Optional tunables already in the restored provider (defaults are fine):
`KORTIX_E2B_RUNTIME_BACKSTOP_MS`, `KORTIX_E2B_REMOVE_TIMEOUT_MS`,
`KORTIX_E2B_RENEW_TIMEOUT_MS`, `KORTIX_E2B_RENEWAL_TOLERANCE_MS`,
`KORTIX_E2B_STOP_TIMEOUT_MS`.

## Build the image

The image is based on `kortix/kortix-api:v0.15.1-base` (a local tag of the
exact `v0.15.1` API image that was running when the overlay was built).

From the repository root, on this branch:

```sh
docker build -f povio/Dockerfile -t kortix/kortix-api:v0.15.1-e2b .
```

Then point the Kortix API service at that tag (`API_IMAGE=kortix/kortix-api:v0.15.1-e2b`)
and recreate the API container. Confirm the API banner reports `Providers: e2b`.

## Reapply on a newer upstream tag

Kortix upgrades are manual while this overlay is in use.

1. Fork or fetch the newer upstream tag (for example `v0.15.2`) into this
   repository.
2. Create a branch from that tag, e.g. `povio/e2b-v0.15.2`.
3. Reapply the source restore. Start from the patch that produced this branch
   (`git diff povio/v0.15.1...povio/e2b-v0.15.1`, excluding `povio/`).
   Prefer `git apply --3way` and fix any type or hook drift by hand.
4. Keep the `e2b@2.37.0` SDK + Bun upload patch unless the base image already
   has a working E2B SDK on Bun.
5. Keep the `E2B_SANDBOX_URL` option if it is still absent upstream.
6. Update `povio/Dockerfile` so `FROM` matches the new base image tag, build,
   typecheck (`tsc` in `apps/api`, ignoring pre-existing test-file errors),
   and smoke-test a sandbox create against the E2B cluster.

## Rollback

To put the API back on the official `v0.15.1` / `stable` image and the previous
sandbox provider:

1. Restore the previous API image tag and env (including
   `ALLOWED_SANDBOX_PROVIDERS` and any provider keys that provider needs).
2. Recreate the API container from that image. Do not leave
   `ALLOWED_SANDBOX_PROVIDERS=e2b` pointed at an image that no longer contains
   the provider.

This document covers only the CompanyOS / Kortix API overlay. Cluster-level
E2B Embed rollback lives with the host install, not in this repository.
