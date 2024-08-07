FROM ubuntu:20.04
MAINTAINER Steven Zhou

ENV REFRESHED_AT=2024-01-29
ARG NI_RELEASE_VERSION=4.7.0
ARG NI_PACKAGE_NAME="Quadra_V${NI_RELEASE_VERSION}.zip"
ARG NI_PACKAGE_FOLDER="Quadra_V${NI_RELEASE_VERSION}"
ARG RC=RC4
#FFMPEG_VERSION can be: n3.4.2, n4.1.3, n4.2.1, n4.3, n4.3.1, n4.4, n4.4.2, n5.0, n5.1.2, n6.0, n6.1, n7.0
ARG FFMPEG_VERSION=n5.0
ARG FFMPEG_PACKAGE_NAME="${FFMPEG_VERSION}.tar.gz"

ARG DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-c"]

#packages install
RUN apt-get update
RUN apt-get install -y pkg-config git gcc make g++ sudo wget uuid-runtime udev unzip

#copy ni release package to docker /NI_Release directory
COPY $NI_PACKAGE_NAME /NI_Release/
WORKDIR /NI_Release
RUN wget -c https://github.com/FFmpeg/FFmpeg/archive/refs/tags/${FFMPEG_VERSION}.tar.gz

#nvme cli install
WORKDIR /NI_Release
RUN wget -c https://github.com/linux-nvme/nvme-cli/archive/refs/tags/v1.16.tar.gz
RUN tar -xzf v1.16.tar.gz
WORKDIR /NI_Release/nvme-cli-1.16/
RUN make
RUN make install
RUN nvme list

#yasm install
WORKDIR /NI_Release
RUN wget -c http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz
RUN tar -zxf yasm-1.3.0.tar.gz
WORKDIR /NI_Release/yasm-1.3.0/
RUN ./configure
RUN make
RUN make install

#SW package install
WORKDIR /NI_Release
RUN unzip "$NI_PACKAGE_NAME"
RUN tar -xzf "$FFMPEG_PACKAGE_NAME"

RUN cp /NI_Release/"$NI_PACKAGE_FOLDER"/Quadra_SW_V"$NI_RELEASE_VERSION"_"$RC"/FFmpeg-"$FFMPEG_VERSION"_netint_v"$NI_RELEASE_VERSION"_"$RC".diff /NI_Release/FFmpeg-"$FFMPEG_VERSION"/
RUN mv /NI_Release/"$NI_PACKAGE_FOLDER"/Quadra_SW_V"$NI_RELEASE_VERSION"_"$RC"/libxcoder /NI_Release
#libxcoder install
WORKDIR /NI_Release/libxcoder
RUN bash ./build.sh
#ffmpeg install
WORKDIR /NI_Release/FFmpeg-"$FFMPEG_VERSION"
RUN patch -t -p 1 < FFmpeg-"$FFMPEG_VERSION"_netint_v"$NI_RELEASE_VERSION"_"$RC".diff
RUN chmod u+x build_ffmpeg.sh
RUN chmod u+x run_ffmpeg_quadra.sh
ENV PKG_CONFIG_PATH /usr/local/lib/pkgconfig/
RUN echo 'y' | bash ./build_ffmpeg.sh --ffprobe --shared
RUN make install

ENV LD_LIBRARY_PATH=/usr/local/lib/:$LD_LIBRARY_PATH
RUN ldconfig
CMD echo "------end-----"
#every time docker is run please run this command inside docker first: ni_rsrc_mon
