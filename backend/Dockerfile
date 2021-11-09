FROM nginx:alpine AS builder

RUN wget "http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" && \
    mkdir /usr/src && \
    tar -C /usr/src -xzvf nginx-${NGINX_VERSION}.tar.gz

# For latest build deps, see https://github.com/nginxinc/docker-nginx/blob/master/mainline/alpine/Dockerfile
RUN apk add --no-cache --virtual .build-deps \
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
  unzip

RUN wget https://github.com/openresty/echo-nginx-module/archive/v0.61.zip && \
    unzip v0.61.zip -d /tmp

# Reuse same cli arguments as the nginx:alpine image used to build
RUN CONFARGS=$(nginx -V 2>&1 | sed -n -e 's/^.*arguments: //p') \
    cd /usr/src/nginx-${NGINX_VERSION} && \
    ./configure --with-compat $CONFARGS --prefix=/opt/nginx \
        --add-dynamic-module=/tmp/echo-nginx-module-0.61 && \
    make -j2 && \
    make install

FROM nginx:alpine
# Extract the ECHO module from the builder image
COPY --from=builder /opt/nginx/modules/ngx_http_echo_module.so /usr/lib/nginx/modules/ngx_http_echo_module.so

EXPOSE 80
STOPSIGNAL SIGTERM
CMD ["nginx", "-g", "daemon off;"]
