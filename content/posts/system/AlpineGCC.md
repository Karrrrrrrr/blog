---
title: Alpine-GCC13
categories: ["系统" , "Linux"]
tags: ["系统" , "Linux"]
date: "2023-09-28T18:21:11+08:00"

---

## Alpine-GCC13
```Dockerfile
ARG ALPINE_VERSION=latest

FROM alpine:${ALPINE_VERSION} as alpine

ARG GCC_VERSION
ENV GCC_VERSION=${GCC_VERSION}


FROM alpine as builder

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories  \
    && apk update  \
    && apk add --no-cache \
            build-base \
            dejagnu \
            isl-dev \
            make \
            mpc1-dev \
            mpfr-dev \
            texinfo \
            zlib-dev
RUN wget -q https://ftp.gnu.org/gnu/gcc/gcc-${GCC_VERSION}/gcc-${GCC_VERSION}.tar.gz && \
    tar -xzf gcc-${GCC_VERSION}.tar.gz && \
    rm -f gcc-${GCC_VERSION}.tar.gz

WORKDIR /gcc-${GCC_VERSION}

RUN ./configure \
        --prefix=/usr/local \
        --build=$(uname -m)-alpine-linux-musl \
        --host=$(uname -m)-alpine-linux-musl \
        --target=$(uname -m)-alpine-linux-musl \
        --with-pkgversion="Alpine ${GCC_VERSION}" \
        --enable-checking=release \
        --disable-fixed-point \
        --disable-libmpx \
        --disable-libmudflap \
        --disable-libsanitizer \
        --disable-libssp \
        --disable-libstdcxx-pch \
        --disable-multilib \
        --disable-nls \
        --disable-symvers \
        --disable-werror \
        --enable-__cxa_atexit \
        --enable-default-pie \
        --enable-languages=c,c++ \
        --enable-shared \
        --enable-threads \
        --enable-tls \
        --with-linker-hash-style=gnu \
        --with-system-zlib
RUN make -j $(nproc)
RUN make -j $(nproc) install-strip

RUN ln -s /usr/bin/gcc /usr/local/bin/cc

RUN gcc -v


FROM alpine

RUN apk add --no-cache \
            autoconf \
            automake \
            binutils \
            cmake \
            file \
            git \
            gmp \
            isl \
            libc-dev \
            libtool \
            make \
            mpc1 \
            mpfr4 \
            musl-dev \
            pkgconf \
            zlib-dev

COPY --from=builder /usr/local/ /usr/

WORKDIR /src
```
```shell
docker build --build-arg GCC_VERSION=13.1.0 -t alpine-gcc .
```
