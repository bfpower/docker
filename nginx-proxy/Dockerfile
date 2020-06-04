FROM nginx:1.19.0-alpine

### Start by pulling in the conf files
WORKDIR /etc/nginx/conf.d
COPY ./*.conf .

### Set the certbot version with an arg
### We'll default to 1.4.0, which is latest as of this writing
ARG CERTBOT_VERSION
ENV CERTBOT_VERSION=1.4.0-r0

### Now we're going to pull in certbot for certs
### Most of this is shamelessly stolen from the certbot image on hub
### https://hub.docker.com/r/certbot/certbot/dockerfile
RUN mkdir -p /opt/certbot/src

WORKDIR /opt/certbot

RUN wget -O certbot-${CERTBOT_VERSION}.tar.gz https://github.com/certbot/certbot/archive/v${CERTBOT_VERSION}.tar.gz \
 && tar xf certbot-${CERTBOT_VERSION}.tar.gz \
 && cp certbot-${CERTBOT_VERSION}/CHANGELOG.md certbot-${CERTBOT_VERSION}/README.rst src/ \
 && cp certbot-${CERTBOT_VERSION}/letsencrypt-auto-source/pieces/dependency-requirements.txt . \
 && cp certbot-${CERTBOT_VERSION}/letsencrypt-auto-source/pieces/pipstrap.py . \
 && cp -r certbot-${CERTBOT_VERSION}/tools tools \
 && cp -r certbot-${CERTBOT_VERSION}/acme src/acme \
 && cp -r certbot-${CERTBOT_VERSION}/certbot src/certbot \
 && rm -rf certbot-${CERTBOT_VERSION}.tar.gz certbot-${CERTBOT_VERSION}

# Generate constraints file to pin dependency versions
RUN cat dependency-requirements.txt | tools/strip_hashes.py > unhashed_requirements.txt \
 && cat tools/dev_constraints.txt unhashed_requirements.txt | tools/merge_requirements.py > docker_constraints.txt

# Install certbot runtime dependencies
RUN apk add --no-cache --virtual .certbot-deps \
        libffi \
        libssl1.1 \
        openssl \
        ca-certificates \
        binutils

# Install certbot from sources
RUN apk add --no-cache --virtual .build-deps \
        gcc \
        linux-headers \
        openssl-dev \
        musl-dev \
        libffi-dev \
    && python pipstrap.py \
    && pip install -r dependency-requirements.txt \
    && pip install --no-cache-dir --no-deps \
        --editable src/acme \
        --editable src/certbot \
&& apk del .build-deps