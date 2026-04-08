FROM python:3.11-alpine AS builder
WORKDIR /tmp/build
COPY package.json .npmrc pnpm-lock.yaml ./
RUN set -x \
  && apk update \
  && apk add nodejs npm git \
  && npm i -g pnpm@8.3.1 pm2 ts-node \
  && pnpm install --frozen-lockfile

COPY . .
RUN pnpm build:front \
  && pnpm build:back

FROM python:3.11-alpine

ARG QL_MAINTAINER="whyour"
LABEL maintainer="${QL_MAINTAINER}"
ARG QL_URL=https://github.com/${QL_MAINTAINER}/qinglong.git
ARG QL_BRANCH=develop
ARG PYTHON_SHORT_VERSION=3.11

ENV QL_DIR=/ql \
  QL_BRANCH=${QL_BRANCH} \
  LANG=C.UTF-8 \
  SHELL=/bin/bash \
  PS1="\u@\h:\w \$ "

VOLUME /ql/data
  
EXPOSE 5700

COPY --from=builder /usr/local/lib/node_modules/. /usr/local/lib/node_modules/
COPY --from=builder /usr/local/bin/. /usr/local/bin/

RUN set -x \
  && apk update -f \
  && apk upgrade \
  && apk --no-cache add -f bash \
  coreutils \
  git \
  curl \
  wget \
  tzdata \
  perl \
  openssl \
  nodejs \
  jq \
  openssh \
  procps \
  netcat-openbsd \
  unzip \
  npm \
  && rm -rf /var/cache/apk/* \
  && apk update \
  && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
  && echo "Asia/Shanghai" > /etc/timezone \
  && git config --global user.email "qinglong@users.noreply.github.com" \
  && git config --global user.name "qinglong" \
  && git config --global http.postBuffer 524288000 \
  && rm -rf /root/.cache \
  && ulimit -c 0

ARG SOURCE_COMMIT
WORKDIR ${QL_DIR}
COPY . ${QL_DIR}
RUN cd ${QL_DIR} \
  && cp -f .env.example .env \
  && chmod 777 ${QL_DIR}/shell/*.sh \
  && chmod 777 ${QL_DIR}/docker/*.sh

ENV PNPM_HOME=${QL_DIR}/data/dep_cache/node \
  PYTHON_HOME=${QL_DIR}/data/dep_cache/python3 \
  PYTHONUSERBASE=${QL_DIR}/data/dep_cache/python3 \
  HOME=/root

ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:${PNPM_HOME}:${PYTHON_HOME}/bin:${HOME}/bin \
  NODE_PATH=/usr/local/bin:/usr/local/lib/node_modules:${PNPM_HOME}/global/5/node_modules \
  PIP_CACHE_DIR=${PYTHON_HOME}/pip \
  PYTHONPATH=${PYTHON_HOME}:${PYTHON_HOME}/lib/python${PYTHON_SHORT_VERSION}:${PYTHON_HOME}/lib/python${PYTHON_SHORT_VERSION}/site-packages

RUN pip3 install --prefix ${PYTHON_HOME} requests

COPY --from=builder /tmp/build/node_modules/. /ql/node_modules/
COPY --from=builder /tmp/build/static/build/. /ql/static/build/
COPY --from=builder /tmp/build/static/dist/. /ql/static/dist/

HEALTHCHECK --interval=5s --timeout=2s --retries=20 \
  CMD curl -sf --noproxy '*' http://127.0.0.1:${QlPort:-5700}/api/health || exit 1

ENTRYPOINT ["./docker/docker-entrypoint.sh"]
