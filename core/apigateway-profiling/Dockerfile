# apigateway
#
# VERSION               1.9.7.3
#
# From https://hub.docker.com/_/alpine/
#
FROM ubuntu:xenial

# install dependencies
RUN apt-get update && apt-get -y install --no-install-recommends \
        tar \
        libtool \
        libjemalloc1 \
        perl \
        ca-certificates \
        wget \
        curl \
        python python-pip \
        musl-dev musl-tools \
        libssl-dev \
        libpcre2-dev \
        zlib1g-dev \
        lua-zlib lua-zlib-dev \
        libtest-longstring-perl \
        liblist-moreutils-perl \
        libhttp-message-perl \
        libgeoip-dev \
        libjemalloc-dev \
        jq \
        elfutils systemtap systemtap-sdt-dev \
        gcc g++ make \
        automake \
        autoconf \
        git \
        libjansson-dev \
        kbuild \
    && rm -rf /var/lib/apt/lists/* \
    && update-ca-certificates

# openresty build
ENV OPENRESTY_VERSION=1.13.6.1 \
    NAXSI_VERSION=0.53-2 \
    PCRE_VERSION=8.37 \
    TEST_NGINX_VERSION=0.24 \
    OPM_VERSION=0.0.3 \
    LUA_RESTY_HTTP_VERSION=0.10 \
    LUA_RESTY_IPUTILS_VERSION=0.2.1 \
    LUA_RESTY_STRING_VERSION=0.09 \
    LUA_RESTY_LRUCACHE_VERSION=0.06 \
    LUA_RESTY_CJOSE_VERSION=0.4 \
    NETURL_LUA_VERSION=0.9-1 \
    CJOSE_VERSION=0.5.1 \
    LD_LIBRARY_PATH=/usr/local/lib \
    _prefix=/usr/local \
    _exec_prefix=/usr/local \
    _localstatedir=/var \
    _sysconfdir=/etc \
    _sbindir=/usr/local/sbin

COPY build-scripts /build-scripts
COPY etc-api-gateway /etc/api-gateway

RUN echo "Beginning OpenRESTy build..." \
     && bash /build-scripts/build-openresty.sh

RUN echo "        - adding Nginx Test support" \
    && curl -sSL "https://github.com/openresty/test-nginx/archive/v${TEST_NGINX_VERSION}.tar.gz" \
        -o "${_prefix}/test-nginx-${TEST_NGINX_VERSION}.tar.gz" \
    && tar -C "${_prefix}" -xf "${_prefix}/test-nginx-${TEST_NGINX_VERSION}.tar.gz" \
    && rm "${_prefix}/test-nginx-${TEST_NGINX_VERSION}.tar.gz" \
    && mkdir -p /usr/local/share/perl5/site_perl \
    && cp -vr "${_prefix}/test-nginx-${TEST_NGINX_VERSION}/inc"/* /usr/local/share/perl5/site_perl/ \
    && ln -s "${_sbindir}/api-gateway-debug" "${_sbindir}/nginx"

RUN echo " ... installing opm..." \
    && mkdir -p /tmp/api-gateway \
    && curl -sSL "https://github.com/openresty/opm/archive/v${OPM_VERSION}.tar.gz" \
        -o "/tmp/api-gateway/opm-${OPM_VERSION}.tar.gz" \
    && tar -xf "/tmp/api-gateway/opm-${OPM_VERSION}.tar.gz" -C /tmp/api-gateway \
    && cp "/tmp/api-gateway/opm-${OPM_VERSION}/bin/opm" "${_prefix}/api-gateway/bin" \
    && mkdir -p "${_prefix}/api-gateway/site/manifest" "${prefix}/api-gateway/site/pod" \
    && if [ ! -x "${_prefix}/api-gateway/lualib" ]; then \
            ln -s "${_prefix}/api-gateway/site/lualib" "${_prefix}/api-gateway/lualib" \
       ; fi \
    && ln -s "${_prefix}/api-gateway/bin/opm" "/usr/bin/opm" \
    && ln -s "${_prefix}/api-gateway/bin/resty" /usr/bin/resty \
    && rm -rf /tmp/api-gateway

RUN echo " ... installing opm packages ... " \
    && opm get "pintsized/lua-resty-http=${LUA_RESTY_HTTP_VERSION}" \
               "hamishforbes/lua-resty-iputils=${LUA_RESTY_IPUTILS_VERSION}" \
               "openresty/lua-resty-string=${LUA_RESTY_STRING_VERSION}" \
               "openresty/lua-resty-lrucache=${LUA_RESTY_LRUCACHE_VERSION}" \
               "taylorking/lua-resty-cjose=${LUA_RESTY_CJOSE_VERSION}" \
               taylorking/lua-resty-rate-limit

RUN echo " ... installing neturl.lua ... " \
    && mkdir -p /tmp/api-gateway \
    && curl -sSL "https://github.com/golgote/neturl/archive/${NETURL_LUA_VERSION}.tar.gz" \
        -o "/tmp/api-gateway/neturl.lua-${NETURL_LUA_VERSION}.tar.gz" \
    && tar -xf "/tmp/api-gateway/neturl.lua-${NETURL_LUA_VERSION}.tar.gz" \
        -C "/tmp/api-gateway/" \
    && export LUA_LIB_DIR="${_prefix}/api-gateway/lualib" \
    && cp "/tmp/api-gateway/neturl-${NETURL_LUA_VERSION}/lib/net/url.lua" "${LUA_LIB_DIR}" \
    && rm -rf /tmp/api-gateway

RUN echo " ... installing cjose ... " \
    && mkdir -p /tmp/api-gateway \
    && curl -L -k "https://github.com/cisco/cjose/archive/${CJOSE_VERSION}.tar.gz" \
            -o "/tmp/api-gateway/cjose-${CJOSE_VERSION}.tar.gz" \
    && tar -xf "/tmp/api-gateway/cjose-${CJOSE_VERSION}.tar.gz" -C /tmp/api-gateway/ \
    && (cd "/tmp/api-gateway/cjose-${CJOSE_VERSION}" && sh configure && make install ) \
    && rm -rf /tmp/api-gateway

COPY init.sh /etc/init-container.sh
RUN pip install dumb-init==1.2.1

# add the default configuration for the Gateway

RUN mkdir -p /profiling \
 && (cd /tmp && git clone https://github.com/openresty/stapxx.git) \
 && (cd /tmp/stapxx && git clone https://github.com/brendangregg/FlameGraph.git)
WORKDIR /tmp/stapp

RUN adduser --system --disabled-password --gecos "" nginx-api-gateway


EXPOSE 80 8080 8423 9000

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["/etc/init-container.sh"]

# VIM: let b:syntastic_dockerfile_hadolint_args = "--ignore DL3008 --ignore SC2154 "
