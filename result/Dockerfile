FROM node:18.20.4-bookworm-slim

# Add curl for healthcheck, tini, and libcap2-bin for privileged port binding
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl tini libcap2-bin && \
    setcap 'cap_net_bind_service=+ep' $(readlink -f $(which node)) && \
    rm -rf /var/lib/apt/lists/*

# Create dedicated non-root user and group (UID/GID 10001)
RUN groupadd -g 10001 appgroup && \
    useradd -u 10001 -g appgroup -s /bin/sh -d /usr/local/app appuser

WORKDIR /usr/local/app

# Have nodemon available for local dev use (file watching)
RUN npm install -g nodemon

COPY package*.json ./

RUN npm ci && \
    npm cache clean --force && \
    mv /usr/local/app/node_modules /node_modules

COPY --chown=10001:10001 . .

RUN chown -R 10001:10001 /usr/local/app /node_modules

ENV PORT=80
EXPOSE 80

USER 10001

ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["node", "server.js"]
