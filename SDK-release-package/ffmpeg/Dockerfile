FROM ubuntu:24.04
LABEL org.opencontainers.image.authors="NETINT Technologies" \
      org.opencontainers.image.description="Docker file example for NETINT VPU using the SDK package"

ARG DEBIAN_FRONTEND=noninteractive \
    NI_RELEASE_VERSION=5.4.0 \
    NI_PACKAGE_NAME="Quadra_V${NI_RELEASE_VERSION}.zip" \
    NI_PACKAGE_FOLDER="Quadra_V${NI_RELEASE_VERSION}" \
    #Supported FFmpeg versions can be found at: https://docs.netint.com/vpu/quadra
    FFMPEG_VERSION=n7.1 \
    FFMPEG_PACKAGE_NAME="${FFMPEG_VERSION}.tar.gz" 

SHELL ["/bin/bash", "-c"]

#packages install
RUN apt-get update && \
    apt-get install -y pkg-config git gcc make g++ sudo wget uuid-runtime udev unzip yasm nvme-cli

#copy ni release package to docker /NI_Release directory
COPY $NI_PACKAGE_NAME /NI_Release/
WORKDIR /NI_Release
RUN wget -c https://github.com/FFmpeg/FFmpeg/archive/refs/tags/${FFMPEG_VERSION}.tar.gz

#SW package install
WORKDIR /NI_Release
RUN unzip "$NI_PACKAGE_NAME" && \
    tar -xzf "$FFMPEG_PACKAGE_NAME"

RUN cp /NI_Release/"$NI_PACKAGE_FOLDER"/Quadra_SW_V"$NI_RELEASE_VERSION"_*/FFmpeg-"$FFMPEG_VERSION"_netint_v"$NI_RELEASE_VERSION"_*.diff /NI_Release/FFmpeg-"$FFMPEG_VERSION"/ && \
    mv /NI_Release/"$NI_PACKAGE_FOLDER"/Quadra_SW_V"$NI_RELEASE_VERSION"_*/libxcoder /NI_Release
#libxcoder install
WORKDIR /NI_Release/libxcoder
RUN bash ./build.sh
#ffmpeg install
WORKDIR /NI_Release/FFmpeg-"$FFMPEG_VERSION"
RUN patch -t -p 1 < FFmpeg-"$FFMPEG_VERSION"_netint_v"$NI_RELEASE_VERSION"_*.diff && \
    chmod u+x build_ffmpeg.sh && \
    chmod u+x run_ffmpeg_quadra.sh
ENV PKG_CONFIG_PATH=/usr/local/lib/pkgconfig/
RUN echo 'y' | bash ./build_ffmpeg.sh --ffprobe --shared && \
    make install
ENV LD_LIBRARY_PATH=/usr/local/lib/ 
RUN ldconfig

# Create entrypoint script to initialize NETINT VPUs on startup
USER root
COPY entrypoint.sh /usr/local/bin/
RUN set -e && chmod +x /usr/local/bin/entrypoint.sh
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
