![Logo](Logo.png)

# nginx-shadow

## NGINX Custom Build with HTTP/3 (QUIC) and RTMP Support

This project contains a custom NGINX build with additional support for HTTP/3 (QUIC) and RTMP streaming protocol. Additionally, it includes a patch to remove server information from the HTTP headers for improved security.

## Changes compared to the original version of NGINX

- [X] Configure the Dockerfile to compile NGINX with the latest version and up-to-date core dependencies.
    - [X] stable [nginx (1.26.2)](https://nginx.org/en/download.html)
    - [X] [boringssl](https://github.com/google/boringssl/commits/master/) 
    - [X] [pcre2 (10.44)](https://github.com/PCRE2Project/pcre2/releases)
    - [X] [zlib](https://github.com/cloudflare/zlib/commits/gcc.amd64/) [(cloudflare)](https://blog.cloudflare.com/cloudflare-fights-cancer/) 
- [X] Create a patch to remove the Server header and change default and error messages in the NGINX source to enhance security by preventing the server's name and version from being exposed. [[See Details]](#changes-in-nginx-source-code)
- [ ] Nginx built-in modules selection.
    - [X] Add support for HTTP/3 (QUIC)
    - [ ] Hardening
- [ ] Nginx Third-party modules selection.
    - [ ] Ensure support for RTMP (Real-Time Messaging Protocol).
    - [ ] Add support for VTS (Virtual Traffic Status) and STS (Stream TLS) modules.
    - [ ] Add Brotli compression support, powered by ngx_brotli.
    - [ ] Add GeoIP2 support, powered by ngx_http_geoip2_module.
    - [ ] Add Headers More support, powered by ngx_headers_more.

## Changes in Nginx Source Code

In the patch applied to the Nginx source code during the image build process to make it harder to determine the name and version of the web server, the following changes were implemented:

- **Removal of the ``Server`` header:** The source code was modified to either remove or mask the Server header in HTTP responses. By default, Nginx sends the Server header containing the name "nginx" and its version. This patch ensures that the Server header is no longer included in the response, preventing external users from discovering the server's identity.
- **Modification of static responses such as ``"Welcome to nginx!"``:** Nginx often returns default static responses that reveal it is running, such as the "Welcome to nginx!" message displayed on default or unconfigured pages. This patch changes those default responses to either remove the reference to Nginx or replace it with a generic message to obscure the server's identity.
- **Modification of error responses and their statuses:** The patch alters the way Nginx handles HTTP error responses, such as ``404 (Not Found)`` or ``500 (Internal Server Error)``. Typically, Nginx may include the server's name in these error messages. The patch modifies these responses so that they no longer include references to Nginx or its version. Additionally, the error status messages are made more generic, ensuring no information about the server is leaked in error responses.


These changes improve the security of the server by preventing automatic detection tools from identifying the server software and version, thereby reducing the risk of targeted attacks.

### Build information

#### Core dependencies

| Name                | Version/Commit/Tag                              | Repo/Site                                               | Info                                    |
|---------------------|-------------------------------------------------|---------------------------------------------------------|----------------------------------------|
| **nginx**           | [1.26.2](https://github.com/nginx/nginx/tree/release-1.26.2)    | [nginx](https://github.com/nginx/nginx) | High-performance web server and reverse proxy                            |
| **boringssl**       | [571c76e](https://github.com/google/boringssl/commit/571c76e919c0c48219ced35bef83e1fc83b00eed) | [boringssl](https://github.com/google/boringssl) | Optimized SSL library enabling HTTP/3 (QUIC) |
| **pcre2**           | [10.44](https://github.com/PCRE2Project/pcre2/releases/tag/pcre2-10.44) | [pcre2](https://github.com/PCRE2Project/pcre2) | Advanced regular expression matching library |
| **zlib-cf**         | [872e5fb](https://github.com/cloudflare/zlib/commit/872e5fb3cf88bb281e19a8327b3ea0889cc34773) | [zlib-cf](https://github.com/cloudflare/zlib) | Zlib optimized by Cloudflare, incompatible with 32-bit processors|

#### Modules


| Name| Version/Commit/Tag | Repo/Site                                               | Description                                                                                       |
|---------------------------|-------------------------------------|-------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|
| **ngx_headers_more**         | 0.37                                 | [ngx_headers_more](https://github.com/openresty/headers-more-nginx-module) | Provides additional control over HTTP headers, allowing modifications to headers sent by NGINX     |
| **ngx_devel_kit**  | 0.3.3                                | [ngx_devel_kit](https://github.com/vision5/ngx_devel_kit)         | Toolkit for extending NGINX with custom directives and additional features                         |
| **ngx_set_misc**             | 0.33                                 | [ngx_set_misc](https://github.com/openresty/set-misc-nginx-module)    | Adds complex variable manipulation capabilities, enhancing NGINX configuration flexibility         |


### Commands

**Create Patch**
```
cd sandbox
```
```
diff -ruN nginx-1.26.2/ nginx-1.26.2_changes/ > ../build/nginx-1.26.2.patch
```
**Docker | Build image**
```
cd build
```
```
docker build . -t nginx-shadow:0.0.1 --progress=plain 
```

**Docker | Run container with image**
```
docker run nginx-shadow:0.0.1    
```