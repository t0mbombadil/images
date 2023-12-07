ARG BASE_IMAGE=gcr.io/distroless/static-debian12
# "x86_64" / "aarch64" - must match the one in the name of /usr/lib/*-linux-gnu directory
ARG ARCH="x86_64"

FROM nginx:1.25.3-bookworm as builder
FROM ${BASE_IMAGE}:latest as prod

ARG ARCH

COPY --from=builder /etc/nginx /etc/nginx
# To copy nginx user, which is specified in /etc/nginx/nginx.conf
# Master process needs to run with root (to access ports 1-1024), but child processes will
# work with nginx user
COPY --from=builder  /etc/passwd /etc/shadow /etc/group /etc/
COPY --from=builder /etc/init.d/nginx /etc/init.d/nginx
#COPY --from=builder /usr/share/nginx/html/ /usr/share/nginx/html/
COPY --from=builder /var/log/nginx /var/log/nginx
COPY --from=builder  /var/run /var/run

COPY --from=builder /usr/share/nginx /usr/share/nginx
COPY --from=builder /usr/sbin/nginx /usr/sbin/nginx
COPY --from=builder /var/cache/nginx/ /var/cache/nginx/

# Optional nginx modules. Uncomment if used
# COPY --from=builder /usr/lib/nginx/modules /usr/lib/nginx/modules

# Shared libraries specific to architecture like /lib/x86_64-linux-gnu/
#
# We will be able to make it platform agnostic once this experimental feature will be added
# to docker engine - https://docs.docker.com/engine/reference/builder/#copy---parents
# Until then it requires ${ARCH} build arg
COPY --from=builder /lib/*-linux-gnu/libdl.so.* /lib/${ARCH}-linux-gnu/
COPY --from=builder /lib/*-linux-gnu/libpthread.so.* /lib/${ARCH}-linux-gnu/
COPY --from=builder /lib/*-linux-gnu/libcrypt.so.* /lib/${ARCH}-linux-gnu/
COPY --from=builder /lib/*-linux-gnu/libpcre*.so.* /lib/${ARCH}-linux-gnu/
COPY --from=builder /lib/*-linux-gnu/libz.so.* /lib/${ARCH}-linux-gnu/
COPY --from=builder /lib/*-linux-gnu/libc.so.* /lib/${ARCH}-linux-gnu/
COPY --from=builder /lib/*-linux-gnu/ld-linux* /lib/${ARCH}-linux-gnu/

COPY --from=builder /usr/lib/*-linux-gnu/libssl.so.* /usr/lib/${ARCH}-linux-gnu/
COPY --from=builder /usr/lib/*-linux-gnu/libcrypto.so.* /usr/lib/${ARCH}-linux-gnu/

# This is important for nginx as it is usinqg libnss to query user information from `/etc/passwd`
# otherwise you will receive error like `nginx: [emerg] getpwnam("nonroot") failed`.
COPY --from=builder /lib/*-linux-gnu/libnss_compat.so.* /lib/${ARCH}-linux-gnu/
COPY --from=builder /lib/*-linux-gnu/libnss_files.so.* /lib/${ARCH}-linux-gnu/

# copy /lib64/ld-linux-x86-64.so.* (only present on x86)
COPY --from=builder /lib64* /lib64
COPY --from=builder /lib/ld-linux-* /lib



FROM debian:stable-slim as debug-strace
RUN apt update -y && apt install strace -y
# Build image with #> docker buildx build ... --target debug
# to execute this debugging step instead of production one
# This additional step helps to debug easier distroless images
FROM  ${BASE_IMAGE}:debug as debug

COPY --from=prod / /
COPY --from=builder /usr/sbin/nginx-debug /usr/sbin/nginx-debug
COPY --from=debug-strace /usr/bin/strace /usr/bin/strace

EXPOSE 80
STOPSIGNAL SIGQUIT
# Single quotes here are required for debug image sh to understand this as single expression
# strace can be used to debug e.g. missing binary dependencies (*.so files)
CMD ["-c", "strace nginx -g 'daemon off;'"]



FROM prod

# Set the container timezone as UTC.
ENV TZ="UTC"

EXPOSE 80
STOPSIGNAL SIGQUIT

ENTRYPOINT [ "/usr/sbin/nginx" ]
CMD ["-g", "daemon off;"]
