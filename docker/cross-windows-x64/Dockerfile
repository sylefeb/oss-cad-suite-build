FROM fedora:43

RUN dnf install -y  mingw64-gcc
RUN dnf install -y  mingw64-gcc-c++
RUN dnf install -y  mingw64-libftdi
RUN dnf install -y  mingw64-boost
RUN dnf install -y  mingw64-pkg-config
RUN dnf install -y  mingw64-tcl
RUN dnf install -y  mingw64-readline
RUN dnf install -y  mingw64-tk
RUN dnf install -y  mingw64-xz
RUN dnf install -y  mingw64-gtk3
RUN dnf install -y  mingw64-qt5-qtbase
RUN dnf install -y  mingw64-eigen3
RUN dnf install -y  mingw64-dlfcn
RUN dnf install -y  mingw64-hidapi
RUN dnf install -y  mingw64-bzip2
RUN dnf install -y  mingw64-curl
RUN dnf install -y  mingw64-gmp
RUN dnf install -y  mingw64-crt
RUN dnf install -y  mingw-w64-tools
RUN dnf install -y  git
RUN dnf install -y  cmake
RUN dnf install -y  autoconf
RUN dnf install -y  autoconf-archive
RUN dnf install -y  automake
RUN dnf install -y  flex
RUN dnf install -y  bison
RUN dnf install -y  patch
RUN dnf install -y  gperf
RUN dnf install -y  binutils
RUN dnf install -y  gcc
RUN dnf install -y  g++
RUN dnf install -y  bc
RUN dnf install -y  rsync
RUN dnf install -y  python3
RUN dnf install -y  which
RUN dnf install -y  libtool
RUN dnf install -y  libtool-ltdl-devel
RUN dnf install -y  diffutils
# RUN dnf install -y  java-11-openjdk-headless
RUN dnf install -y  java-21-openjdk-headless
RUN dnf install -y  cpio
RUN dnf install -y  xz
RUN dnf install -y  help2man
RUN dnf install -y  wget boost-devel
RUN rpm -qi mingw64-crt
RUN dnf upgrade 'mingw*'

RUN git clone --recursive https://github.com/gsauthof/pe-util && \
    cd pe-util && \
    CC=gcc CXX=g++ LD=g++ cmake . && \
    make install -j9

ENV CROSS_NAME x86_64-w64-mingw32

ENV AS=/usr/bin/${CROSS_NAME}-as \
    AR=/usr/bin/${CROSS_NAME}-ar \
    CC=/usr/bin/${CROSS_NAME}-gcc \
    CPP=/usr/bin/${CROSS_NAME}-cpp \
    CXX=/usr/bin/${CROSS_NAME}-g++ \
    LD=/usr/bin/${CROSS_NAME}-ld \
    STRIP=/usr/bin/${CROSS_NAME}-strip

ENV PKG_CONFIG_PATH /usr/x86_64-w64-mingw32/sys-root/mingw/lib/pkgconfig

ENV CROSS_PREFIX /opt/${CROSS_NAME}

RUN cp /usr/include/FlexLexer.h /usr/x86_64-w64-mingw32/sys-root/mingw/include/.

COPY Toolchain.cmake ${CROSS_PREFIX}/

ENV CMAKE_TOOLCHAIN_FILE ${CROSS_PREFIX}/Toolchain.cmake

ENV RUSTUP_HOME /opt/rust/rustup

ENV PATH ${PATH}:/opt/rust/cargo/bin

RUN curl https://sh.rustup.rs -sSf | RUSTUP_HOME=/opt/rust/rustup CARGO_HOME=/opt/rust/cargo bash -s -- --default-toolchain stable --profile default --no-modify-path -y

RUN rustup target add x86_64-pc-windows-gnu && \
    mkdir -p /.cargo && \
    echo "[target.x86_64-pc-windows-gnu]" > /.cargo/config && \
    echo "linker = \"x86_64-w64-mingw32-gcc\"" >> /.cargo/config && \
    echo "ar = \"x86_64-w64-mingw32-ar\"" >> /.cargo/config

# RUN ln -s /usr/bin/python3 /usr/bin/python

RUN cd /tmp \
    && curl -L http://archives.fedoraproject.org/pub/archive/fedora/linux/releases/39/Everything/source/tree/Packages/m/mingw-gmp-6.1.2-16.fc39.src.rpm --output mingw-gmp-6.1.2-16.fc39.src.rpm
RUN cd /tmp \
    && rpm2cpio mingw-gmp-6.1.2-16.fc39.src.rpm | cpio -idmv \
    && tar -xf gmp-6.1.2.tar.xz \
    && cd gmp-6.1.2
RUN curl -L "https://gitlab.archlinux.org/archlinux/packaging/packages/gmp/-/raw/main/gmp-gcc-15.patch?ref_type=heads&inline=false" --output /tmp/gmp.patch \
    && cd /tmp/gmp-6.1.2 \
    && patch -p1 < /tmp/gmp.patch
RUN cd /tmp/gmp-6.1.2 \
    && autoconf
RUN cd /tmp/gmp-6.1.2 \
    && mingw64-configure --disable-shared --enable-cxx --enable-static ; cat config.log \
    && mingw64-make -j9 \
    && mingw64-make install

RUN cd /tmp \
    && curl http://ftp.gnu.org/gnu/bison/bison-3.5.tar.gz --output /tmp/bison-3.5.tar.gz
RUN cd /tmp \
    && curl -L https://raw.githubusercontent.com/reactos/RosBE/master/Patches/bison-3.5-reactos-fix-win32-build.patch --output /tmp/bison.patch
RUN cd /tmp \
    && tar -xzf bison-3.5.tar.gz
RUN cd /tmp \
    && cd bison-3.5 \
    && patch -p1 < /tmp/bison.patch \
    && mingw64-configure \
    && mingw64-make CFLAGS+="-Wno-incompatible-pointer-types" -j9 \
    && mingw64-make install
RUN cd \
    && rm -rf /tmp/*

WORKDIR /work

RUN RUSTUP_HOME=/opt/rust/rustup CARGO_HOME=/opt/rust/cargo rustup toolchain install nightly
RUN RUSTUP_HOME=/opt/rust/rustup CARGO_HOME=/opt/rust/cargo rustup override set nightly
RUN RUSTUP_HOME=/opt/rust/rustup CARGO_HOME=/opt/rust/cargo rustup target add x86_64-pc-windows-gnu
RUN dnf install -y  openssl
