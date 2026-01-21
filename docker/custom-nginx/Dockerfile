FROM ubuntu:22.04 as source
RUN apt update && apt install -y curl git \
  && git clone --depth 1 https://gitlab.com/mikroways/public/mikroways.net.git \
      /tmp/www \
  && curl -sL https://github.com/gohugoio/hugo/releases/download/v0.82.0/hugo_extended_0.82.0_Linux-64bit.tar.gz \
      | tar xzf - -C /usr/local/bin hugo \
  && cd /tmp/www \
  && hugo

FROM nginx:1.23-alpine
COPY --from=source /tmp/www/public /usr/share/nginx/html
