# --- Stage 1/2 --- #

FROM alpine:3.20.3 AS nginx-build

RUN mkdir /app
WORKDIR /app

# NGINX CORE
ARG NGINX_VERSION=1.26.2
ARG OPENSSL_VERSION=3.0.15
ARG PCRE2_VERSION=10.44
ARG ZLIB_VERSION=1.3.1

# 3RD PARTY MODULES
ARG HEADERS_MORE_NGINX_VERSION=0.37
ARG NGINX_DEVEL_NGINX_VERSION=0.3.3
ARG SET_MISC_NGINX_VERSION=0.33

# NGINX PATCH FILENAME
#ARG NGINX_PATCH_FILENAME="nginx-1.22.0"

# Set TimeZone
#ENV TZ=Europe/Warsaw
#RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Add nginx user
RUN addgroup -S nonroot && adduser -S -G nonroot -s /bin/false nonroot

# Download needed packages from repository
RUN apk update && \
    apk add --no-cache --virtual .build-deps \
      linux-headers \
      build-base \
      curl \
      git \
      cmake \
      wget \
      gcc \
      make \
      unzip \
      ca-certificates \
      autoconf \
      automake \
      libtool \
      pkgconf \
      zlib-dev \
      openssl-dev \
      pcre-dev \
      libxml2-dev \
      yajl-dev \
      lua5.2-dev \
      geoip-dev && \
    apk add --no-cache \
      libcurl \
      pax-utils && \
    rm -rf /var/cache/apk/*


# Download dependencies
RUN wget "https://github.com/PCRE2Project/pcre2/releases/download/pcre2-${PCRE2_VERSION}/pcre2-${PCRE2_VERSION}.tar.gz" -O /app/pcre2.tar.gz && tar xzvf /app/pcre2.tar.gz && \
   wget "https://github.com/openresty/headers-more-nginx-module/archive/refs/tags/v${HEADERS_MORE_NGINX_VERSION}.tar.gz" -O /app/headers_more.tar.gz && tar xzvf /app/headers_more.tar.gz && \
  wget "https://github.com/openssl/openssl/releases/download/openssl-${OPENSSL_VERSION}/openssl-${OPENSSL_VERSION}.tar.gz" -O /app/openssl.tar.gz && tar xzvf /app/openssl.tar.gz && \
  wget "https://github.com/madler/zlib/archive/refs/tags/v${ZLIB_VERSION}.tar.gz" -O /app/zlib.tar.gz && tar xzvf /app/zlib.tar.gz && \
  wget "https://github.com/openresty/set-misc-nginx-module/archive/refs/tags/v${SET_MISC_NGINX_VERSION}.tar.gz" -O /app/set_misc.tar.gz && tar xzvf /app/set_misc.tar.gz && \
  wget "https://github.com/vision5/ngx_devel_kit/archive/refs/tags/v${NGINX_DEVEL_NGINX_VERSION}.tar.gz" -O /app/nginx_devel.tar.gz && tar xzvf /app/nginx_devel.tar.gz && \
  wget "https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" -O /app/nginx.tar.gz && tar xzvf /app/nginx.tar.gz

  

# Create temp folders
RUN mkdir -p /var/cache/nginx/client_temp && \
  mkdir -p /var/cache/nginx/proxy_temp

# Apply patch
#COPY ${NGINX_PATCH_FILENAME}.patch /app/${NGINX_PATCH_FILENAME}.patch
#RUN cd /app/nginx-${NGINX_VERSION} && patch -p01 < /app/${NGINX_PATCH_FILENAME}.patch

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
  --with-http_stub_status_module \
  --without-http_ssi_module \
  --without-http_userid_module \
  --without-http_mirror_module \
  --without-http_autoindex_module \
  --without-http_split_clients_module \
  --without-http_fastcgi_module \
  --without-http_uwsgi_module \
  --without-http_scgi_module \
  --without-http_grpc_module \
  --without-http_gzip_module \
  --without-http_memcached_module \
  --http-log-path=/var/log/nginx/access.log \
  --http-client-body-temp-path=/var/cache/nginx/client_temp \
  --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
  --without-mail_pop3_module \
  --without-mail_imap_module \
  --without-mail_smtp_module \
  --add-module=/app/ngx_devel_kit-${NGINX_DEVEL_NGINX_VERSION} \
  --add-module=/app/set-misc-nginx-module-${SET_MISC_NGINX_VERSION} \
  --add-module=/app/headers-more-nginx-module-${HEADERS_MORE_NGINX_VERSION}  \
  --with-cc-opt=-O2 \
  --with-ld-opt='-Wl,-rpath,/usr/local/lib' \
  --with-pcre=/app/pcre2-${PCRE2_VERSION} \
  --with-pcre-jit \
  --with-zlib=/app/zlib-${ZLIB_VERSION} \
  --with-openssl=/app/openssl-${OPENSSL_VERSION} \
  --with-openssl-opt=no-nextprotoneg \
  --with-debug && \
  echo $(eval $CONFIGURE) && \
  make -j$(nproc) && make install


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
 FROM gcr.io/distroless/base:nonroot as nginx

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