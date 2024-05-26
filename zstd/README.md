
nginx.conf
```nginx
http {
    ...

    zstd on;
    zstd_static on;
    zstd_comp_level 8;
    zstd_types application/atom+xml application/javascript application/json application/vnd.api+json application/rss+xml
             application/vnd.ms-fontobject application/x-font-opentype application/x-font-truetype
             application/x-font-ttf application/x-javascript application/xhtml+xml application/xml
             font/eot font/opentype font/otf font/truetype image/svg+xml image/vnd.microsoft.icon
             image/x-icon image/x-win-bitmap text/css text/javascript text/plain text/xml;
    #gzip  on;
    #brotli on;
    #brotli_static on;
    #brotli on;
    #brotli_comp_level 8;
    #brotli_static on;
    
    ....
}
```
look at: https://github.com/tokers/zstd-nginx-module?tab=readme-ov-file#directives