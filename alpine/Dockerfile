FROM debian:bullseye

ENV DEBIAN_FRONTEND noninteractive
ENV DEBCONF_NONINTERACTIVE_SEEN true

RUN set -x \
  && apt-get update \
  && apt-get install -y --no-install-recommends apt-transport-https ca-certificates curl

RUN set -x \
  && curl -fsSL -O https://github.com/rxi/json.lua/raw/master/json.lua \
  && curl -fsSL -O https://github.com/haproxytech/haproxy-lua-http/raw/master/http.lua \
  && curl -fsSL -O https://github.com/TimWolla/haproxy-auth-request/raw/main/auth-request.lua



FROM {BASE_IMAGE}

LABEL org.opencontainers.image.source https://github.com/voyagermesh/haproxy

# Installs required packages
# Change timezone to UTC
RUN set -x \
  && apk add --update --no-cache ca-certificates tzdata lua5.3 lua-socket lua-ossl libcap \
  && echo 'Etc/UTC' > /etc/timezone \
  && setcap 'cap_net_bind_service=+ep' $(which haproxy)

# CVE-2022-1586 CVE-2022-1587
RUN set -x \
  && apk add --update --upgrade --no-cache pcre2

# CVE-2022-37434
RUN set -x \
  && apk add --update --upgrade --no-cache zlib

# CVE-2022-41723
RUN set -x \
  && rm -rf /usr/local/bin/dataplaneapi

ENV TZ     :/etc/localtime
ENV LANG   en_US.utf8

COPY --from=0 /json.lua         /usr/share/lua/5.3/json.lua
COPY --from=0 /http.lua         /usr/share/haproxy/haproxy-lua-http/http.lua
COPY --from=0 /auth-request.lua /usr/share/haproxy/auth-request.lua
