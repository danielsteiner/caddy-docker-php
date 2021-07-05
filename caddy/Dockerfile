FROM caddy/caddy:2.3.0-builder-alpine AS builder
LABEL maintainer "Daniel Steiner <kontakt@danielsteiner.net>"
LABEL org.opencontainers.image.version=v2.3.0
LABEL org.opencontainers.image.title=Caddy
LABEL org.opencontainers.image.description="a powerful, enterprise-ready, open source web server with automatic HTTPS written in Go"
LABEL org.opencontainers.image.url=https://danielsteiner.net/docker/images/caddy
LABEL org.opencontainers.image.documentation=https://danielsteiner.net/docker/docs/caddy
LABEL org.opencontainers.image.vendor="Daniel Steiner Web- & Appentwicklung"
LABEL org.opencontainers.image.licenses=Apache-2.0
LABEL org.opencontainers.image.source="https://github.com/danielsteiner/caddy-docker-php"

ARG PUID="1000"
ARG PGID="1000"

RUN xcaddy build \
  --with github.com/abiosoft/caddy-exec \
  --with github.com/abiosoft/caddy-hmac \
  --with github.com/aksdb/caddy-cgi/v2 \
  --with github.com/caddy-dns/cloudflare \
  --with github.com/caddyserver/ntlm-transport \
  --with github.com/caddyserver/forwardproxy@03a7df4bf7e807cb04f22fb647a6bd5daeb386d9 \
  --with github.com/chukmunnlee/caddy-openapi \
  --with github.com/greenpau/caddy-auth-jwt \
  --with github.com/greenpau/caddy-auth-portal \
  --with github.com/greenpau/caddy-trace \
  --with github.com/hairyhenderson/caddy-teapot-module \
  --with github.com/lolPants/caddy-requestid \
  --with github.com/mholt/caddy-webdav \
  --with github.com/porech/caddy-maxmind-geolocation

FROM caddy/caddy:2.3.0-alpine
COPY --from=builder /usr/bin/caddy /usr/bin/caddy

RUN caddy version
RUN caddy list-modules

EXPOSE 80 443
WORKDIR /etc/caddy/

ENTRYPOINT ["caddy"]
CMD ["run", "--config", "/etc/caddy/Caddyfile", "--adapter", "caddyfile"]
