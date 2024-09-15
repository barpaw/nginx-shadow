# ----------------------------------------------------------- #
# -----------------nginx-secured by barpaw------------------- #
# ----------------------------------------------------------- #

# -----------------STAGE 1 modsecurity-build----------------- #

FROM ubuntu:22.04 AS modsecurity-build
LABEL maintainer="barpaw <barpaw@gmail.com>"

# ModSecurity dependencies
ENV MODSECURITY_VERSION=3.0.7
ENV YAJL_VERSION=2.1.0

RUN mkdir /app
WORKDIR /app

# Set TimeZone
ENV TZ=Europe/Warsaw
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Download needed packages from repository
RUN apt-get update && apt-get install -y \
  ca-certificates=20211016 \
  automake=1:1.16.5-1.3 \
  autoconf=2.71-2 \
  build-essential=12.9ubuntu3 \
  libcurl4-openssl-dev=7.81.0-1ubuntu1.3 \
  libpcre++-dev=0.9.5-7 \
  libtool=2.4.6-15build2 \
  libxml2-dev=2.9.13+dfsg-1ubuntu0.1 \
  libyajl-dev=2.1.0-3build2 \
  lua5.2-dev=5.2.4-2 \
  pkgconf=1.8.0-1 \
  ssdeep=2.14.1+git20180629.57fcfff-2 \
  libgeoip-dev=1.6.12-8 \
  tree=2.0.2-1 \
  curl=7.81.0-1ubuntu1.3 \
  git=1:2.34.1-1ubuntu1.4 \
  cmake=3.22.1-1ubuntu1 \
  wget=1.21.2-2ubuntu1 \
  gcc=4:11.2.0-1ubuntu1 \
  make=4.3-4.1build1 \
  unzip=6.0-26ubuntu3

# Download & extract ModSecurity dependencies
RUN wget "https://github.com/lloyd/yajl/archive/refs/tags/${YAJL_VERSION}.tar.gz" -O /app/yajl.tar.gz && tar xzvf /app/yajl.tar.gz

# Install ModSecurity dependencies
RUN cd yajl-${YAJL_VERSION} && \
  ./configure && \
  make && \
  make install && \
  export PKG_CONFIG_PATH=/usr/local/share/pkgconfig:PKG_CONFIG_PATH

#Download & Build ModSecurity
RUN wget "https://github.com/SpiderLabs/ModSecurity/releases/download/v${MODSECURITY_VERSION}/modsecurity-v${MODSECURITY_VERSION}.tar.gz" -O /app/modsecurity.tar.gz && tar xzvf /app/modsecurity.tar.gz && \
  cd modsecurity-v${MODSECURITY_VERSION} && \
  ./build.sh && \
  ./configure --with-yajl="/usr/local/" && \
  make -j2 && \
  make install

# Clean
RUN strip /usr/local/modsecurity/bin/* /usr/local/modsecurity/lib/*.a /usr/local/modsecurity/lib/*.so*

# -----------------STAGE 2 nginx-build----------------- #

FROM ubuntu:22.04 AS nginx-build

RUN mkdir /app
WORKDIR /app

# NGINX CORE
ENV NGINX_VERSION=1.22.0
ENV OPENSSL_VERSION=3.0.5
ENV PCRE2_VERSION=10.40
ENV ZLIB_VERSION=1.2.11

# 3RD PARTY MODULES
ENV MOD_SECURITY_NGINX_VERSION=1.0.3
ENV OWASP_CRS_NGINX_VERSION=3.3.2
ENV HEADERS_MORE_NGINX_VERSION=0.34
ENV NGINX_DEVEL_NGINX_VERSION=0.3.1
ENV SET_MISC_NGINX_VERSION=0.33

# NGINX PATCH FILENAME
ENV NGINX_PATCH_FILENAME="nginx-1.22.0"

# Set TimeZone
ENV TZ=Europe/Warsaw
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Add nginx user
RUN groupadd -r nonroot && useradd -r -s /bin/false -g nonroot nonroot

# Download needed packages from repository
RUN apt-get update && apt-get install -y \
  build-essential=12.9ubuntu3 \
  tree=2.0.2-1 \
  curl=7.81.0-1ubuntu1.3 \
  git=1:2.34.1-1ubuntu1.4 \
  cmake=3.22.1-1ubuntu1 \
  wget=1.21.2-2ubuntu1 \
  gcc=4:11.2.0-1ubuntu1 \
  make=4.3-4.1build1 \
  unzip=6.0-26ubuntu3 \
  ca-certificates=20211016 \
  autoconf=2.71-2 \
  automake=1:1.16.5-1.3 \
  libtool=2.4.6-15build2 \
  pkgconf=1.8.0-1 \
  zlib1g-dev=1:1.2.11.dfsg-2ubuntu9 \
  libssl-dev=3.0.2-0ubuntu1.6  \
  libpcre3-dev=2:8.39-13ubuntu0.22.04.1 \
  libxml2-dev=2.9.13+dfsg-1ubuntu0.1 \
  libyajl-dev=2.1.0-3build2 \
  lua5.2-dev=5.2.4-2 \
  libgeoip-dev=1.6.12-8 \
  libcurl4-openssl-dev=7.81.0-1ubuntu1.3 \
  openssl=3.0.2-0ubuntu1.6

# Download dependencies
RUN wget "https://github.com/PCRE2Project/pcre2/releases/download/pcre2-${PCRE2_VERSION}/pcre2-${PCRE2_VERSION}.tar.gz" -O /app/pcre2.tar.gz && tar xzvf /app/pcre2.tar.gz && \
  wget "https://github.com/SpiderLabs/ModSecurity-nginx/releases/download/v${MOD_SECURITY_NGINX_VERSION}/modsecurity-nginx-v${MOD_SECURITY_NGINX_VERSION}.tar.gz" -O /app/modsecuritynginxconnector.tar.gz && tar xzvf /app/modsecuritynginxconnector.tar.gz && \
  wget "https://github.com/coreruleset/coreruleset/archive/refs/tags/v${OWASP_CRS_NGINX_VERSION}.tar.gz" -O /app/coreruleset.tar.gz && tar xzvf /app/coreruleset.tar.gz && \
  wget "https://github.com/openresty/headers-more-nginx-module/archive/refs/tags/v${HEADERS_MORE_NGINX_VERSION}.tar.gz" -O /app/headers_more.tar.gz && tar xzvf /app/headers_more.tar.gz && \
  wget "https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz" -O /app/openssl.tar.gz && tar xzvf /app/openssl.tar.gz && \
  wget "https://github.com/madler/zlib/archive/refs/tags/v${ZLIB_VERSION}.tar.gz" -O /app/zlib.tar.gz && tar xzvf /app/zlib.tar.gz && \
  wget "https://github.com/openresty/set-misc-nginx-module/archive/refs/tags/v${SET_MISC_NGINX_VERSION}.tar.gz" -O /app/set_misc.tar.gz && tar xzvf /app/set_misc.tar.gz && \
  wget "https://github.com/vision5/ngx_devel_kit/archive/refs/tags/v${NGINX_DEVEL_NGINX_VERSION}.tar.gz" -O /app/nginx_devel.tar.gz && tar xzvf /app/nginx_devel.tar.gz && \
  wget "https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" -O /app/nginx.tar.gz && tar xzvf /app/nginx.tar.gz

# Create temp folders
RUN mkdir -p /var/cache/nginx/client_temp && \
  mkdir -p /var/cache/nginx/proxy_temp

#Copy modsecurity
COPY --from=modsecurity-build /usr/local/modsecurity/ /usr/local/modsecurity/

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
  --build=nginx-secured \
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
  --add-module=/app/modsecurity-nginx-v${MOD_SECURITY_NGINX_VERSION} \
  --with-cc-opt=-O2 \
  --with-ld-opt='-Wl,-rpath,/usr/local/lib' \
  --with-pcre=/app/pcre2-${PCRE2_VERSION} \
  --with-pcre-jit \
  --with-zlib=/app/zlib-${ZLIB_VERSION} \
  --with-openssl=/app/openssl-${OPENSSL_VERSION} \
  --with-openssl-opt=no-nextprotoneg \
  --with-debug && \
  echo $(eval $CONFIGURE) && \
  make -j2 && make && make install

# Create ModSecurity folders
RUN mkdir -p /etc/nginx/modsecurity.d && \
  mkdir -p /var/log/modsec/

# Copy ModSecurity OWASP Rules
RUN mv /app/coreruleset-${OWASP_CRS_NGINX_VERSION}/ /etc/nginx/modsecurity.d/

RUN chown -R nonroot:nonroot /etc/nginx
RUN chown -R nonroot:nonroot /etc/ssl/
RUN chown -R nonroot:nonroot /var/log/nginx/
RUN chown -R nonroot:nonroot /var/log/modsec/
RUN chown -R nonroot:nonroot /var/cache/nginx/
RUN chown -R nonroot:nonroot /var/run/

# Forward logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
  && ln -sf /dev/stderr /var/log/nginx/error.log \
  && ln -sf /dev/stderr /var/log/modsec/modsec_audit.log


## BAD BOTS BLOCKER
## https://github.com/mitchellkrogza/nginx-ultimate-bad-bot-blocker
## https://github.com/mitchellkrogza/nginx-ultimate-bad-bot-blocker/blob/master/MANUAL-CONFIGURATION.md

RUN mkdir -p /etc/nginx/conf.d
RUN wget https://raw.githubusercontent.com/mitchellkrogza/nginx-ultimate-bad-bot-blocker/master/conf.d/globalblacklist.conf -O /etc/nginx/conf.d/globalblacklist.conf
RUN mkdir -p /etc/nginx/bots.d
RUN wget https://raw.githubusercontent.com/mitchellkrogza/nginx-ultimate-bad-bot-blocker/master/bots.d/blockbots.conf -O /etc/nginx/bots.d/blockbots.conf
RUN wget https://raw.githubusercontent.com/mitchellkrogza/nginx-ultimate-bad-bot-blocker/master/bots.d/ddos.conf -O /etc/nginx/bots.d/ddos.conf
RUN wget https://raw.githubusercontent.com/mitchellkrogza/nginx-ultimate-bad-bot-blocker/master/bots.d/whitelist-ips.conf -O /etc/nginx/bots.d/whitelist-ips.conf
RUN wget https://raw.githubusercontent.com/mitchellkrogza/nginx-ultimate-bad-bot-blocker/master/bots.d/whitelist-domains.conf -O /etc/nginx/bots.d/whitelist-domains.conf
RUN wget https://raw.githubusercontent.com/mitchellkrogza/nginx-ultimate-bad-bot-blocker/master/bots.d/blacklist-user-agents.conf -O /etc/nginx/bots.d/blacklist-user-agents.conf
RUN wget https://raw.githubusercontent.com/mitchellkrogza/nginx-ultimate-bad-bot-blocker/master/bots.d/custom-bad-referrers.conf -O /etc/nginx/bots.d/custom-bad-referrers.conf
RUN wget https://raw.githubusercontent.com/mitchellkrogza/nginx-ultimate-bad-bot-blocker/master/bots.d/blacklist-ips.conf -O /etc/nginx/bots.d/blacklist-ips.conf
RUN wget https://raw.githubusercontent.com/mitchellkrogza/nginx-ultimate-bad-bot-blocker/master/bots.d/bad-referrer-words.conf -O /etc/nginx/bots.d/bad-referrer-words.conf
RUN wget https://raw.githubusercontent.com/mitchellkrogza/nginx-ultimate-bad-bot-blocker/master/conf.d/botblocker-nginx-settings.conf -O /etc/nginx/conf.d/botblocker-nginx-settings.conf


# -----------------STAGE 3 final stage------------------------- #


# Grab the distroless static container.
FROM gcr.io/distroless/base:nonroot as nginx

# Set the container timezone as Europe/Warsaw.
ENV TZ=Europe/Warsaw

# Use this command to list of dependency libraries you should copy:
# command: ldd /usr/sbin/nginx

COPY --from=nginx-build /var/log /var/log
COPY --from=nginx-build /var/cache/nginx /var/cache/nginx
COPY --from=nginx-build /var/run /var/run
# Copy the nginx configuration and binary from build image.
COPY --from=nginx-build /etc/nginx /etc/nginx
COPY --from=nginx-build /usr/sbin/nginx /usr/bin/nginx
# # Copy the necessary dependencies from build image to run nginx properly in static container.
COPY --from=nginx-build /lib/x86_64-linux-gnu/libdl.so.2 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libpthread.so.0 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libcrypt.so.1 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libc.so.6 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libcurl.so.4 /lib/x86_64-linux-gnu/

COPY --from=nginx-build /lib/x86_64-linux-gnu/libGeoIP.so.1 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libxml2.so.2 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/liblua5.2.so.0 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libpcre.so.3 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libyajl.so.2 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libstdc++.so.6 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libgcc_s.so.1 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libnghttp2.so.14 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libidn2.so.0 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/librtmp.so.1 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libssh.so.4 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libpsl.so.5 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libssl.so.3 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libcrypto.so.3 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libgssapi_krb5.so.2 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libldap-2.5.so.0 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/liblber-2.5.so.0 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libzstd.so.1 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libbrotlidec.so.1 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libz.so.1 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libicuuc.so.70 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/liblzma.so.5 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libm.so.6 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libunistring.so.2 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libgnutls.so.30 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libhogweed.so.6 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libnettle.so.8 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libgmp.so.10 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libkrb5.so.3 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libk5crypto.so.3 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libcom_err.so.2 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libkrb5support.so.0 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libsasl2.so.2 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libbrotlicommon.so.1 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libicudata.so.70 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libp11-kit.so.0 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libtasn1.so.6 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libkeyutils.so.1 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libresolv.so.2 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libffi.so.8 /lib/x86_64-linux-gnu/


COPY --from=nginx-build /lib64/ld-linux-x86-64.so.2 /lib64/
# # This is important for nginx as it is using libnss to query user information from `/etc/passwd` otherwise you will receive error like `nginx: [emerg] getpwnam("nonroot") failed`.
COPY --from=nginx-build /lib/x86_64-linux-gnu/libnss_compat.so.2 /lib/x86_64-linux-gnu/
COPY --from=nginx-build /lib/x86_64-linux-gnu/libnss_files.so.2 /lib/x86_64-linux-gnu/

# Copy ModSecuirty
COPY --from=nginx-build /usr/local/modsecurity/ /usr/local/modsecurity/


# NGINX needs to be started as root. (for master proces)
# Only root processes can listen to ports below 1024
USER root

EXPOSE 80/tcp 443/tcp

STOPSIGNAL SIGTERM

CMD ["nginx", "-g", "daemon off;"]