# --- Stage 1/2 --- #

FROM alpine:3.20.3 AS nginx-build

RUN mkdir /app
WORKDIR /app

# NGINX CORE
ARG NGINX_VERSION=1.26.2
ARG PCRE2_VERSION=10.44
ARG ZLIB_VERSION=1.3.1
ARG BORINGSSL_COMMIT_SHA=c691779ed0e98b36eff7ad945a738c402f127122

# 3RD PARTY MODULES
ARG HEADERS_MORE_NGINX_VERSION=0.37
ARG NGINX_DEVEL_NGINX_VERSION=0.3.3
ARG SET_MISC_NGINX_VERSION=0.33
ARG VTS_NGINX_VERSION=0.2.2
ARG STS_NGINX_VERSION=0.1.1
ARG STS_STREAM_NGINX_VERSION=0.1.1
ARG BROTLI_NGINX_COMMIT_SHA=a71f9312c2deb28875acc7bacfdd5695a111aa53
ARG GEOIP2_NGINX_VERSION=3.4

# NGINX PATCH FILENAME
ARG NGINX_PATCH_FILENAME="nginx-1.26.2"

# Add nginx user
RUN addgroup -S nonroot && adduser -S -G nonroot -s /bin/false nonroot

# Download needed packages from repository
RUN apk add --no-cache git make cmake ninja gcc openssl-dev linux-headers g++ perl-dev patch bzip2-dev readline-dev libedit-dev zlib-dev libunwind-dev libmaxminddb-dev && \
    rm -rf /var/cache/apk/*

# Downloading NGINX
RUN echo "Downloading NGINX version ${NGINX_VERSION}" && \
    wget "https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" -O /app/nginx.tar.gz && tar xzvf /app/nginx.tar.gz

# Clone and Build zlib
RUN echo "Cloning zlib... Version: ${ZLIB_VERSION}" && \
    git clone https://github.com/madler/zlib.git /app/zlib && \
    cd /app/zlib && \
    git checkout tags/v${ZLIB_VERSION} -b build && \
    echo "Building zlib..." && \
    mkdir build && \
    cd build && \
    cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_FLAGS="-O3 -march=native -flto" .. && \
    make -j$(nproc) && make install

# Clone and Build boringssl 
RUN echo "Cloning boringssl... SHA: ${BORINGSSL_COMMIT_SHA}" && \
    git clone https://github.com/google/boringssl.git -b master /app/boringssl && \
    cd /app/boringssl && \
    git checkout ${BORINGSSL_COMMIT_SHA} && \
    echo "Building boringssl..." && \
    mkdir build && \
    cd build && \
    cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_FLAGS="-O3 -march=native" -DCMAKE_CXX_FLAGS="-O3 -march=native -flto" .. && \
    make -j$(nproc) && make install

# Clone and Build pcre2 
RUN echo "Cloning PCRE2... Version: ${PCRE2_VERSION}" && \
    git clone https://github.com/PCRE2Project/pcre2.git /app/pcre2 && \
    cd /app/pcre2 && \
    git checkout tags/pcre2-${PCRE2_VERSION} && \
    echo "Building PCRE2..." && \
    mkdir build && \
    cd build && \
    cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON -DPCRE2_BUILD_PCRE2GREP=ON -DPCRE2_SUPPORT_JIT=ON -DCMAKE_C_FLAGS="-O3 -march=native" .. && \
    make -j$(nproc) && make install

# ---NGINX MODULES--- #

# Clone the headers-more-nginx-module repository and checkout the specific tag
RUN echo "Cloning headers-more-nginx-module version ${HEADERS_MORE_NGINX_VERSION}" && \
    git clone https://github.com/openresty/headers-more-nginx-module.git /app/headers-more-nginx-module && \
    cd /app/headers-more-nginx-module && \
    git checkout tags/v${HEADERS_MORE_NGINX_VERSION} -b build

# Clone the set-misc-nginx-module repository and checkout the specific tag
RUN echo "Cloning set-misc-nginx-module version ${SET_MISC_NGINX_VERSION}" && \
    git clone https://github.com/openresty/set-misc-nginx-module.git /app/set-misc-nginx-module && \
    cd /app/set-misc-nginx-module && \
    git checkout tags/v${SET_MISC_NGINX_VERSION} -b build

# Clone the ngx_devel_kit repository and checkout the specific tag
RUN echo "Cloning ngx_devel_kit version ${NGINX_DEVEL_NGINX_VERSION}" && \
    git clone https://github.com/vision5/ngx_devel_kit.git /app/ngx_devel_kit && \
    cd /app/ngx_devel_kit && \
    git checkout tags/v${NGINX_DEVEL_NGINX_VERSION} -b build

# Clone the nginx-module-vts repository and checkout the specific tag
RUN echo "Cloning nginx-module-vts version ${VTS_NGINX_VERSION}" && \
    git clone https://github.com/vozlt/nginx-module-vts.git /app/nginx-module-vts && \
    cd /app/nginx-module-vts && \
    git checkout tags/v${VTS_NGINX_VERSION} -b build

# Clone the nginx-module-sts repository and checkout the specific tag
RUN echo "Cloning nginx-module-sts version ${STS_NGINX_VERSION}" && \
git clone https://github.com/vozlt/nginx-module-sts.git /app/nginx-module-sts && \
cd /app/nginx-module-sts && \
git checkout tags/v${STS_NGINX_VERSION} -b build

# Clone the nginx-module-stream-sts repository and checkout the specific tag
RUN echo "Cloning nginx-module-stream-sts version ${STS_STREAM_NGINX_VERSION}" && \
    git clone https://github.com/vozlt/nginx-module-stream-sts.git /app/nginx-module-stream-sts && \
    cd /app/nginx-module-stream-sts && \
    git checkout tags/v${STS_STREAM_NGINX_VERSION} -b build

# Clone the ngx_brotli repository and checkout the specific commit
RUN echo "Cloning ngx_brotli with commit ${BROTLI_NGINX_COMMIT_SHA}" && \
    git clone https://github.com/google/ngx_brotli.git /app/ngx_brotli && \
    cd /app/ngx_brotli && \
    git checkout ${BROTLI_NGINX_COMMIT_SHA} -b build && \
    git submodule update --init --recursive && \
    mkdir -p /app/ngx_brotli/deps/brotli/out && \
    cd /app/ngx_brotli/deps/brotli && \
    cmake -DCMAKE_BUILD_TYPE=Release . && \
    make -j$(nproc) && \
    make install

# Clone the ngx_http_geoip2_module repository and checkout the specific tag
RUN echo "Cloning ngx_http_geoip2_module version ${GEOIP2_NGINX_VERSION}" && \
    git clone https://github.com/leev/ngx_http_geoip2_module.git /app/ngx_http_geoip2_module && \
    cd /app/ngx_http_geoip2_module && \
    git checkout tags/${GEOIP2_NGINX_VERSION} -b build


# Create temp folders
RUN mkdir -p /var/cache/nginx/client_temp && \
  mkdir -p /var/cache/nginx/proxy_temp

# Apply patch
COPY ${NGINX_PATCH_FILENAME}.patch /app/${NGINX_PATCH_FILENAME}.patch
RUN cd /app/nginx-${NGINX_VERSION} && patch -p01 < /app/${NGINX_PATCH_FILENAME}.patch

# Configure NGINX
RUN cd /app/nginx-${NGINX_VERSION} && \
  ./configure \
  --prefix=/etc/nginx \
  --sbin-path=/usr/sbin/nginx \
  --modules-path=/usr/lib/nginx/modules \
  --conf-path=/etc/nginx/nginx.conf \
  --error-log-path=/var/log/nginx/error.log \
  --pid-path=/var/run/nginx.pid \
  --lock-path=/var/run/nginx.lock \
  --user=nonroot \
  --group=nonroot \
  --build=nginx-shadow \
  --with-select_module \
  --with-poll_module \
  --with-threads \
  --with-file-aio \
  --with-http_ssl_module \
  --with-http_v2_module \
  --with-http_v3_module \
  --with-http_realip_module \
  --with-http_auth_request_module \
  --with-http_stub_status_module \
  --with-stream \
  --with-stream_ssl_module \
  --with-stream_realip_module \
  --without-http_ssi_module \
  --without-http_userid_module \
  --without-http_mirror_module \
  --without-http_autoindex_module \
  --without-http_split_clients_module \
  --without-http_fastcgi_module \
  --without-http_uwsgi_module \
  --without-http_scgi_module \
  --without-http_grpc_module \
  --without-http_memcached_module \
  --http-log-path=/var/log/nginx/access.log \
  --http-client-body-temp-path=/var/cache/nginx/client_temp \
  --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
  --without-mail_pop3_module \
  --without-mail_imap_module \
  --without-mail_smtp_module \
  --add-module=/app/ngx_devel_kit \
  --add-module=/app/set-misc-nginx-module \
  --add-module=/app/headers-more-nginx-module  \
  --add-module=/app/nginx-module-vts \
  --add-module=/app/nginx-module-sts \
  --add-module=/app/nginx-module-stream-sts \
  --add-module=/app/ngx_brotli \
  --add-module=/app/ngx_http_geoip2_module \
  --with-cc=c++ \
  --with-cc-opt="-I/app/boringssl/include -I/app/pcre2/include -O2 -x c" \
  --with-ld-opt="-L/app/ngx_brotli/deps/brotli/out -L/app/boringssl/build/ssl -L/app/boringssl/build/crypto -L/app/pcre2/lib -Wl,-rpath,/app/boringssl/build/ssl" \
  --with-pcre-jit \
  --with-zlib=/app/zlib \
  --with-debug && \
  echo $(eval $CONFIGURE) && \
  make -j$(nproc) && make install

RUN nginx -V

RUN chown -R nonroot:nonroot /etc/nginx
RUN chown -R nonroot:nonroot /etc/ssl/
RUN chown -R nonroot:nonroot /var/log/nginx/
RUN chown -R nonroot:nonroot /var/cache/nginx/
RUN chown -R nonroot:nonroot /var/run/

RUN mkdir -p /etc/nginx/logs/


# Forward logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
  && ln -sf /dev/stderr /etc/nginx/logs/error.log 

# Copying nginx and its dependencies to the final image
RUN mkdir -p /app/deps && \
  ldd $(which nginx) | grep "=> /" | awk '{print $3}' | xargs -I '{}' cp -v '{}' /app/deps/ && \
  cp $(which nginx) /app/nginx


# --- Stage 2/2 --- #


# # Grab the distroless static container.
FROM gcr.io/distroless/base:nonroot AS nginx

COPY --from=nginx-build /var/log/nginx/ /var/log/nginx/
COPY --from=nginx-build /var/cache/nginx/ /var/cache/nginx/
COPY --from=nginx-build /etc/nginx/ /etc/nginx/
COPY --from=nginx-build /etc/nginx/logs/ /etc/nginx/logs/
#COPY --from=nginx-build /etc/ssl/ /etc/ssl/
#COPY --from=nginx-build /var/run/ /var/run/

COPY ./nginx.conf /etc/nginx/

# Skopiowanie nginx i zależności do finalnego obrazu
COPY --from=nginx-build /app/nginx /usr/sbin/nginx
COPY --from=nginx-build /app/deps /lib

# NGINX needs to be started as root. (for master proces)
# Only root processes can listen to ports below 1024
USER root

EXPOSE 80/tcp 443/tcp

STOPSIGNAL SIGTERM

CMD ["nginx", "-g", "daemon off;"]
