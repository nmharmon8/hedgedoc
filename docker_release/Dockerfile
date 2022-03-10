FROM node:16.14.0-slim@sha256:3f34fa94510e16bf619fefb53c9c5d2b11ead626863bdb6b26b49d38d1b56db8 AS base

FROM base AS builder

RUN apt-get update && apt-get install --no-install-recommends -y git jq ca-certificates

RUN git clone -b blog https://github.com/nmharmon8/hedgedoc.git

WORKDIR /hedgedoc

RUN yarn install --production=false --pure-lockfile
RUN yarn run build
RUN yarn install --production=true --pure-lockfile

FROM base

LABEL org.opencontainers.image.source="https://github.com/nmharmon8/hedgedoc"

ARG UID=10000
ENV NODE_ENV=production
ENV UPLOADS_MODE=0700

RUN apt-get update && \
    apt-get install --no-install-recommends -y gosu && \
    rm -r /var/lib/apt/lists/*

# Create hedgedoc user
RUN adduser --uid $UID --home /hedgedoc/ --disabled-password --system hedgedoc

COPY --chown=$UID --from=builder /hedgedoc /hedgedoc


# For backwards compatibility
RUN ln -s /hedgedoc /codimd

WORKDIR /hedgedoc
EXPOSE 3000

COPY ["docker-entrypoint.sh", "/usr/local/bin/docker-entrypoint.sh"]
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]

CMD ["node", "app.js"]