ARG ERLANG_VERSION=23.3.2
ARG VERNEMQ_VERSION=###TO BE REPLACED###
ARG VERNEMQ_DIRTY=###TO BE REPLACED###
ARG VERNEMQ_RELEASED=1.12.3

FROM erlang:${ERLANG_VERSION} as builder
ARG VERNEMQ_VERSION
ARG VERNEMQ_DIRTY

RUN apt-get update && \
    apt-get install -y \
    libsnappy-dev

RUN git clone https://github.com/ratonlaveur38/vernemq.git --branch ${VERNEMQ_VERSION} /tmp/vernemq

WORKDIR /tmp/vernemq
RUN git checkout ${VERNEMQ_VERSION}${VERNEMQ_DIRTY}

RUN make -C /tmp/vernemq rel

FROM erlang:${ERLANG_VERSION}
ARG VERNEMQ_VERSION
ARG VERNEMQ_DIRTY
ARG VERNEMQ_RELEASED

RUN apt-get update && \
    apt-get -y install bash procps openssl iproute2 curl jq libsnappy-dev net-tools && \
    rm -rf /var/lib/apt/lists/* && \
    addgroup --gid 10000 vernemq && \
    adduser --uid 10000 --system --ingroup vernemq --home /vernemq --disabled-password vernemq

COPY --from=builder --chown=10000:10000 /tmp/vernemq/_build/default/rel/vernemq /vernemq

RUN curl --create-dirs -o /usr/sbin/start_vernemq --silent -L https://raw.githubusercontent.com/vernemq/docker-vernemq/${VERNEMQ_RELEASED}/bin/vernemq.sh && \
    chown vernemq:vernemq /usr/sbin/start_vernemq && \
    chmod +x /usr/sbin/start_vernemq

RUN curl --create-dirs -o /vernemq/etc/vm.args --silent -L https://raw.githubusercontent.com/vernemq/docker-vernemq/${VERNEMQ_RELEASED}/files/vm.args

RUN ln -s /vernemq/etc /etc/vernemq && \
    ln -s /vernemq/data /var/lib/vernemq && \
    ln -s /vernemq/log /var/log/vernemq

ENV PATH="/vernemq/bin:$PATH" \
    DOCKER_VERNEMQ_LOG__CONSOLE=console

HEALTHCHECK CMD vernemq ping | grep -q pong

WORKDIR /vernemq
USER vernemq

CMD ["/usr/sbin/start_vernemq"]
