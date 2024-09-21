![Logo](Logo.png)

# nginx-shadow

## NGINX Custom Build with HTTP/3 and RTMP Support

This project contains a custom NGINX build with additional support for HTTP/3 (QUIC) and RTMP streaming protocol. Additionally, it includes a patch to remove server information from the HTTP headers for improved security.

## To-Do List

- [X] **Prepare a build with the latest version of NGINX and the latest dependencies.**
    - [ ] **Check "cloudflare" deps**
- [X] **Create a patch to remove NGINX server name and version information from the source code.**
- [ ] **Ensure support for HTTP/3 (QUIC).**
- [ ] **Ensure support for RTMP (Real-Time Messaging Protocol).**
- [ ] **Add support for STS (Stream TLS) and VTS (Virtual Traffic Status) modules.**
- [ ] **Add Brotli compression support, powered by ngx_brotli.**
- [ ] **Add GeoIP2 support, powered by ngx_http_geoip2_module.**
- [ ] **Add Headers More support, powered by ngx_headers_more.**
- [ ] **Remove unnecessary modules to improve performance.**

## Changes in Nginx Source Code

In the patch applied to the Nginx source code during the image build process to make it harder to determine the name and version of the web server, the following changes were implemented:

- **Removal of the ``Server`` header:** The source code was modified to either remove or mask the Server header in HTTP responses. By default, Nginx sends the Server header containing the name "nginx" and its version. This patch ensures that the Server header is no longer included in the response, preventing external users from discovering the server's identity.
- **Modification of static responses such as ``"Welcome to nginx!"``:** Nginx often returns default static responses that reveal it is running, such as the "Welcome to nginx!" message displayed on default or unconfigured pages. This patch changes those default responses to either remove the reference to Nginx or replace it with a generic message to obscure the server's identity.
- **Modification of error responses and their statuses:** The patch alters the way Nginx handles HTTP error responses, such as ``404 (Not Found)`` or ``500 (Internal Server Error)``. Typically, Nginx may include the server's name in these error messages. The patch modifies these responses so that they no longer include references to Nginx or its version. Additionally, the error status messages are made more generic, ensuring no information about the server is leaked in error responses.


These changes improve the security of the server by preventing automatic detection tools from identifying the server software and version, thereby reducing the risk of targeted attacks.

### Commands

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

**Create Patch**
```
cd sandbox
```
```
diff -ruN nginx-1.26.2/ nginx-1.26.2_changes/ > ../build/nginx-1.26.2.patch
```
