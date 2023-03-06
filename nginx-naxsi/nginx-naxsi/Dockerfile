ARG NGINX_VERSION=1.22.1
ARG NAXSI_VERSION=1.3

FROM nginx:$NGINX_VERSION as build

ARG NGINX_VERSION
ARG NAXSI_VERSION

RUN apt update && apt install -y \
        wget \
        libpcre3-dev \
        gcc \
        make \
        gzip \
        zlib1g \
        zlib1g-dev \
        libssl-dev

RUN wget https://github.com/nbs-system/naxsi/archive/$NAXSI_VERSION.tar.gz -O naxsi_$NAXSI_VERSION.tar.gz && \
    tar vxf naxsi_$NAXSI_VERSION.tar.gz

RUN wget https://nginx.org/download/nginx-$NGINX_VERSION.tar.gz && \
    tar vxf nginx-$NGINX_VERSION.tar.gz && \
    cd nginx-$NGINX_VERSION && \
    ./configure --add-dynamic-module=../naxsi-$NAXSI_VERSION/naxsi_src/ \
    --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-g -O2 -ffile-prefix-map=/data/builder/debuild/nginx-$NGINX_VERSION/debian/debuild-base/nginx-$NGINX_VERSION=. -fstack-protector-strong -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -fPIC' --with-ld-opt='-lpcre -Wl,-z,relro -Wl,-z,now -Wl,--as-needed -pie' && \
    make modules

FROM nginx:$NGINX_VERSION

ARG NGINX_VERSION
ARG NAXSI_VERSION

COPY --from=build /nginx-$NGINX_VERSION/objs/ngx_http_naxsi_module.so /etc/nginx/modules/
COPY --from=build /naxsi-$NAXSI_VERSION/naxsi_config/naxsi_core.rules /etc/nginx/

RUN sed -i "1 i\load_module /etc/nginx/modules/ngx_http_naxsi_module.so;" /etc/nginx/nginx.conf
RUN sed -i "/^http {/a include /etc/nginx/naxsi_core.rules;" /etc/nginx/nginx.conf
