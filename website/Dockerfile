FROM --platform=${BUILDPLATFORM} docker.io/library/node:26.3.0-trixie@sha256:e3ffe0cbaeebdcddbfe1ee7bca9b564a92863a8386d5b99a3d72677b3667b61d AS docs-builder

ENV NODE_ENV=production

WORKDIR /work

# Bootstrap pnpm via the version pinned in the root package.json. Node ships
# with npm; we use it once here to land pnpm on the PATH, after which pnpm's
# `packageManager` self-bootstrap takes over.
RUN --mount=type=bind,target=/work/package.json,src=./package.json \
    --mount=type=bind,target=/work/scripts/node/,src=./scripts/node/ \
    --mount=type=bind,target=/work/packages/logger-js/,src=./packages/logger-js/ \
    --mount=type=bind,target=/work/website/package.json,src=./website/package.json \
    npm install -g "pnpm@$(node -p 'require(\"./package.json\").packageManager.split(\"@\")[1].split(\"+\")[0]')" && \
    node ./scripts/node/lint-runtime.mjs ./website

RUN --mount=type=bind,target=/work/.npmrc,src=./.npmrc \
    --mount=type=bind,target=/work/packages/docusaurus-config/,src=./packages/docusaurus-config/ \
    --mount=type=bind,target=/work/website/package.json,src=./website/package.json \
    --mount=type=bind,target=/work/website/pnpm-lock.yaml,src=./website/pnpm-lock.yaml \
    --mount=type=bind,target=/work/website/pnpm-workspace.yaml,src=./website/pnpm-workspace.yaml \
    --mount=type=bind,target=/work/website/vendored/detect-package-manager,src=./website/vendored/detect-package-manager \
    --mount=type=bind,target=/work/website/docusaurus-theme/package.json,src=./website/docusaurus-theme/package.json \
    --mount=type=bind,target=/work/website/api/package.json,src=./website/api/package.json \
    --mount=type=bind,target=/work/website/integrations/package.json,src=./website/integrations/package.json \
    --mount=type=bind,target=/work/website/docs/package.json,src=./website/docs/package.json \
    --mount=type=cache,id=pnpm-website,sharing=shared,target=/root/.local/share/pnpm/store \
    pnpm --dir website install --frozen-lockfile

WORKDIR /work/website

COPY ./website /work/website/
COPY ./blueprints /work/blueprints/
COPY ./schema.yml /work/
COPY ./lifecycle/container/compose.yml /work/lifecycle/container/
COPY ./SECURITY.md /work/

RUN  --mount=type=bind,target=/work/packages/docusaurus-config/,src=./packages/docusaurus-config/ \
    pnpm run build

FROM docker.io/library/nginx:1.31.1-trixie@sha256:5aca99593157f4ae539a5dec1092a0ad8762f8e2eb1789085a13a0f5622369f6
LABEL org.opencontainers.image.authors="Authentik Security Inc." \
    org.opencontainers.image.source="https://github.com/goauthentik/authentik" \
    org.opencontainers.image.description="authentik product documentation" \
    org.opencontainers.image.documentation="https://docs.goauthentik.io" \
    org.opencontainers.image.licenses="https://github.com/goauthentik/authentik/blob/main/LICENSE" \
    org.opencontainers.image.title="authentik docs image" \
    org.opencontainers.image.url="https://goauthentik.io" \
    org.opencontainers.image.vendor="Authentik Security Inc."


COPY --from=docs-builder /work/website/docs/build /usr/share/nginx/html
