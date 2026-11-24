FROM --platform=$BUILDPLATFORM alpine:3.22 AS temp
ARG TARGETARCH

RUN [ "${TARGETARCH}" = "arm64" -o "${TARGETARCH}" = "amd64" ] && echo ok || exit 1

ENV LANG=C.UTF-8 LC_ALL=C.UTF-8

RUN apk add --no-cache ca-certificates tzdata \
  && mkdir -p /rootfs/etc/ssl/certs \
  && cp -a /etc/ssl/certs/ca-certificates.crt /rootfs/etc/ssl/certs/ \
  && cp -a /usr/share/zoneinfo/Asia/Shanghai /rootfs/etc/localtime \
  && echo "Asia/Shanghai" >/rootfs/etc/timezone

COPY bin/xlp-${TARGETARCH} /rootfs/bin/xlp

# 迅雷主程序是 glibc 二进制，Alpine(musl) 无法直接运行，
# 从 ubuntu:26.04 提取最新 glibc / libstdc++ 运行库作为兼容层，放入 /lib 下，
# 这样 chroot 时绑定 /lib、/lib64 即可正常加载。
FROM ubuntu:26.04 AS glibc
ARG TARGETARCH

RUN set -eux; \
  case "${TARGETARCH}" in \
  amd64) ARCH=x86_64-linux-gnu; LOADER=ld-linux-x86-64.so.2 ;; \
  arm64) ARCH=aarch64-linux-gnu; LOADER=ld-linux-aarch64.so.1 ;; \
  *) echo "unsupported arch: ${TARGETARCH}" >&2; exit 1 ;; \
  esac; \
  mkdir -p "/glibc/lib/${ARCH}" /glibc/lib64; \
  for lib in \
  "${LOADER}" \
  libc.so.6 \
  libm.so.6 \
  libdl.so.2 \
  libpthread.so.0 \
  librt.so.1 \
  libresolv.so.2 \
  libutil.so.1 \
  libcrypt.so.1 \
  libanl.so.1 \
  libstdc++.so.6 \
  libgcc_s.so.1 \
  libnss_dns.so.2 \
  libnss_files.so.2 \
  libnss_compat.so.2; do \
  cp -L "/lib/${ARCH}/${lib}" "/glibc/lib/${ARCH}/${lib}"; \
  done; \
  if [ "${TARGETARCH}" = "arm64" ]; then \
  ln -sf "../lib/${ARCH}/${LOADER}" "/glibc/lib/${LOADER}"; \
  else \
  ln -sf "../lib/${ARCH}/${LOADER}" "/glibc/lib64/${LOADER}"; \
  fi

FROM alpine:3.22

LABEL org.opencontainers.image.authors=cnk3x
LABEL org.opencontainers.image.source=https://github.com/cnk3x/xunlei

COPY --from=glibc /glibc/lib/ /lib/
COPY --from=glibc /glibc/lib64/ /lib64/
COPY --from=temp /rootfs/ /

ENV LANG=C.UTF-8 LC_ALL=C.UTF-8 \
  XL_DEBUG=false \
  XL_DASHBOARD_PORT=2345 \
  XL_DASHBOARD_USERNAME= \
  XL_DIR_DOWNLOAD=/xunlei/downloads \
  XL_DIR_DATA=/xunlei/data \
  XL_DASHBOARD_PASSWORD= \
  XL_CHROOT=/xunlei \
  XL_PREVENT_UPDATE=true \
  XL_UID= \
  XL_GID=

CMD [ "/bin/xlp" ]
