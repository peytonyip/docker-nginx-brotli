[![Open in Visual Studio Code](https://img.shields.io/static/v1?logo=visualstudiocode&label=&message=Open%20in%20Visual%20Studio%20Code&labelColor=2c2c32&color=007acc&logoColor=007acc)](https://open.vscode.dev/peytonyip/Dockerfile)

[![Build Docker Image CI](https://github.com/peytonyip/docker-nginx-brotli/actions/workflows/build-docker-image.yml/badge.svg)](https://github.com/peytonyip/docker-nginx-brotli/actions/workflows/build-docker-image.yml)     [![Build Stable Docker Image CI](https://github.com/peytonyip/docker-nginx-brotli/actions/workflows/build-stable-docker-image.yml/badge.svg)](https://github.com/peytonyip/docker-nginx-brotli/actions/workflows/build-stable-docker-image.yml)

[![Build ip2location Docker Image CI](https://github.com/peytonyip/docker-nginx-brotli/actions/workflows/build-ip2location-docker-image.yml/badge.svg)](https://github.com/peytonyip/docker-nginx-brotli/actions/workflows/build-ip2location-docker-image.yml)  [![Build ip2region Docker Image CI](https://github.com/peytonyip/docker-nginx-brotli/actions/workflows/build-ipregion-docker-image.yml/badge.svg)](https://github.com/peytonyip/docker-nginx-brotli/actions/workflows/build-ipregion-docker-image.yml)

---
# What is this?
This project is based on Alpine Linux, the official nginx image and an nginx module that provides static and dynamic brotli compression. [Brotli](https://github.com/google/brotli) and the [nginx brotli module ](https://github.com/google/ngx_brotli) are built by Google.

1.19.6 and before,is nginx + brotli

since 1.19.8 is nginx  + brotli + [ngx_http_geoip2_module](https://github.com/leev/ngx_http_geoip2_module) + [ngx_http_ipdb_module](https://github.com/vislee/ngx_http_ipdb_module)

**after 1.25.4, add http3, and ngx_otel_module module**

see doc in https://nginx.org/en/docs/quic.html and https://nginx.org/en/docs/ngx_otel_module.html

~~the http3 is test for latest nginx + brotli + ngx_http_geoip2_module + ngx_http_ipdb_module + http/3,http/3 support provided from the [cloudflare/quiche](https://github.com/cloudflare/quiche) projectthe, availability of the mirror is not guaranteed (Discontinued)~~

~~the quic is test for latest [nginx-quic](https://hg.nginx.org/nginx-quic/) + brotli + ngx_http_geoip2_module +  ngx_http_ipdb_module,the availability of the mirror is not guaranteed~~

the ip2location is  latest nginx + brotli + [ip2location-nginx](https://github.com/ip2location/ip2location-nginx) + ngx_http_ipdb_module

the ip2region is  latest nginx + brotli + [ngx_http_ip2region-module](https://github.com/liangwenrong/ngx_http_ip2region-module) + ngx_http_geoip2_module

lite is only nginx + brotli


# How to use this image
Just official Nginx,there are some configtion example,You can also see the relevant configuration examples from the corresponding module project address.

Simple configuration example(/etc/nginx/nginx.conf):

```nginx
user nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

# load dynamic-module,it depends on the tag you choose 
load_module modules/ngx_http_geoip2_module.so;
load_module modules/ngx_http_ipdb_module.so;
load_module modules/ngx_http_ip2region_module.so;


events {
  use epoll;
  worker_connections  1024;
}



http {
    ......
    log_format json_log escape=json '{'
       
        ........
        #Select the content of the part according to the actual selection

        '"ipdb_country_name": "$ipdb_country_name",'
        '"ipdb_city_name": "$ipdb_city_name",'
        '"ip2region_country_name": "$ip2region_country_name",'
        '"ip2region_city_name": "$ip2region_city_name",'
        '"ip2region_isp_domain": "$ip2region_isp_domain",'
        '"ip2region_province_name": "$ip2region_province_name",'
        '"ip2region_city_id": "$ip2region_city_id",'
        '"geoip2_data_country_code": "$geoip2_data_country_code",'
        '"geoip2_data_city_name": "$geoip2_data_city_name"'
        '}';
    
    # for geoip2
    geoip2 /path/to/GeoLite2-Country.mmdb {
        auto_reload 60m;
        $geoip2_metadata_country_build metadata build_epoch;
        $geoip2_data_country_code country iso_code;
        $geoip2_data_country_name country names en;
    }
    geoip2 /path/to/GeoLite2-City.mmdb {
        auto_reload 60m;
        $geoip2_metadata_city_build metadata build_epoch;
        $geoip2_data_city_name city names en;
    }
    
    # for ipdb
    ipdb /path/to/qqwry.ipdb;
    ipdb_language CN;
    ipdb_proxy 127.0.0.1;
    ipdb_proxy_recursive on;
    
    # for ip2region-v1
    ip2region "/path/to/ip2region.db" "btree"; 

    # for ip2region-v2
    # https://github.com/lionsoul2014/ip2region/tree/master/binding/nginx
    # set xdb file path
    ip2region_db ip2region.xdb;
    # ip2region_db ip2region.xdb vectorIndex;
    # ip2region_db ip2region.xdb file;
    # ip2region_db ip2region.xdb content;




    .........
    
    # brotli
    brotli on;
    brotli_comp_level 6;
    brotli_buffers 16 8k;
    brotli_min_length 20;
    brotli_static on;
    brotli_types application/atom+xml application/javascript application/json application/rss+xml application/vnd.ms-fontobject application/x-font-opentype application/x-font-truetype application/x-font-ttf application/x-javascript application/xhtml+xml application/xml font/eot font/opentype font/otf font/truetype image/svg+xml image/vnd.microsoft.icon image/x-icon image/x-win-bitmap text/css text/javascript text/plain text/xml;


    include conf.d/*.conf;
}

```

http3 example configuration:
```nginx
server {
  # Enable QUIC and HTTP/3.
  listen 443 quic reuseport;
  # Ensure that HTTP/2 is enabled for the server
  listen 443;
  server_name localhost;

  # Enable TLS versions (TLSv1.3 is required for QUIC).
  ssl_protocols TLSv1.2 TLSv1.3;

  ssl_certificate /etc/ssl/localhost.pem;
  ssl_certificate_key /etc/ssl/private/localhost.key;

  ssl_session_cache shared:SSL:1m;
  ssl_session_timeout 5m;

  # Enable TLSv1.3's 0-RTT. Use $ssl_early_data when reverse proxying to
  # prevent replay attacks.
  #
  # @see: http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_early_data
  ssl_early_data on;
  ssl_ciphers HIGH:!aNULL:!MD5;
  ssl_prefer_server_ciphers on;

  # Add Alt-Svc header to negotiate HTTP/3.
  add_header alt-svc 'h3=":443"; ma=86400';
  # Debug 0-RTT.
  add_header X-Early-Data $tls1_3_early_data;

  add_header x-frame-options "deny";
  add_header Strict-Transport-Security "max-age=31536000" always;

  location / {
    root /usr/share/nginx/html;
    index index.html index.htm;
  }
}

map $ssl_early_data $tls1_3_early_data {
  "~." $ssl_early_data;
  default "";
}

```

quic example configuration:
```nginx
http {
        log_format quic '$remote_addr - $remote_user [$time_local] '
                        '"$request" $status $body_bytes_sent '
                        '"$http_referer" "$http_user_agent" "$quic" "$http3"';

        access_log logs/access.log quic;

        server {
            # for better compatibility it's recommended
            # to use the same port for quic and https
            listen 8443 quic reuseport;
            listen 8443;

            ssl_certificate     certs/example.com.crt;
            ssl_certificate_key certs/example.com.key;
            ssl_protocols       TLSv1.3;

            location / {
                # required for browsers to direct them into quic port
                add_header Alt-Svc 'h3=":8443"; ma=86400';
            }
        }
    }

```

### HEALTHCHECK
the image had add healthcheck:
```
HEALTHCHECK --interval=10s --timeout=5s --start-period=60s \
  CMD nc -z localhost 80 || exit 1
```
the default port is 80, so if you are using a different port, you need to change it:

```yaml
#docker-compose.yaml:

version: "3.8"
services:
  nginx:
    image: peytonyip/nginx-brotli:<tag>
    healthcheck:
    test: ["CMD", "nc", "-z", "localhost", "<replace it with the port you used.>"]
    interval: 10s
    timeout: 5s
    start_period: 60s
    ......
```

free ip database download links:

[GeoLite2](https://dev.maxmind.com/geoip/geoip2/geolite2/)

[qqwry.ipdb](https://github.com/metowolf/qqwry.ipdb)

[ip2location](https://lite.ip2location.com/)

[ip2region.xdb](https://github.com/lionsoul2014/ip2region/tree/master/data)

# Thanks for

[Support for QUIC and HTTP/3](https://nginx.org/en/docs/quic.html)

[fholzer/docker-nginx-brotli](https://github.com/fholzer/docker-nginx-brotli)

[RanadeepPolavarapu/docker-nginx-http3](https://github.com/RanadeepPolavarapu/docker-nginx-http3)

[marinelli/quiche](https://github.com/marinelli/quiche/tree/quiche-nginx-1.19.7)

[vislee/ngx_http_ipdb_module](https://github.com/vislee/ngx_http_ipdb_module.git)

[quiche](https://github.com/cloudflare/quiche)
