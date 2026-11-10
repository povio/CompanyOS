# Kortix API v0.15.1 + restored E2B provider.
# The deployed image kortix/kortix-api:v0.15.1-e2b was built on
# kortix/kortix-api:v0.15.1-base (the v0.15.1/stable image as of 2026-11-06).
#
# Build from the repository root, on this branch:
#   docker build -f povio/Dockerfile -t kortix/kortix-api:v0.15.1-e2b .
FROM kortix/kortix-api:v0.15.1-base
USER root
COPY povio/e2b@2.37.0.patch /tmp/e2b.patch
RUN set -eux; mkdir -p /tmp/e2bdeps && cd /tmp/e2bdeps && echo '{"name":"e2bdeps","private":true}' > package.json \
 && bun add e2b@2.37.0 \
 && rm -rf /app/node_modules/e2b && cp -a node_modules/e2b /app/node_modules/e2b \
 && mkdir -p /app/node_modules/e2b/node_modules \
 && for d in node_modules/*; do [ "$d" = node_modules/e2b ] && continue; cp -a "$d" /app/node_modules/e2b/node_modules/; done \
 && cd /app/node_modules/e2b && (command -v patch >/dev/null || (apt-get update && apt-get install -y --no-install-recommends patch)) && patch -p1 < /tmp/e2b.patch \
 && rm -rf /tmp/e2bdeps /tmp/e2b.patch
# The files on this branch are v0.15.1 plus the E2B restore. Copying the full
# src trees is equivalent to the original overlay (unchanged files match the base).
COPY apps/api/src/ /app/apps/api/src/
COPY packages/api-contract/src/ /app/packages/api-contract/src/
LABEL org.opencontainers.image.title="kortix-api (v0.15.1 + E2B restored)" povio.e2b-overlay="2026-11-09"
