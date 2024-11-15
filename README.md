![Logo](Logo.png)

# 🥷🏿 **nginx-shadow** 🥷🏿

## **Custom NGINX Build with HTTP/3 (QUIC) Support** 

`nginx-shadow` is a custom NGINX build that includes support for HTTP/3 (QUIC) along with additional modules and patches to enhance server security. This build conceals server details by hiding the `Server` header and masking version information, making it harder for attackers to identify the server software.

---

## **Key Changes Compared to the Original NGINX**

### 👷‍♂️ **Own Nginx Build (Dockerfile)**
- Compiling NGINX with the latest stable version and updated core dependencies:
  - **NGINX**: [1.26.2](https://nginx.org/en/download.html)
  - **PCRE2**: [10.44](https://github.com/PCRE2Project/pcre2/releases)
  - **Zlib**: [1.3.1](https://github.com/madler/zlib/releases)
  - **BoringSSL**: [commit](https://github.com/google/boringssl/commits/master/)
- [Changes in NGINX source code](#changes-in-nginx-source-code)

### 🛠 **NGINX Module Selection:**
- **Removed mountains of useless modules to improve performance and security**
- **Built-in Modules**:
  - ✅ **`select_module`**: Provides support for the `select()` system call for event handling.
  - ✅ **`poll_module`**: Provides support for the `poll()` system call for event handling.
  - ✅ **`thread_pool_module`**: Enables support for multi-threaded processing.
  - ✅ **`file_aio_module`**: Enables asynchronous file I/O for improved disk performance.
  - ✅ **`http_ssl_module`**: Adds HTTPS support using SSL/TLS.
  - ✅ **`http_v2_module`**: Implements support for the HTTP/2 protocol.
  - ✅ **`http_v3_module`**: Provides support for HTTP/3 (QUIC), enabling modern transport protocols.
  - ✅ **`http_realip_module`**: Allows handling of client IP addresses from reverse proxy headers.
  - ✅ **`http_auth_request_module`**: Facilitates external authentication mechanisms.
  - ✅ **`http_stub_status_module`**: Displays server statistics for real-time performance monitoring.
  - ✅ **`stream_module`**: Supports proxying of TCP/UDP streams for non-HTTP protocols.
  - ✅ **`stream_ssl_module`**: Adds SSL/TLS support to the Stream module for secure streaming.
  - ✅ **`stream_realip_module`**: Extends the Real IP handling to Stream connections.

- **Third-party Modules**:
    - ✅ **Headers More**: Modify HTTP headers for enhanced control.
    - ✅ **Brotli**: High-performance compression for faster content delivery.
    - ✅ **GeoIP2**: Location-based routing and access control.
    - ✅ **VTS/STS/Stream STS**: HTTP/TLS/STREAM traffic monitoring.
    - ✅ **Devel Kit**: Adds advanced development features for custom extensions.
    - ✅ **Set Misc**: Provides additional variable manipulation capabilities.
    - ⏳ **TBA**: TBA

### 🌿 **Distroless Image**
  - `gcr.io/distroless/base:nonroot`

---

## **Changes in NGINX Source Code**

During the build process, a patch is applied to the NGINX source code with the following modifications:

- **Removal of the `Server` Header**:
  - By default, NGINX sends the `Server` header containing its name and version in HTTP responses. This patch removes the header to prevent server identification by attackers.
  
- **Modification of Default Static Responses**:
  - The default `"Welcome to nginx!"` message and other static content are altered to conceal any references to NGINX.

- **Modification of Error Responses**:
  - Error messages like `404 Not Found` and `500 Internal Server Error` are adjusted to remove any references to NGINX or its version. This ensures that no server information is exposed in error responses.

These changes enhance the server's security by preventing automated tools from detecting the server type and version, reducing the risk of targeted attacks.

---

## **Build Information**

### **Core Dependencies**

| **Name**       | **Version/Commit/Tag**                             | **Repository**                                              | **Description**                                              |
|----------------|---------------------------------------------------|-------------------------------------------------------------|--------------------------------------------------------------|
| **NGINX**      | [1.26.2](https://nginx.org/en/download.html) | [nginx](https://github.com/nginx/nginx/tree/release-1.26.2) | High-performance web server and reverse proxy                |
| **BoringSSL**  | [c691779](https://github.com/google/boringssl/commit/c691779ed0e98b36eff7ad945a738c402f127122) | [boringssl](https://github.com/google/boringssl)            | Optimized SSL library for enhanced security and HTTP/3 (QUIC) |
| **PCRE2**      | [10.44](https://github.com/PCRE2Project/pcre2/releases/tag/pcre2-10.44) | [pcre2](https://github.com/PCRE2Project/pcre2)             | Advanced regular expression matching with JIT optimization    |
| **Zlib**       | [1.3.1](https://github.com/madler/zlib/releases/tag/v1.3.1) | [zlib](https://github.com/madler/zlib)                     | General-purpose compression library for gzip and HTTP/2       |

--- 

### **Modules**

| **Name**                       | **Version/Commit/Tag**                       | **Repository**                                             | **Description**                                          |
|--------------------------------|----------------------------------------------|------------------------------------------------------------|----------------------------------------------------------|
| **ngx_headers_more**           | [0.37](https://github.com/openresty/headers-more-nginx-module/tags) | [ngx_headers_more](https://github.com/openresty/headers-more-nginx-module) | Allows for modifying HTTP headers dynamically            |
| **ngx_set_misc**               | [0.33](https://github.com/openresty/set-misc-nginx-module/tags) | [ngx_set_misc](https://github.com/openresty/set-misc-nginx-module) | Provides advanced variable and data manipulation capabilities |
| **nginx-module-vts**           | [0.2.2](https://github.com/vozlt/nginx-module-vts/tags) | [nginx-module-vts](https://github.com/vozlt/nginx-module-vts) | Monitors and visualizes HTTP traffic statistics           |
| **nginx-module-sts**           | [0.1.1](https://github.com/vozlt/nginx-module-sts/tags) | [nginx-module-sts](https://github.com/vozlt/nginx-module-sts) | TLS traffic monitoring for HTTP                           |
| **nginx-module-stream-sts**    | [0.1.1](https://github.com/vozlt/nginx-module-stream-sts/tags) | [nginx-module-stream-sts](https://github.com/vozlt/nginx-module-stream-sts) | Monitors stream protocol traffic                          |
| **ngx_brotli**                 | [a71f931](https://github.com/google/ngx_brotli/commit/a71f9312c2deb28875acc7bacfdd5695a111aa53) | [ngx_brotli](https://github.com/google/ngx_brotli)         | High-performance compression using the Brotli algorithm   |
| **ngx_http_geoip2_module**     | [3.4](https://github.com/leev/ngx_http_geoip2_module/tags) | [ngx_http_geoip2_module](https://github.com/leev/ngx_http_geoip2_module) | Provides IP geolocation-based routing and access control  |
| **ngx_devel_kit**              | [0.3.3](https://github.com/vision5/ngx_devel_kit/tags) | [ngx_devel_kit](https://github.com/vision5/ngx_devel_kit)   | Provides essential utilities for developing advanced NGINX modules |

--- 


### Output of ```nginx -V```

```
nginx version:  (nginx-shadow)
built by gcc 13.2.1 20240309 (Alpine 13.2.1_git20240309) 
built with OpenSSL 1.1.1 (compatible; BoringSSL) (running with BoringSSL)
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --user=nonroot --group=nonroot --build=nginx-shadow --with-select_module --with-poll_module --with-threads --with-file-aio --with-http_ssl_module --with-http_v2_module --with-http_v3_module --with-http_realip_module --with-http_auth_request_module --with-http_stub_status_module --with-stream --with-stream_ssl_module --with-stream_realip_module --without-http_ssi_module --without-http_userid_module --without-http_mirror_module --without-http_autoindex_module --without-http_split_clients_module --without-http_fastcgi_module --without-http_uwsgi_module --without-http_scgi_module --without-http_grpc_module --without-http_memcached_module --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --without-mail_pop3_module --without-mail_imap_module --without-mail_smtp_module --add-module=/app/ngx_devel_kit --add-module=/app/set-misc-nginx-module --add-module=/app/headers-more-nginx-module --add-module=/app/nginx-module-vts --add-module=/app/nginx-module-sts --add-module=/app/nginx-module-stream-sts --add-module=/app/ngx_brotli --add-module=/app/ngx_http_geoip2_module --with-cc=c++ --with-cc-opt='-I/app/boringssl/include -I/app/pcre2/include -O2 -x c' --with-ld-opt='-L/app/ngx_brotli/deps/brotli/out -L/app/boringssl/build/ssl -L/app/boringssl/build/crypto -L/app/pcre2/lib -Wl,-rpath,/app/boringssl/build/ssl' --with-pcre-jit --with-zlib=/app/zlib --with-debug
```
---

## **Build Instructions**

### **Creating the Patch**

To create a patch for NGINX:

```bash
cd sandbox
diff -ruN nginx-1.26.2/ nginx-1.26.2_changes/ > ../build/nginx-1.26.2.patch
```

### **Building the Docker Image**

To build the Docker image:

```bash
cd build
docker build . -t nginx-shadow:0.0.1 --progress=plain
```

### **Running the Docker Container**

To run the container using the built image:

```bash
docker run -d -p 80:80 -p 443:443 --name nginx-shadow nginx-shadow:0.0.1
```

---
## Recommended NGINX config

```nginx
http {
  brotli on;
  gzip on;
  http2 on;
  http3 on;
  quic_gso on;
  quic_retry on;
  ssl_certificate /path/to/cert_plus_intermediate;
  ssl_certificate_key /path/to/key;
  ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305; # change `ECDSA` to `RSA` if you use RSA certificate
  ssl_early_data on;
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_session_cache shared:SSL:10m;
  ssl_session_timeout 1d;
  ssl_stapling on;
  ssl_stapling_verify on;
  server {
    listen 80 reuseport;
    listen [::]:80 reuseport; # delete if ipv6 is unavailable
    return 444;
  }
  server {
    listen 443 reuseport ssl;
    listen [::]:443 reuseport ssl;
    listen 443 reuseport quic;
    listen [::]:443 reuseport quic;
    ssl_reject_handshake on;
  }
  server {
    listen 80;
    listen [::]:80;
    server_name example.com dynamic.example.com php.example.com www.example.com;
    return 308 https://$host$request_uri;
  }
  server { # example for static site
    listen 443;
    listen [::]:443;
    listen 443 quic;
    listen [::]:443 quic;
    server_name example.com;
    root /path/to/static/site;
    add_header Alt-Svc 'h3=":443"; ma=86400';
  }
  server { # example for dynamic site
    listen 443;
    listen [::]:443;
    listen 443 quic;
    listen [::]:443 quic;
    server_name dynamic.example.com;
    add_header Alt-Svc 'h3=":443"; ma=86400';
    location / {
      proxy_pass http://ip:port;
    }
  }
  server { # example for dynamic site with php
    listen 443;
    listen [::]:443;
    listen 443 quic;
    listen [::]:443 quic;
    server_name php.example.com;
    root /path/to/php/site;
    index index.php;
    add_header Alt-Svc 'h3=":443"; ma=86400';
    location ~ ^.+\.php$ {
      include fastcgi_params;
      fastcgi_param HTTP_PROXY '';
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      fastcgi_pass unix:/path/to/php/sock;
    }
  }
  server {
    listen 443;
    listen [::]:443;
    listen 443 quic;
    listen [::]:443 quic;
    server_name www.example.com;
    add_header Alt-Svc 'h3=":443"; ma=86400';
    return 308 https://example.com$request_uri;
  }
}
```
---

## **Docker Compose**

TODO

---

## **CertBot**

TODO

---

## **Monitoring (STS/VTS)**

TODO

---

## **WAF (ModSecurity)**

TODO

[#1](https://github.com/owasp-modsecurity/ModSecurity-nginx)

[#2](https://www.elastic.co/docs/current/integrations/modsecurity)

---

## **Useful Links**

- [Official NGINX Documentation](https://nginx.org/en/docs/)
- [HTTP/3 Check](https://quic.nginx.org/)
- [HTTP/3 Explained](https://nginx.org/en/docs/http/ngx_http_v3_module.html)
- [Cloudflare blog post about their zlib lib](https://blog.cloudflare.com/cloudflare-fights-cancer/) 

---

## **License**

This project is released under the **MIT License**.

---

