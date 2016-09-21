# This file is part of rustup-wrapper. It is subject to the license terms in the COPYRIGHT file found in the top-level directory of this distribution and at https://raw.githubusercontent.com/lemonrock/rustup-wrapper/master/COPYRIGHT. No part of rustup-wrapper, including this file, may be copied, modified, propagated, or distributed except according to the terms contained in the COPYRIGHT file.
# Copyright © 2016 The developers of rustup-wrapper. See the COPYRIGHT file in the top-level directory of this distribution and at https://raw.githubusercontent.com/lemonrock/rustup-wrapper/master/COPYRIGHT.


FROM alpine:3.4
MAINTAINER https://github.com/lemonrock/rustup-wrapper

ENV PATH=/usr/sbin:/usr/bin:/sbin:/bin
RUN \
&& apk update \
&& apk upgrade \
&& apk --no-cache add ca-certificates zlib libgcc sudo gcc make cmake zlib-dev openssl-dev musl-dev curl-dev curl python

COPY home/rust /home/rust/
RUN \
rm -rf /usr/local /opt \
&& ln -s /home/rust/sysroot /usr/local \
&& adduser -D -h /home/rust -s /bin/sh rust \
&& find /home/rust -type f ! -perm +0111 | xargs chmod 0640 \
&& find /home/rust -type f -perm +0111 | xargs chmod 0750 \
&& find /home/rust -type d | xargs chmod 0750 \
&& chown -R rust:rust /home/rust
USER rust
WORKDIR /home/rust
ENV PATH=/home/rust/.local/bin:/usr/bin:/bin

RUN \
rustc -Vv

# ENV PREFIX=/home/rust/sysroot
#
# RUN \
# cd packages/musl \
# && CFLAGS='-O3 -pipe -g0' ./configure --prefix="$PREFIX" --disable-shared --enable-optimize --enable-wrapper=all \
# && make \
# && make install
#
# ENV CC "$PREFIX"/bin/musl-gcc
#
# RUN \
# mv packages/libunwind packages/llvm/projects &&
# cd packages/llvm/projects/libunwind \
# && mkdir -m 0755 -p build \
# && cd build \
# && cmake -DLLVM_PATH=../../.. -DLIBUNWIND_ENABLE_SHARED=0 .. \
# && make \
# cp lib/libunwind.a "$PREFIX"/lib/
#
# RUN \
# cd packages/rust \
# && ./configure --target=x86_64-unknown-linux-musl --musl-root="$PREFIX" --prefix="$PREFIX" --llvm-root=/home/rust/packages \
# && make \
# && make install
#
# RUN \
# echo 'fn main() { println!("Hello World"); panic!("failed"); }' >example.rs \
# && "$PREFIX"/rustc --target=x86_64-unknown-linux-musl example.rs \
# && ./example
# # Should fail with a nasty error...
#
# RUN \
# cd packages/cargo \
# && ./configure --prefix="$PREFIX" --localstatedir="$PREFIX"/var/lib --sysconfdir="$PREFIX"/etc --local-rust-root="$PREFIX" \
# && make \
# && make install
