# Builder
FROM erlang:26.2-alpine AS builder
RUN apk add --update git build-base coreutils bsd-compat-headers openssl-dev snappy-dev curl alpine-sdk bash \
    && git clone -b 2.0.1 https://github.com/bmappi/mqtt vernemq \
    && cd vernemq \
    && make VERBOSE=1 -j 16 rel || (echo "First attempt failed, trying again..." && make VERBOSE=1 -j 16 rel) && echo "Should be okay..."

# Executor
FROM alpine:3.20
COPY --from=builder /vernemq/_build/default/rel/ /

RUN apk --no-cache --update --available upgrade && \
    apk add --no-cache ncurses-libs openssl libstdc++ jq curl bash snappy-dev nano && \
    addgroup --gid 10000 vernemq && \
    adduser --uid 10000 -H -D -G vernemq -h /vernemq vernemq && \
    install -d -o vernemq -g vernemq /vernemq

# Defaults
ENV DOCKER_VERNEMQ_KUBERNETES_LABEL_SELECTOR="app=vernemq" \
    DOCKER_VERNEMQ_LOG__CONSOLE=console \
    PATH="/vernemq/bin:$PATH" \
    VERNEMQ_VERSION="2.0.1"

WORKDIR /vernemq

COPY --chown=10000:10000 docker/bin/vernemq.sh /usr/sbin/start_vernemq
COPY --chown=10000:10000 docker/bin/join_cluster.sh /usr/sbin/join_cluster
COPY --chown=10000:10000 docker/bin/vm.args /vernemq/etc/vm.args


RUN chown -R 10000:10000 /vernemq && \
    ln -s /vernemq/etc /etc/vernemq && \
    ln -s /vernemq/data /var/lib/vernemq && \
    ln -s /vernemq/log /var/log/vernemq

# Ports
# 1883  MQTT
# 8883  MQTT/SSL
# 8080  MQTT WebSockets
# 44053 VerneMQ Message Distribution
# 4369  EPMD - Erlang Port Mapper Daemon
# 8888  Health, API, Prometheus Metrics
# 9100 9101 9102 9103 9104 9105 9106 9107 9108 9109  Specific Distributed Erlang Port Range

EXPOSE 1883 8883 8080 44053 4369 8888 \
       9100 9101 9102 9103 9104 9105 9106 9107 9108 9109


VOLUME ["/vernemq/log", "/vernemq/data", "/vernemq/etc"]

HEALTHCHECK CMD vernemq ping | grep -q pong

USER vernemq

RUN chmod +x /usr/sbin/start_vernemq

CMD ["start_vernemq"]