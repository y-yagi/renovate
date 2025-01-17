ARG BASE_IMAGE_TYPE=slim

# --------------------------------------
# slim image
# --------------------------------------
FROM ghcr.io/renovatebot/base-image:2.0.1@sha256:30fb6614d638f7ed887ae19b53d7d81e88b475b7b67c830cbb07a87eee82fa61 AS slim-base

# --------------------------------------
# full image
# --------------------------------------
FROM ghcr.io/renovatebot/base-image:2.0.1-full@sha256:61c6feb57a8d8a8b7e5d36fa6eaf86c07ade3e43c2cde91e99e483415d226c28 AS full-base

# --------------------------------------
# build image
# --------------------------------------
FROM slim-base as build

WORKDIR /usr/local/renovate

ENV CI=1 npm_config_modules_cache_max_age=0
ENV RE2_DOWNLOAD_MIRROR=https://github.com/containerbase/node-re2-prebuild/releases/download RE2_DOWNLOAD_SKIP_PATH=1

COPY pnpm-lock.yaml ./

# only fetch deps from lockfile https://pnpm.io/cli/fetch
RUN pnpm fetch --prod

COPY . ./

# install
RUN set -ex; \
  pnpm install --prod --offline --ignore-scripts; \
  npm explore re2 -- npm run install; \
  true

# test
COPY tools/docker/bin/ /usr/local/bin/
RUN set -ex; \
  renovate --version; \
  renovate-config-validator; \
  node -e "new require('re2')('.*').exec('test')"; \
  true

# --------------------------------------
# final image
# --------------------------------------
FROM ${BASE_IMAGE_TYPE}-base

LABEL name="renovate"
LABEL org.opencontainers.image.source="https://github.com/renovatebot/renovate" \
  org.opencontainers.image.url="https://renovatebot.com" \
  org.opencontainers.image.licenses="AGPL-3.0-only"

WORKDIR /usr/src/app

ENV RENOVATE_X_IGNORE_NODE_WARN=true

COPY tools/docker/bin/ /usr/local/bin/
CMD ["renovate"]

ARG RENOVATE_VERSION

COPY --from=build --chown=root:root /usr/local/renovate/ /usr/local/renovate/

# Compabillity, so `config.js` can access renovate and deps
RUN set -ex; \
  mkdir /opt/containerbase/tools/renovate; \
  echo "${RENOVATE_VERSION}" > /opt/containerbase/versions/renovate; \
  ln -sf /usr/local/renovate /opt/containerbase/tools/renovate/${RENOVATE_VERSION}; \
  ln -sf /usr/local/renovate/node_modules ./node_modules; \
  true

RUN set -ex; \
  renovate --version; \
  renovate-config-validator; \
  node -e "new require('re2')('.*').exec('test')"; \
  true

LABEL \
  org.opencontainers.image.version="${RENOVATE_VERSION}" \
  org.label-schema.version="${RENOVATE_VERSION}"

# Numeric user ID for the ubuntu user. Used to indicate a non-root user to OpenShift
USER 1000
