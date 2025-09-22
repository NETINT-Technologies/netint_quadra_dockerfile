# Quadra SDK Release Package Dockerfile

Using these Dockerfiles requires access to NETINT SDK package. 
## Instructions
The Dockerfile here can be used to create a docker image with Netint Quadra libxcoder and FFmpeg installed.

1. Copy the Netint SW release package(eg. Quadra_SW_Vx.y.z.zip) to same folder as Dockerfile
2. Generate docker image:

    ```
    sudo docker build --tag ni_quadra_sw .
    ```

   Two `--build-arg` options are supported in Dockerfile:

   **FFmpeg build arguments**

       NI_RELEASE_VERSION=x.y.z        version number of Netint Quadra SDK Release package
       FFMPEG_VERSION=n7.1             version number of FFmpeg to use

    **GSTreamer build arguments**

        NI_RELEASE_VERSION=x.y.z        version number of Netint Quadra SDK Release package
        GST_VER=1.26.2		            Gstreamer Release


3. Start docker targeting quadra NVMe device:
   
   ```
    sudo docker run -it --device=/dev/nvme0 --device=/dev/nvme0n1 ni_quadra_sw /bin/bash
   ```

   Please make sure you are targeting the correct Quadra NVMe device and block paths.
   If you want to give the container sudo permission to control the device, you can add `--privileged` argument 
   
4. Run the test program:

    ```
    cd /NI_Release/FFmpeg-* && \
    bash run_ffmpeg.sh
    ```

### Export Docker Image:

```
sudo docker save ni_quadra_sw | gzip -c > Quadra_Docker.tar.gz
```

### Import Docker Image:

```
gunzip Quadra_Docker.tar.gz && \
sudo docker load -i Quadra_Docker.tar
```

If you need to upgrade Quadra card firmware, use quadra_auto_upgrade.sh in the Netint FW release tarball (Quadra_FW_V*.tar.gz)
