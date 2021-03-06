FROM ubuntu:xenial

LABEL maintainer="Conan.io <info@conan.io>"

ENV PYENV_ROOT=/opt/pyenv \
    PATH=/opt/pyenv/shims:/usr/local/bin:${PATH} \
    CXX=/usr/bin/g++ \
    CC=/usr/bin/gcc

RUN apt-get -qq update \
    && apt-get -qq install -y --no-install-recommends \
       sudo \
       build-essential \
       wget \
       git \
       libc6-dev \
       gcc \
       libgmp-dev \
       libmpfr-dev \
       libmpc-dev \
       libc6-dev \
       nasm \
       dh-autoreconf \
       ninja-build  \
       libffi-dev \
       libssl-dev \
       pkg-config \
       subversion \
       zlib1g-dev \
       libbz2-dev \
       libsqlite3-dev \
       libreadline-dev \
       xz-utils \
       curl \
       libncurses5-dev \
       libncursesw5-dev \
       liblzma-dev \
       ca-certificates \
       autoconf-archive \
       flex \
    && rm -rf /var/lib/apt/lists/*

RUN wget -q --no-check-certificate https://cmake.org/files/v3.17/cmake-3.17.2-Linux-x86_64.tar.gz \
    && tar -xzf cmake-3.17.2-Linux-x86_64.tar.gz \
       --exclude=bin/cmake-gui \
       --exclude=doc/cmake \
       --exclude=share/cmake-3.12/Help \
    && cp -fR cmake-3.17.2-Linux-x86_64/* /usr \
    && rm -rf cmake-3.17.2-Linux-x86_64 \
    && rm cmake-3.17.2-Linux-x86_64.tar.gz

RUN wget --no-check-certificate --quiet -O /opt/gcc-10.1.0.tar.gz https://github.com/gcc-mirror/gcc/archive/releases/gcc-10.1.0.tar.gz \
    && tar zxf /opt/gcc-10.1.0.tar.gz -C /opt \
    && cd /opt/gcc-releases-gcc-10.1.0 \
    && ./configure --prefix=/usr/local \
                   --enable-languages=c,c++ \
                   --disable-bootstrap \
                   --with-system-zlib \
                   --enable-multiarch \
                   --disable-multilib \
                   --enable-shared \
                   --enable-threads=posix \
                   --build=x86_64-linux-gnu \
                   --host=x86_64-linux-gnu \
                   --target=x86_64-linux-gnu \
                   --without-included-gettext \
                   --with-tune=generic \
                   --with-gmp=/usr/local/lib \
                   --with-mpc=/usr/lib \
                   --with-mpfr=/usr/lib \
                   --disable-checking \
    && make -j "$(nproc)" \
    && make install-strip \
    && cd - \
    && rm -rf /opt/gcc* \
    && apt-get remove -y gcc \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && update-alternatives --install /usr/bin/gcc gcc /usr/local/bin/gcc 100 \
    && update-alternatives --install /usr/bin/cc cc /usr/local/bin/gcc 100 \
    && update-alternatives --install /usr/bin/g++ g++ /usr/local/bin/g++ 100 \
    && update-alternatives --install /usr/bin/c++ c++ /usr/local/bin/g++ 100 \
    && update-alternatives --install /usr/bin/cpp cpp /usr/local/bin/g++ 100 \
    && printf "/usr/local/lib64" > /etc/ld.so.conf.d/local-lib64.conf \
    && ldconfig -v

RUN groupadd 1001 -g 1001 \
    && groupadd 1000 -g 1000 \
    && groupadd 2000 -g 2000 \
    && groupadd 999 -g 999 \
    && useradd -ms /bin/bash conan -g 1001 -G 1000,2000,999 \
    && printf "conan:conan" | chpasswd \
    && adduser conan sudo \
    && printf "conan ALL= NOPASSWD: ALL\\n" >> /etc/sudoers

RUN wget --no-check-certificate --quiet -O /tmp/pyenv-installer https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer \
    && chmod +x /tmp/pyenv-installer \
    && /tmp/pyenv-installer \
    && rm /tmp/pyenv-installer \
    && update-alternatives --install /usr/bin/pyenv pyenv /opt/pyenv/bin/pyenv 100 \
    && PYTHON_CONFIGURE_OPTS="--enable-shared" pyenv install 3.8.1 \
    && pyenv global 3.8.1 \
    && pip install -q --upgrade --no-cache-dir pip \
    && pip install -q --no-cache-dir conan conan-package-tools \
    && chown -R conan:1001 /opt/pyenv \
    # remove all __pycache__ directories created by pyenv
    && find /opt/pyenv -iname __pycache__ -print0 | xargs -0 rm -rf \
    && update-alternatives --install /usr/bin/python python /opt/pyenv/shims/python 100 \
    && update-alternatives --install /usr/bin/python3 python3 /opt/pyenv/shims/python3 100 \
    && update-alternatives --install /usr/bin/pip pip /opt/pyenv/shims/pip 100 \
    && update-alternatives --install /usr/bin/pip3 pip3 /opt/pyenv/shims/pip3 100

USER conan
WORKDIR /home/conan

RUN mkdir -p /home/conan/.conan \
    && printf 'eval "$(pyenv init -)"\n' >> ~/.bashrc \
    && printf 'eval "$(pyenv virtualenv-init -)"\n' >> ~/.bashrc
