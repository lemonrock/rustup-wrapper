# This file is part of rustup-wrapper. It is subject to the license terms in the COPYRIGHT file found in the top-level directory of this distribution and at https://raw.githubusercontent.com/lemonrock/rustup-wrapper/master/COPYRIGHT. No part of rustup-wrapper, including this file, may be copied, modified, propagated, or distributed except according to the terms contained in the COPYRIGHT file.
# Copyright © 2016 The developers of rustup-wrapper. See the COPYRIGHT file in the top-level directory of this distribution and at https://raw.githubusercontent.com/lemonrock/rustup-wrapper/master/COPYRIGHT.


MAINTAINER https://github.com/lemonrock/rustup-wrapper
FROM ubuntu:16.04

RUN \
apt-get update \
&& \
apt-get install --yes build-essential cmake curl file git sudo xutils-dev \
&& \
apt-get clean \
&& \
rm -rf /var/lib/apt/lists/*
&& \
useradd rust --user-group --create-home --shell /bin/sh

COPY etc /etc/
COPY usr /usr/

USER rust
RUN rustup-init --default-host x86_64-unknown-linux-musl --default-toolchain stable
COPY home/rust /home/rust/
WORKDIR /home/rust/packages
ENV PATH=/home/rust/sysroot/bin:/home/rust/.cargo/bin:/usr/local/bin:/usr/bin:/bin

RUN \
cd musl \
&& \
./configure --prefix=/home/rust/sysroot \
&& \
make \
&& \
make install
