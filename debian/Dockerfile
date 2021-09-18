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

# Installs required packages
# Change timezone to UTC
RUN set -x \
  && apt-get update \
  && apt-get install -y --no-install-recommends ca-certificates lua5.3 lua-socket lua-luaossl \
  && rm -rf /var/lib/apt/lists/* /usr/share/doc /usr/share/man /tmp/* \
  && echo 'Etc/UTC' > /etc/timezone

ENV TZ     :/etc/localtime
ENV LANG   en_US.utf8

COPY --from=0 /json.lua         /usr/share/lua/5.3/json.lua
COPY --from=0 /http.lua         /usr/share/haproxy/haproxy-lua-http/http.lua
COPY --from=0 /auth-request.lua /usr/share/haproxy/auth-request.lua
