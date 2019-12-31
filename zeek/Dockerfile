from alpine:3 AS build

ENV ZEEK_VERSION="3.0.1"

RUN apk add --no-cache \
    bison \
    cmake \
    g++ \
    gcc \
    flex \
    fts-dev \
    krb5-dev \
    libmaxminddb-dev \
    linux-headers \
    libpcap-dev \
    make \
    openssl-dev \
    python3-dev \
    swig \
    zlib-dev

RUN curl -L https://www.zeek.org/downloads/zeek-"${ZEEK_VERSION}".tar.gz \
      | tar -xzC / \
      && find /zeek-"${ZEEK_VERSION}"/ -type f -exec sed -i 's/bash/sh/' {} +

WORKDIR /zeek-"${ZEEK_VERSION}"

RUN ./configure \
    --build-type=MinSizeRel \
    --prefix=/usr \
    --conf-files-dir=/etc/zeek \
    --enable-mobile-ipv6 \
    --disable-auxtools \
    --disable-broker-tests

RUN make

# Start final build
FROM alpine:3

ENV ZEEK_VERSION="3.0.1"
ENV USER=zeek
ENV UID=1005
ENV GID=1005

COPY --from=build /zeek-"${ZEEK_VERSION}" /zeek-"${ZEEK_VERSION}"

RUN addgroup --gid "${GID}" "${USER}" \
    && adduser \
      --disabled-password \
      --gecos "${USER} Service Account" \
      --home "$(pwd)" \
      --ingroup "${USER}" \
      --no-create-home \
      --uid "${UID}" \
      "${USER}" \
    && apk add --no-cache \
      bind \
      fts \
      libmaxminddb \
      libpcap \
      openssl \
      zlib

# Not using WORKDIR here, because it gets removed in same step
RUN apk add --no-cache --virtual .make \
      cmake \
      g++ \
      gcc \
      flex \
      fts-dev \
      krb5-dev \
      libmaxminddb-dev \
      linux-headers \
      libpcap-dev \
      make \
      openssl-dev \
      python3-dev \
      swig \
      zlib-dev \
    && cd /zeek-"${ZEEK_VERSION}" \
    && make install \
    && apk del --no-network .make \
    && cd / \
    && rm -rf /zeek-"${ZEEK_VERSION}" \
    && chown -R zeek.zeek /usr/share/zeek \
    && chown -R zeek.zeek /etc/zeek \
    && chown zeek.zeek /usr/bin/zeek

USER zeek

CMD ["/usr/bin/zeek","-h"]
