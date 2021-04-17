FROM nvidia/cuda:11.1-cudnn8-devel-ubuntu20.04

ENV DEBIAN_FRONTEND="noninteractive"
ENV TZ="Etc/UTC"
ENV PATH="/app/ffmpeg/bin:${PATH}"
ENV PATH="/app/opencv/bin:${PATH}"
ENV NVCCPARAMS="-O3"

RUN mkdir -p /app/source && \
    mkdir -p /app/ffmpeg && \
    mkdir -p /app/opencv
WORKDIR /app/source
COPY src-ffmpeg src-ffmpeg
COPY src-dav1d src-dav1d
COPY src-nvcodec src-nvcodec
COPY src-svtav1 src-svtav1

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    autoconf \
    automake \
    build-essential \
    ca-certificates \
    ccache \
    cmake \
    curl \
    flake8 \
    git-core \
    gnu-standards \
    gobjc \
    gobjc++ \
    libaom-dev \
    libass-dev \
    libdrm-dev \
    libegl-dev \
    libelf-dev \
    libfdk-aac-dev \
    libfreetype6-dev \
    libgif-dev \
    libgnutls28-dev \
    libharfbuzz-dev \
    libjack-jackd2-dev \
    libmp3lame-dev \
    libnuma-dev \
    libopengl-dev \
    libopenjp2-7-dev \
    libopus-dev \
    libpng-dev \
    libpng++-dev \
    librsvg2-dev \
    libsass-dev \
    libsdl2-dev \
    libssh-dev \
    libssl-dev \
    libtesseract-dev \
    libtiff-opengl \
    libtiff5-dev \
    libtool \
    libunistring-dev \
    libv4l-dev \
    libva-dev \
    libvdpau-dev \
    libvidstab-dev \
    libvorbis-dev \
    libvpx-dev \
    libwebp-dev \
    libx264-dev \
    libx265-dev \
    libxcb-shm0-dev \
    libxcb-xfixes0-dev \
    libxcb1-dev \
    libxvidcore-dev \
    libzstd-dev \
    make \
    meson \
    nasm \
    ninja-build\
    p7zip-full \
    pkg-config \
    pylint \
    python3 \
    python3-dev \
    python3-pip \
    python3-setuptools \
    python3-venv \
    software-properties-common \
    texinfo \
    wget \
    yasm \
    zlib1g-dev && \
    rm -rf /var/lib/apt/lists/*

### FFmpeg

RUN mkdir /app/source/src-dav1d/build && \
    cd /app/source/src-dav1d/build && \
    meson setup \
    -Denable_tools="false" \
    -Denable_test="false" \
    --default-library="static" .. \
    --prefix="/app/ffmpeg" \
    --libdir="/app/ffmpeg/lib" && \
    ninja && \
    ninja install

RUN mkdir /app/source/src-svtav1/build && \
    cd /app/source/src-svtav1/build && \
    cmake -G "Unix Makefiles" \
    -DCMAKE_INSTALL_PREFIX="/app/ffmpeg" \
    -DCMAKE_BUILD_TYPE="Release" \
    -DBUILD_DEC="ON" \
    -DBUILD_SHARED_LIBS="OFF" .. && \
    make -j$(nproc) && \
    make install

RUN cd /app/source/src-nvcodec && \
    make install

ARG CUDA_ARCH
ENV NVCCPARAMS="${NVCCPARAMS} -gencode arch=compute_${CUDA_ARCH},code=sm_${CUDA_ARCH}"

RUN cd /app/source/src-ffmpeg && \
    PKG_CONFIG_PATH="/app/ffmpeg/lib/pkgconfig" ./configure \
    --bindir="/app/ffmpeg/bin" \
    --disable-debug \
    --enable-cuda-nvcc \
    --enable-gpl \
    --enable-hardcoded-tables \
    --enable-libaom \
    --enable-libass \
    --enable-libdav1d \
    --enable-libdrm \
    --enable-libfdk-aac \
    --enable-libfreetype \
    --enable-libmp3lame \
    --enable-libnpp \
    --enable-libopus \
    --enable-libsvtav1 \
    --enable-libvidstab \
    --enable-libvorbis \
    --enable-libvpx \
    --enable-libwebp \
    --enable-libx264 \
    --enable-libx265 \
    --enable-libxvid \
    --enable-nonfree \
    --enable-nvenc \
    --enable-opengl \
    --enable-openssl \
    --enable-vaapi \
    --extra-cflags="-I/app/ffmpeg/include -I/usr/local/cuda/include" \
    --extra-ldflags="-L/app/ffmpeg/lib -L/usr/local/cuda/lib64" \
    --extra-libs="-lpthread -lm" \
    --nvccflags="${NVCCPARAMS}" \
    --pkg-config-flags="--static" \
    --prefix="/app/ffmpeg" && \
    make -j$(nproc) && \
    make install

RUN rm -rf src-ffmpeg src-dav1d src-nvcodec src-svtav

### OpenCV
