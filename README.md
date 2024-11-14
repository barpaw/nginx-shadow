![Logo](Logo.png)

# **nginx-shadow**

## **Custom NGINX Build with HTTP/3 (QUIC) Support**

`nginx-shadow` is a custom NGINX build that includes support for HTTP/3 (QUIC) along with additional modules and patches to enhance server security. This build conceals server details by hiding the `Server` header and masking version information, making it harder for attackers to identify the server software.

---

## **Key Changes Compared to the Original NGINX**

### ✅ **Modifications in the Dockerfile:**
- Compiling NGINX with the latest version and updated core dependencies:
  - **NGINX**: [1.26.2](https://nginx.org/en/download.html)
  - **PCRE2**: [10.44](https://github.com/PCRE2Project/pcre2/releases)
  - **BoringSSL**: [commit](https://github.com/google/boringssl/commits/master/)
  - **Zlib (Cloudflare)**: [zlib-cf](https://github.com/cloudflare/zlib/commits/gcc.amd64)
- Added a custom patch to remove the `Server` header and modify default error messages to prevent the exposure of server details.

### 🛠 **NGINX Module Selection:**
- **Built-in Modules**:
  - ✅ Support for HTTP/3 (QUIC)
  - ⏳ Hardening (in progress)
- **Third-party Modules**:
  - ✅ **VTS (Virtual Traffic Status)** for HTTP traffic monitoring.
  - ✅ **STS (Stream TLS Status)** for monitoring TLS streams.
  - ✅ **Stream STS** for stream protocol monitoring.
  - ⏳ Brotli compression (ngx_brotli).
  - ⏳ GeoIP2 support (ngx_http_geoip2_module).
  - ⏳ Headers More (ngx_headers_more).

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

| **Name**            | **Version/Commit/Tag**                        | **Repository**                                             | **Description**                                     |
|---------------------|----------------------------------------------|------------------------------------------------------------|-----------------------------------------------------|
| **nginx**           | [1.26.2](https://github.com/nginx/nginx/tree/release-1.26.2) | [nginx](https://github.com/nginx/nginx) | High-performance web server and reverse proxy      |
| **boringssl**       | [571c76e](https://github.com/google/boringssl/commit/571c76e919c0c48219ced35bef83e1fc83b00eed) | [boringssl](https://github.com/google/boringssl) | Optimized SSL library enabling HTTP/3 (QUIC)       |
| **pcre2**           | [10.44](https://github.com/PCRE2Project/pcre2/releases/tag/pcre2-10.44) | [pcre2](https://github.com/PCRE2Project/pcre2) | Advanced regular expression matching library       |
| **zlib-cf**         | [872e5fb](https://github.com/cloudflare/zlib/commit/872e5fb3cf88bb281e19a8327b3ea0889cc34773) | [zlib-cf](https://github.com/cloudflare/zlib) | Cloudflare-optimized zlib (incompatible with 32-bit) |

---

### **Modules**

| **Name**                | **Version/Commit/Tag**                       | **Repository**                                             | **Description**                                    |
|-------------------------|----------------------------------------------|------------------------------------------------------------|----------------------------------------------------|
| **ngx_headers_more**    | 0.37                                         | [ngx_headers_more](https://github.com/openresty/headers-more-nginx-module) | Control over HTTP headers                          |
| **ngx_set_misc**        | 0.33                                         | [ngx_set_misc](https://github.com/openresty/set-misc-nginx-module) | Adds advanced variable manipulation capabilities   |
| **nginx-module-vts**    | 0.2.2                                        | [nginx-module-vts](https://github.com/vozlt/nginx-module-vts) | HTTP traffic monitoring                            |
| **nginx-module-sts**    | 0.1.1                                        | [nginx-module-sts](https://github.com/vozlt/nginx-module-sts) | Stream TLS traffic monitoring                      |
| **nginx-module-stream-sts**          | 0.1.1                                        | [nginx-module-stream-sts](https://github.com/vozlt/nginx-module-stream-sts) | Stream protocol monitoring                         |

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

## **Useful Links**

- [Official NGINX Documentation](https://nginx.org/en/docs/)
- [HTTP/3 Explained](https://quic.nginx.org/)
- [Cloudflare Blog](https://blog.cloudflare.com/)

---

## **License**

This project is released under the **MIT License**.

---
