![Logo](Logo.png)

# nginx-shadow

## NGINX Custom Build with HTTP/3 and RTMP Support

This project contains a custom NGINX build with additional support for HTTP/3 (QUIC) and RTMP streaming protocol. Additionally, it includes a patch to remove server information from the HTTP headers for improved security.

## To-Do List

- [ ] **Prepare a build with the latest version of NGINX and the latest dependencies.**
- [ ] **Create a patch to remove NGINX server name and version information from the source code.**
- [ ] **Ensure support for HTTP/3 (QUIC).**
- [ ] **Ensure support for RTMP (Real-Time Messaging Protocol).**
- [ ] **Add support for STS (Stream TLS) and VTS (Virtual Traffic Status) modules.**
- [ ] **Add Brotli compression support, powered by ngx_brotli.**
- [ ] **Add GeoIP2 support, powered by ngx_http_geoip2_module.**
- [ ] **Add Headers More support, powered by ngx_headers_more.**
- [ ] **Remove unnecessary modules to improve performance.**

### Commands

```
docker build . -t nginx-shadow:0.0.1 --progress=plain 
```

```
docker run nginx-shadow:0.0.1    
```


