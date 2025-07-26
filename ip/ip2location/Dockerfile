ARG NGINX_VERSION=1.29.0
ARG NGX_BROTLI_COMMIT=a71f9312c2deb28875acc7bacfdd5695a111aa53
ARG CONFIG="\
	--prefix=/etc/nginx \
	--sbin-path=/usr/sbin/nginx \
	--modules-path=/usr/lib/nginx/modules \
	--conf-path=/etc/nginx/nginx.conf \
	--error-log-path=/var/log/nginx/error.log \
	--http-log-path=/var/log/nginx/access.log \
	--pid-path=/var/run/nginx.pid \
	--lock-path=/var/run/nginx.lock \
	--http-client-body-temp-path=/var/cache/nginx/client_temp \
	--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
	--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
	--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
	--http-scgi-temp-path=/var/cache/nginx/scgi_temp \
	--user=nginx \
	--group=nginx \
	--with-http_ssl_module \
	--with-http_realip_module \
	--with-http_addition_module \
	--with-http_sub_module \
	--with-http_dav_module \
	--with-http_flv_module \
	--with-http_mp4_module \
	--with-http_gunzip_module \
	--with-http_gzip_static_module \
	--with-http_random_index_module \
	--with-http_secure_link_module \
	--with-http_stub_status_module \
	--with-http_auth_request_module \
	--with-http_xslt_module=dynamic \
	--with-http_image_filter_module=dynamic \
	--with-http_geoip_module=dynamic \
	--with-threads \
	--with-stream \
	--with-stream_ssl_module \
	--with-stream_ssl_preread_module \
	--with-stream_realip_module \
	--with-stream_geoip_module=dynamic \
	--with-http_slice_module \
	--with-mail \
	--with-mail_ssl_module \
	--with-compat \
	--with-file-aio \
	--with-http_v2_module \
	--add-module=/usr/src/ngx_brotli \
	--add-dynamic-module=/usr/src/ip2location-nginx \
	"

FROM alpine:3.22.0
LABEL maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>"

ARG NGINX_VERSION
ARG NGX_BROTLI_COMMIT
ARG CONFIG

ENV OTEL_VERSION   0.1.1
ENV PKG_RELEASE 1

RUN \
	apk add --no-cache --virtual .build-deps \
	gcc \
	libc-dev \
	make \
	openssl-dev \
	pcre-dev \
	zlib-dev \
	linux-headers \
	curl \
	gnupg \
	libxslt-dev \
	gd-dev \
	geoip-dev \
	perl \
	autoconf \
	libtool \
	automake \
	git \
	g++ \
	cmake

RUN set -x \
    && apkArch="$(cat /etc/apk/arch)" \
    && nginxPackages=" \
        nginx-module-otel=${NGINX_VERSION}.${OTEL_VERSION}-r${PKG_RELEASE} \
    " \
# install prerequisites for public key and pkg-oss checks
    && apk add --no-cache --virtual .checksum-deps \
        openssl \
	&& KEY_SHA512="e09fa32f0a0eab2b879ccbbc4d0e4fb9751486eedda75e35fac65802cc9faa266425edf83e261137a2f4d16281ce2c1a5f4502930fe75154723da014214f0655" \
	&& wget -O /tmp/nginx_signing.rsa.pub https://nginx.org/keys/nginx_signing.rsa.pub \
	&& if echo "$KEY_SHA512 */tmp/nginx_signing.rsa.pub" | sha512sum -c -; then \
		echo "key verification succeeded!"; \
		mv /tmp/nginx_signing.rsa.pub /etc/apk/keys/; \
	else \
		echo "key verification failed!"; \
		exit 1; \
	fi \
	&& apk add -X "https://nginx.org/packages/mainline/alpine/v$(egrep -o '^[0-9]+\.[0-9]+' /etc/alpine-release)/main" --no-cache $nginxPackages

COPY nginx.pub /tmp/nginx.pub

RUN \
	mkdir -p /usr/src/ngx_brotli \
	&& cd /usr/src/ngx_brotli \
	&& git init \
	&& git remote add origin https://github.com/google/ngx_brotli.git \
	&& git fetch --depth 1 origin $NGX_BROTLI_COMMIT \
	&& git checkout --recurse-submodules -q FETCH_HEAD \
	&& git submodule update --init --depth 1 \
	&& cd .. \
	&& curl -fSL https://nginx.org/download/nginx-$NGINX_VERSION.tar.gz -o nginx.tar.gz \
	&& curl -fSL https://nginx.org/download/nginx-$NGINX_VERSION.tar.gz.asc  -o nginx.tar.gz.asc \
	&& git clone https://github.com/chrislim2888/IP2Location-C-Library.git /IP2Location-C-Library \
	&& cd /IP2Location-C-Library \
	&& autoreconf -i -v --force \
	&& ./configure \
	&& make \
	&& make install \
	&& cd /usr/src \
	&& git clone https://github.com/ip2location/ip2location-nginx.git /usr/src/ip2location-nginx \
	&& tar -zxC /usr/src -f nginx.tar.gz

RUN \
	cd /usr/src/nginx-$NGINX_VERSION \
	&& ./configure $CONFIG \
	&& make -j$(getconf _NPROCESSORS_ONLN)

RUN \
	cd /usr/src/nginx-$NGINX_VERSION \
	&& make install \
	&& rm -rf /etc/nginx/html/ \
	&& mkdir -p /usr/share/nginx/html/ \
	&& install -m644 html/index.html /usr/share/nginx/html/ \
	&& install -m644 html/50x.html /usr/share/nginx/html/ \
	&& strip /usr/sbin/nginx* \
	&& strip /usr/lib/nginx/modules/*.so \
	\
	&& apk add --no-cache --virtual .gettext gettext \
	\
	&& scanelf --needed --nobanner /usr/sbin/nginx /usr/lib/nginx/modules/*.so /usr/bin/envsubst \
	| awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
	| sort -u \
	| xargs -r apk info --installed \
	| sort -u > /tmp/runDeps.txt

FROM alpine:3.22.0
ARG NGINX_VERSION

COPY --from=0 /tmp/runDeps.txt /tmp/runDeps.txt
COPY --from=0 /etc/nginx /etc/nginx
COPY --from=0 /usr/lib/nginx/modules/*.so /usr/lib/nginx/modules/
COPY --from=0 /usr/sbin/nginx  /usr/sbin/
COPY --from=0 /usr/share/nginx/html/* /usr/share/nginx/html/
COPY --from=0 /usr/bin/envsubst /usr/local/bin/envsubst

RUN \
	addgroup -S nginx \
	&& adduser -D -S -h /var/cache/nginx -s /sbin/nologin -G nginx nginx \
	&& apk add --no-cache --virtual .build-deps \
	autoconf \
	libtool \
	automake \
	git \
	g++ \
	cmake \
	gcc \
	libc-dev \
	make \
	openssl3-dev \
	pcre-dev \
	zlib-dev \
	linux-headers \
	curl \
	gnupg \
	libxslt-dev \
	gd-dev \
	geoip-dev \
	perl \
	&& git clone https://github.com/chrislim2888/IP2Location-C-Library.git /IP2Location-C-Library \
	&& cd /IP2Location-C-Library \
	&& autoreconf -i -v --force \
	&& ./configure \
	&& make \
	&& make install \
	&& apk del .build-deps \
	&& rm -rf /IP2Location-C-Library \
	&& apk add --no-cache --virtual .nginx-rundeps openssl tzdata $(cat /tmp/runDeps.txt) \
	&& rm /tmp/runDeps.txt \
	&& ln -s /usr/lib/nginx/modules /etc/nginx/modules \
	# forward request and error logs to docker log collector
	&& mkdir /var/log/nginx \
	#&& touch /var/log/nginx/access.log /var/log/nginx/error.log \
	&& ln -sf /dev/stdout /var/log/nginx/access.log \
	&& ln -sf /dev/stderr /var/log/nginx/error.log

COPY nginx.conf /etc/nginx/nginx.conf
COPY nginx.vh.default.conf /etc/nginx/conf.d/default.conf

EXPOSE 80 443

STOPSIGNAL SIGQUIT

HEALTHCHECK --interval=10s --timeout=5s --start-period=60s \
  CMD nc -z localhost 80 || exit 1

CMD ["nginx", "-g", "daemon off;"]
