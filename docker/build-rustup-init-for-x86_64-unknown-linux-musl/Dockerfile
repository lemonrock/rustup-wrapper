# This file is part of rustup-wrapper. It is subject to the license terms in the COPYRIGHT file found in the top-level directory of this distribution and at https://raw.githubusercontent.com/lemonrock/rustup-wrapper/master/COPYRIGHT. No part of rustup-wrapper, including this file, may be copied, modified, propagated, or distributed except according to the terms contained in the COPYRIGHT file.
# Copyright © 2016 The developers of rustup-wrapper. See the COPYRIGHT file in the top-level directory of this distribution and at https://raw.githubusercontent.com/lemonrock/rustup-wrapper/master/COPYRIGHT.


FROM alpine:3.4
MAINTAINER https://github.com/lemonrock/rustup-wrapper

# Uses releases made by https://github.com/sgerrand/alpine-pkg-glibc
# /etc/apk/keys/sgerrand.rsa.pub is from https://raw.githubusercontent.com/sgerrand/alpine-pkg-glibc/master/sgerrand.rsa.pub
# /tmp/glibc.apk is from https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.23-r3/glibc-2.23-r3.apk

ENV PATH=/usr/sbin:/usr/bin:/sbin:/bin
COPY etc /etc/
COPY tmp /tmp/
COPY home/rust /home/rust/
RUN \
rm -rf /usr/local /opt \
&& ln -s /home/rust/sysroot /usr/local \
&& apk update \
&& apk add /tmp/glibc.apk \
&& apk add /tmp/glibc-bin.apk \
&& apk add /tmp/glibc-i18n.apk \
&& /usr/glibc-compat/bin/localedef -i en_US -f UTF-8 en_US.UTF-8 \
&& rm /etc/apk/keys/sgerrand.rsa.pub /tmp/*.apk \
&& apk upgrade \
&& apk --no-cache add ca-certificates zlib libgcc sudo gcc make zlib-dev openssl-dev musl-dev curl-dev \
&& adduser -D -h /home/rust -s /bin/sh rust \
&& chmod 0755 /tmp/rustup-init \
&& find /home/rust -type f ! -perm +0111 | xargs chmod 0640 \
&& find /home/rust -type f -perm +0111 | xargs chmod 0750 \
&& find /home/rust -type d | xargs chmod 0750 \
&& chown -R rust:rust /home/rust \
&& /usr/glibc-compat/sbin/ldconfig

RUN \
cd /home/rust/packages/musl \
&& CFLAGS='-O3 -pipe -g0' ./configure --prefix=/usr/local --disable-shared --enable-optimize --enable-wrapper=all \
&& make \
&& make install
#ENV CC musl-gcc
# rustc-cfg and rustc-flags can also be overridden for a build script

# RUSTUP_HOME=~/.rustup.tmp
USER rust
WORKDIR /home/rust
ENV PATH /usr/glibc-compat/bin:/usr/local/bin:/usr/bin:/bin
#ENV LD_LIBRARY_PATH /usr/glibc-compat/lib:/usr/lib:/lib
RUN /tmp/rustup-init -y --no-modify-path --default-toolchain stable
ENV PATH /home/rust/.cargo/bin:/usr/local/bin:/usr/bin:/bin
RUN rustup self update
RUN rustup target add x86_64-unknown-linux-musl
RUN rustup update
RUN rustup show
RUN \
cd /home/rust/packages/rustup.rs \
&& cargo fetch

RUN ls -la /usr/glibc-compat/lib

RUN \
cd /home/rust/packages/rustup.rs \
&& mkdir -p /home/rust/packages/rustup.rs/target/release/build/rustup-48ebcb395e1e5302 /home/rust/packages/rustup.rs/target/release/deps \
&& rustc build.rs --crate-name build_script_build --crate-type bin -C opt-level=3 \
--cfg 'feature="curl-backend"' --cfg 'feature="download"' --cfg 'feature="default"' \
--cfg 'feature="hyper-backend"' -C metadata=48ebcb395e1e5302 \
--out-dir /home/rust/packages/rustup.rs/target/release/build/rustup-48ebcb395e1e5302 \
--emit=dep-info,link -L dependency=/home/rust/packages/rustup.rs/target/release/deps \
-L native=/usr/glibc-compat/lib

RUN \
exit 1

RUN \
cd /home/rust/packages/rustup.rs \
&& cargo build --verbose --jobs 1 --release --target=x86_64-unknown-linux-musl
