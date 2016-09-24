# This file is part of rustup-wrapper. It is subject to the license terms in the COPYRIGHT file found in the top-level directory of this distribution and at https://raw.githubusercontent.com/lemonrock/rustup-wrapper/master/COPYRIGHT. No part of rustup-wrapper, including this file, may be copied, modified, propagated, or distributed except according to the terms contained in the COPYRIGHT file.
# Copyright © 2016 The developers of rustup-wrapper. See the COPYRIGHT file in the top-level directory of this distribution and at https://raw.githubusercontent.com/lemonrock/rustup-wrapper/master/COPYRIGHT.


FROM alpine:3.4
MAINTAINER https://github.com/lemonrock/rustup-wrapper

ENV PATH=/usr/sbin:/usr/bin:/sbin:/bin
WORKDIR /


ARG glibcCompatibilityBaseUrl=https://github.com/sgerrand/alpine-pkg-glibc/releases/download
ARG glibcCompatibilityVersion=2.23-r3
ARG channel=nightly
ARG target=x86_64-unknown-linux-musl
ARG PREFIX=/usr/local
ARG officialBaseUrl=https://static.rust-lang.org/rustup/dist
ARG fromTarget=x86_64-unknown-linux-gnu
ARG muslGitVersion=v1.1.15
ARG muslCrossMakeVersion=99ee12d604e564fa5b403c942e2809881d34d114
ARG rustGitVersion=5cc6c6b1b76960441c39ef29e951f22de45da15b
# LLVM 3.7
ARG libunwindGitVersion=release_37
ARG cargoGitVersion=939922974bc18df2daac055fd1bcb2c2d87d629d



RUN \
printf '%s\n%s\n%s\n' 'http://dl-cdn.alpinelinux.org/alpine/v3.4/main' 'http://dl-cdn.alpinelinux.org/alpine/v3.4/community' 'http://dl-cdn.alpinelinux.org/alpine/edge/main' 'http://dl-cdn.alpinelinux.org/alpine/edge/community' 'http://dl-cdn.alpinelinux.org/alpine/edge/testing' >/etc/apk/repositories \
&& apk update \
&& apk upgrade \
&& apk add zlib libgcc sudo binutils gcc g++ git make cmake zlib-dev openssl-dev musl-dev curl-dev curl wget python file patchelf

# Install unofficial glibc packages
RUN \
cd tmp \
&& curl --location --output glibc-"$glibcCompatibilityVersion".apk "$glibcCompatibilityBaseUrl"/"$glibcCompatibilityVersion"/glibc-"$glibcCompatibilityVersion".apk \
&& curl --location --output glibc-bin-"$glibcCompatibilityVersion".apk "$glibcCompatibilityBaseUrl"/"$glibcCompatibilityVersion"/glibc-bin-"$glibcCompatibilityVersion".apk \
&& curl --location --output glibc-i18n-"$glibcCompatibilityVersion".apk "$glibcCompatibilityBaseUrl"/"$glibcCompatibilityVersion"/glibc-i18n-"$glibcCompatibilityVersion".apk \
&& curl --location --output sgerrand.rsa.pub "$glibcCompatibilityBaseUrl"/"$glibcCompatibilityVersion"/sgerrand.rsa.pub
RUN \
apk --no-cache add ca-certificates \
&& cd tmp \
&& mv sgerrand.rsa.pub /etc/apk/keys \
&& apk add glibc-"$glibcCompatibilityVersion".apk \
&& rm -rf glibc-"$glibcCompatibilityVersion".apk \
&& apk add glibc-bin-"$glibcCompatibilityVersion".apk \
&& rm -rf glibc-bin-"$glibcCompatibilityVersion".apk \
&& apk add glibc-i18n-"$glibcCompatibilityVersion".apk \
&& rm -rf glibc-i18n-"$glibcCompatibilityVersion".apk \
&& rm -rf /etc/apk/keys/sgerrand.rsa.pub \
&& /usr/glibc-compat/bin/localedef -i en_US -f UTF-8 en_US.UTF-8 \
&& /usr/glibc-compat/sbin/ldconfig
ENV PATH=/usr/glibc-compat/sbin:/usr/glibc-compat/bin:/usr/sbin:/usr/bin:/sbin:/bin

COPY home/rust /home/rust/
RUN \
rm -rf /usr/local /opt \
&& ln -s /home/rust/sysroot /usr/local \
&& adduser -D -h /home/rust -s /bin/sh rust \
&& find /home/rust -type f ! -perm +0111 -print0 | xargs -0 -r chmod 0640 \
&& find /home/rust -type f -perm +0111 -print0 | xargs -0 -r chmod 0750 \
&& find /home/rust -type d -print0 | xargs -0 -r chmod 0750 \
&& chown -R rust:rust /home/rust
USER rust
WORKDIR /home/rust
ENV PATH=/usr/bin:/bin


# https://static.rust-lang.org/dist/channel-rust-stable.toml.sha256  and  https://static.rust-lang.org/dist/channel-rust-stable.toml
RUN \
mkdir -m 0750 -p packages/official \
&& cd packages/official \
&& curl --location --output rustup-init "$officialBaseUrl"/"$fromTarget"/rustup-init
RUN \
cd packages/official \
&& chmod +x rustup-init \
&& patchelf --set-rpath '/usr/glibc-compat/lib:/usr/lib:/lib' rustup-init \
&& ./rustup-init --verbose -y --no-modify-path --default-toolchain "$channel"
RUN \
patchelf --set-rpath '/usr/glibc-compat/lib:/usr/lib:/lib' /home/rust/.cargo/bin/rustup \
&& patchelf --set-rpath '/usr/glibc-compat/lib:/usr/lib:/lib' /home/rust/.cargo/bin/cargo \
&& patchelf --set-rpath '/usr/glibc-compat/lib:/usr/lib:/lib' /home/rust/.cargo/bin/rust-gdb \
&& patchelf --set-rpath '/usr/glibc-compat/lib:/usr/lib:/lib' /home/rust/.cargo/bin/rust-lldb \
&& patchelf --set-rpath '/usr/glibc-compat/lib:/usr/lib:/lib' /home/rust/.cargo/bin/rustc \
&& patchelf --set-rpath '/usr/glibc-compat/lib:/usr/lib:/lib' /home/rust/.cargo/bin/rustdoc
RUN patchelf --set-rpath '/usr/glibc-compat/lib:/usr/lib:/lib' /home/rust/.multirust/toolchains/"$channel"-"$fromTarget"/bin/cargo \
RUN patchelf --set-rpath '/usr/glibc-compat/lib:/usr/lib:/lib' /home/rust/.multirust/toolchains/"$channel"-"$fromTarget"/bin/rust-gdb \
RUN patchelf --set-rpath '/usr/glibc-compat/lib:/usr/lib:/lib' /home/rust/.multirust/toolchains/"$channel"-"$fromTarget"/bin/rustc \
RUN patchelf --set-rpath '/usr/glibc-compat/lib:/usr/lib:/lib' /home/rust/.multirust/toolchains/"$channel"-"$fromTarget"/bin/rustdoc
ENV PATH=/home/rust/.multirust/toolchains/"$channel"-"$fromTarget"/bin:/usr/bin:/bin
RUN /home/rust/.cargo/bin/rustup target add "$target"
RUN /home/rust/.cargo/bin/rustup update
RUN rustc -Vv


RUN \
cd packages \
&& git clone --recursive --depth 100 https://github.com/rust-lang/rust.git

RUN \
cd packages/rust \
&& git checkout "$rustGitVersion" \
&& git submodule update --init --recursive


RUN \
cd packages \
&& git clone --recursive --depth 100 https://github.com/rust-lang/cargo.git

RUN \
cd packages/cargo \
&& git checkout "$cargoGitVersion" \
&& git submodule update --init --recursive


RUN \
cd packages \
&& git clone --recursive --depth 100 git://git.musl-libc.org/musl

RUN \
cd packages/musl \
&& git checkout "$muslGitVersion" \
&& git submodule update --init --recursive


RUN \
cd packages \
&& git clone --recursive --depth 10 https://github.com/richfelker/musl-cross-make.git

RUN \
cd packages/musl-cross-make \
&& git checkout "$muslCrossMakeVersion" \
&& git submodule update --init --recursive


RUN \
cd packages \
&& git clone --recursive https://github.com/llvm-mirror/libunwind.git

RUN \
cd packages/libunwind \
&& git checkout "$libunwindGitVersion" \
&& git submodule update --init --recursive

RUN \
cd packages \
&& git clone --recursive https://github.com/llvm-mirror/llvm.git

# Yes, we checkout the same branch as libunwind - libunwindGitVersion
RUN \
cd packages/llvm \
&& git checkout "$libunwindGitVersion" \
&& git submodule update --init --recursive



RUN \
cd packages/musl \
&& CFLAGS='-O3 -pipe -g0' ./configure --prefix="$PREFIX" --enable-optimize --enable-wrapper=all \
&& make --jobs 8 \
&& make install

# Disgusting; but the only way we can use Alpine's system linker without installing musl-cross-make - which doesn't support binutils 2.27 (and rustc compilation needs that version)
RUN cp /usr/local/lib/libutil.a /usr/local/lib/libssp_nonshared.a

RUN printf '#include <stdlib.h>\nint main() { return 0; }' >a.c
RUN sed -i -e 's/"\$@"/"$@" --pie/g' /usr/local/bin/musl-gcc
RUN /usr/local/bin/musl-gcc -o a a.c
RUN ./a

RUN \
mv packages/libunwind packages/llvm/projects \
&& cd packages/llvm \
&& mkdir -m 0750 -p projects/libunwind/build \
&& cd projects/libunwind/build \
&& CC=/usr/local/bin/musl-gcc cmake -DLLVM_PATH=../../.. -DLIBUNWIND_ENABLE_SHARED=0 .. \
&& make --jobs 8 \
&& cp lib/libunwind.a "$PREFIX"/lib

RUN \
cd packages/rust \
&& printf '#!/bin/sh\n' >src/libbacktrace/missing \
&& chmod +x src/libbacktrace/missing \
&& sed -i -e '/\${LIB_PREFIX}extra\*\${LIB_SUF}/d' src/etc/local_stage0.sh \
&& sed -i -e 's/CFG_OSTYPE=unknown-linux-gnu/CFG_OSTYPE=unknown-linux-musl/g' configure \
&& sed -i -e 's/#if defined(__linux__) and !defined(__ANDROID__)/#if defined(__GLIBC__)/g' src/llvm/lib/Support/DynamicLibrary.cpp \
&& sed -i -e 's/^namespace llvm/#undef fopen64\n#undef fseeko64\n#undef fstat64\n#undef fstatvfs64\n#undef ftello64\n#undef lstat64\n#undef stat64\n#undef tmpfile64\n\nnamespace llvm/g' src/llvm/include/llvm/Analysis/TargetLibraryInfo.h
RUN \
cd packages/rust \
&& ./configure --help \
&& CC=gcc CXX=g++ CFLAGS='-pipe -g0' CXXFLAGS='-pipe -g0' ./configure \
--prefix="$PREFIX" --build="$target" --host="$target" --target="$target" \
--disable-docs \
--release-channel=nightly \
--musl-root="$PREFIX" --disable-jemalloc --disable-rpath \
--local-rust-root=/home/rust/.cargo/bin --enable-local-rust --enable-local-rebuild \
--disable-manage-submodules --disable-codegen-tests --disable-llvm-assertions --enable-llvm-static-stdcpp
RUN \
cd packages/rust \
&& make help \
&& make tips
RUN \
cd packages/rust \
&& make --jobs 8
RUN \
make install
















# ARG cargoTarget=x86_64-alpine-linux-musl
# ARG independentBaseUrl=https://alpine.geeknet.cz/distfiles
# ARG packageVersion=1.10.0
# ARG cargoPackageVersion=0.11.0-nightly
# RUN \
# cd packages/independent \
# && curl --location --output rustc-"$packageVersion"-"$target".tar.gz "$independentBaseUrl"/rustc-"$packageVersion"-"$target".tar.gz \
# && curl --location --output rustc-"$packageVersion"-"$target".tar.gz.sha1 "$independentBaseUrl"/rustc-"$packageVersion"-"$target".tar.gz.sha1 \
# && sha1sum -c rustc-"$packageVersion"-"$target".tar.gz.sha1
# RUN \
# cd packages/independent \
# && curl --location --output rust-std-"$packageVersion"-"$target".tar.gz "$independentBaseUrl"/rust-std-"$packageVersion"-"$target".tar.gz \
# && curl --location --output rust-std-"$packageVersion"-"$target".tar.gz.sha1 "$independentBaseUrl"/rust-std-"$packageVersion"-"$target".tar.gz.sha1 \
# && sha1sum -c rust-std-"$packageVersion"-"$target".tar.gz.sha1
# RUN \
# cd packages/independent \
# && curl --location --output cargo-"$cargoPackageVersion"-"$cargoTarget".tar.gz "$independentBaseUrl"/cargo-"$cargoPackageVersion"-"$cargoTarget".tar.gz \
# && curl --location --output cargo-"$cargoPackageVersion"-"$cargoTarget".tar.gz.sha1 "$independentBaseUrl"/cargo-"$cargoPackageVersion"-"$cargoTarget".tar.gz.sha1 \
# && sha1sum -c cargo-"$cargoPackageVersion"-"$cargoTarget".tar.gz.sha1
#
# # We separate tarball extraction from download to avoid extracting anything if any download is corrupted or mischevious
# # Note that the extracted cargo tarball has a DIFFERENT root folder naming convention to the other two (it doesn't match tarball name)
# RUN \
# mkdir -m 0750 bootstrap \
# && cp -a packages/independent/home-folder-layout/. bootstrap \
# && cd packages/independent \
# && tar --extract --gzip --file rustc-"$packageVersion"-"$target".tar.gz \
# && cd rustc-"$packageVersion"-x86_64-unknown-linux-musl \
# && ./install.sh --destdir=/home/rust/bootstrap --prefix=.multirust/toolchains/"$channel"-"$target" --components="$(cat components | tr $'\n' ',')" --disable-ldconfig --disable-verify --verbose \
# && cd .. \
# && rm -rf rustc-"$packageVersion"-x86_64-unknown-linux-musl \
# && tar --extract --gzip --file rust-std-"$packageVersion"-"$target".tar.gz \
# && cd rust-std-"$packageVersion"-x86_64-unknown-linux-musl \
# && ./install.sh --destdir=/home/rust/bootstrap --prefix=.multirust/toolchains/"$channel"-"$target" --components="$(cat components | tr $'\n' ',')" --disable-ldconfig --disable-verify --verbose \
# && cd .. \
# && rm -rf rust-std-"$packageVersion"-x86_64-unknown-linux-musl \
# && tar --extract --gzip --file cargo-"$cargoPackageVersion"-"$cargoTarget".tar.gz \
# && cd cargo-nightly-"$target" \
# && ./install.sh --destdir=/home/rust/bootstrap --prefix=.multirust/toolchains/"$channel"-"$target" --components="$(cat components | tr $'\n' ',')" --disable-ldconfig --disable-verify --verbose \
# && cd .. \
# && rm -rf cargo-nightly-"$target"
#
# ENV PATH=/home/rust/bootstrap/.local/bin:/usr/bin:/bin
#
# RUN \
# rustc -Vv
#
# RUN \
# cd packages/cargo \
# && cargo fetch
#
# RUN \
# cd packages/cargo \
# && cargo build --release --verbose
#
# RUN \
# ls -la packages/cargo/target/release
#
# WORKDIR /home/rust/build

# RUN \
# cd packages/rustup.rs \
# && cargo fetch
#
# RUN \
# cd packages/rustup.rs \
# && cargo build --release --verbose

# ENV PREFIX=/home/rust/sysroot
#
# RUN \
# cd packages/musl \
# && CFLAGS='-O3 -pipe -g0' ./configure --prefix="$PREFIX" --disable-shared --enable-optimize --enable-wrapper=all \
# && make \
# && make install
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
