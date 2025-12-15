FROM ubuntu:24.04
LABEL org.opencontainers.image.authors="NETINT Technologies" \
      org.opencontainers.image.description="Docker file example for NETINT VPU using the SDK package"

ARG DEBIAN_FRONTEND=noninteractive \
    NI_RELEASE_VERSION=5.5.0 \
    # Supported GStreamer versions can be found at: https://docs.netint.com/vpu/quadra
    GST_VER=1.26.2 \
    NI_INSTALL_DIR="/NI_Release" \
    NI_PACKAGE_NAME="Quadra_V${NI_RELEASE_VERSION}.zip" \
    NI_PACKAGE_FOLDER="Quadra_V${NI_RELEASE_VERSION}"

SHELL ["/bin/bash", "-c"]

# Copy NETINT SDK to docker $NI_INSTALL_DIR directory
COPY $NI_PACKAGE_NAME $NI_INSTALL_DIR/

# Install required packages and clean up
RUN apt-get update && apt-get install -y \
    nvme-cli yasm pkg-config git gcc make g++ sudo wget uuid-runtime udev unzip python3-pip \
    flex bison libpng-dev zlib1g-dev gnutls-bin uuid-dev libglib2.0-dev libxml2 libxml2-dev \
    libsrt-openssl-dev graphviz curl libaom-dev libaom3 aom-tools nasm \
    ninja-build pipx \
    && pipx install meson \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Add pipx meson to PATH
ENV PATH="/root/.local/bin:${PATH}"
    
# Unzip NETINT SDK
WORKDIR $NI_INSTALL_DIR
RUN set -e && unzip "$NI_PACKAGE_NAME"

# Install libxcoder
RUN set -e && mv ${NI_INSTALL_DIR}/${NI_PACKAGE_FOLDER}/Quadra_SW_V*/libxcoder ${NI_INSTALL_DIR}
WORKDIR ${NI_INSTALL_DIR}/libxcoder
RUN set -e && bash ./build.sh

# Clone GStreamer repo
WORKDIR $NI_INSTALL_DIR
RUN set -e && git clone --depth 1 --branch ${GST_VER} https://gitlab.freedesktop.org/gstreamer/gstreamer.git

# GStreamer patch & install
RUN set -e && cp ${NI_INSTALL_DIR}/${NI_PACKAGE_FOLDER}/Quadra_SW_V*/gstreamer-${GST_VER}_*.diff ${NI_INSTALL_DIR}/gstreamer/
RUN set -e && mv ${NI_INSTALL_DIR}/${NI_PACKAGE_FOLDER}/Quadra_SW_V*/ni-gstreamer-plugins ${NI_INSTALL_DIR}/gstreamer/.
WORKDIR ${NI_INSTALL_DIR}/gstreamer
RUN set -e && patch -t -p 1 < gstreamer-*.diff
RUN set -e && bash ./build_gstreamer.sh --install
WORKDIR ${NI_INSTALL_DIR}/gstreamer/ni-gstreamer-plugins    
RUN set -e && bash ./build_ni_gst_plugins.sh -i

RUN set -e && export LD_LIBRARY_PATH=/usr/local/lib/x86_64-linux-gnu/
RUN set -e && export GST_PLUGIN_SYSTEM_PATH=/usr/local/lib/x86_64-linux-gnu/gstreamer-1.0
RUN set -e && export GST_PLUGIN_PATH=/usr/local/lib/x86_64-linux-gnu/gstreamer-1.0

RUN set -e && ldconfig

# Create entrypoint script to initialize NETINT VPUs on startup
USER root
COPY entrypoint.sh /usr/local/bin/
RUN set -e && chmod +x /usr/local/bin/entrypoint.sh
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]