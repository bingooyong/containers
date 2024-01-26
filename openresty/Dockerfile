
FROM docker.io/bitnami/minideb:bullseye as builder

ARG TARGETARCH
ARG RESTY_VERSION
ARG RESTY_OPENSSL_VERSION
ARG RESTY_ZLIB_VERSION

LABEL maintainer="bingooyong@github.io"

# Docker Build Arguments

ENV HOME="/home/openresty" \
    OS_ARCH="${TARGETARCH:-amd64}" \
    OS_FLAVOUR="debian-11" \
    OS_NAME="linux" \
    RESTY_VERSION="1.21.4.3" \
    RESTY_OPENSSL_VERSION="1.1.1q" \
    RESTY_ZLIB_VERSION="1.3.1"

COPY prebuildfs /
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
# Install required system packages and dependencies
RUN install_packages ca-certificates curl libbrotli1 libbsd0 libcrypt1 libdeflate0 libexpat1 libfontconfig1 libfreetype6 libgcc-s1 libgcrypt20 libgd3 libgeoip1 libgpg-error0 libicu67 libjbig0 libjpeg62-turbo liblzma5 libmd0 libpcre3 libpng16-16 libssl1.1 libstdc++6 libtiff5 libuuid1 libwebp6 libx11-6 libxau6 libxcb1 libxdmcp6 libxml2 libxpm4 libxslt1.1 libzstd1 perl procps zlib1g wget build-essential unzip net-tools
RUN mkdir -p /app
WORKDIR /app

RUN wget --no-check-certificate https://www.openssl.org/source/openssl-${RESTY_OPENSSL_VERSION}.tar.gz \
&& wget https://openresty.org/download/openresty-${RESTY_VERSION}.tar.gz \
&& wget https://www.zlib.net/zlib-${RESTY_ZLIB_VERSION}.tar.gz \
&& wget --no-check-certificate https://cytranet.dl.sourceforge.net/project/pcre/pcre/8.45/pcre-8.45.tar.bz2 \
&& tar xf openresty-${RESTY_VERSION}.tar.gz && tar xf openssl-${RESTY_OPENSSL_VERSION}.tar.gz && tar xf pcre-8.45.tar.bz2 && tar xf zlib-${RESTY_ZLIB_VERSION}.tar.gz

RUN cd openresty-${RESTY_VERSION} && ./configure -j$(grep -c ^processor /proc/cpuinfo) \
    --prefix=/opt/bitnami/openresty \
    --with-luajit=$LUAROOT \
    --with-pcre-jit \
    --with-pcre=../pcre-8.45 \
    --with-zlib=../zlib-${RESTY_ZLIB_VERSION} \
    --with-openssl=../openssl-${RESTY_OPENSSL_VERSION} \
    --without-http_redis2_module \
    --with-http_realip_module \
    --with-http_ssl_module \
    --with-http_stub_status_module \
    --with-http_v2_module \
    && gmake -j$(grep -c ^processor /proc/cpuinfo) && gmake install

# https://github.com/brimworks/lua-zlib/archive/master.zip
# https://github.com/bingoohuang/lua-zlibx/archive/master.zip
# add lua-zlib
RUN cd zlib-${RESTY_ZLIB_VERSION} && make && make install \ 
    && install_packages cmake \
    && cd /app \
    && curl -SsLf https://github.com/bingoohuang/lua-zlibx/archive/master.zip -O \
    && unzip master.zip \
    && cd lua-zlibx-master \
    && cmake -DLUA_INCLUDE_DIR=/opt/bitnami/openresty/luajit/include/luajit-2.1 -DLUA_LIBRARIES=/opt/bitnami/openresty/luajit/lib -DUSE_LUAJIT=ON -DUSE_LUA=OFF \
    && make \
    && cp zlib.so /opt/bitnami/openresty/lualib/zlib.so

RUN apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives
RUN chmod g+rwX /opt/bitnami

COPY rootfs /
RUN /opt/bitnami/scripts/openresty/postunpack.sh

ENV APP_VERSION=${RESTY_VERSION} \
    BITNAMI_APP_NAME="openresty" \
    PATH="/opt/bitnami/openresty/bin:/opt/bitnami/openresty/luajit/bin:$PATH"

WORKDIR /app
USER root
ENTRYPOINT [ "/opt/bitnami/scripts/openresty/entrypoint.sh" ]
CMD [ "/opt/bitnami/scripts/openresty/run.sh" ]


#########################################
FROM docker.io/bitnami/minideb:bullseye
USER root

ENV HOME="/home/openresty" \
    OS_ARCH="${TARGETARCH:-amd64}" \
    OS_FLAVOUR="debian-11" \
    OS_NAME="linux"

COPY prebuildfs /
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# 将上一个阶段publish文件夹下的所有文件复制进来
COPY --from=builder /opt/bitnami/openresty /opt/bitnami/openresty

RUN install_packages ca-certificates curl libbrotli1 libbsd0 libcrypt1 libdeflate0 libexpat1 libfontconfig1 libfreetype6 libgcc-s1 libgcrypt20 libgd3 libgeoip1 libgpg-error0 libicu67 libjbig0 libjpeg62-turbo liblzma5 libmd0 libpcre3 libpng16-16 libssl1.1 libstdc++6 libtiff5 libuuid1 libwebp6 libx11-6 libxau6 libxcb1 libxdmcp6 libxml2 libxpm4 libxslt1.1 libzstd1 perl procps zlib1g

RUN apt-get update && apt-get upgrade -y && \
    apt-get clean && rm -rf /var/lib/apt/lists /var/cache/apt/archives
RUN chmod g+rwX /opt/bitnami

COPY rootfs /
RUN /opt/bitnami/scripts/openresty/postunpack.sh

ENV APP_VERSION=${RESTY_VERSION} \
    BITNAMI_APP_NAME="openresty" \
    PATH="/opt/bitnami/openresty/bin:/opt/bitnami/openresty/luajit/bin:$PATH"

WORKDIR /app
ENTRYPOINT [ "/opt/bitnami/scripts/openresty/entrypoint.sh" ]
CMD [ "/opt/bitnami/scripts/openresty/run.sh" ]