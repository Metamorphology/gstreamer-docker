# gstreamer-docker

GStreamer builds in a form of a bare Docker image.

## Usage example

```Dockerfile

# (optionally) declare gstreamer-docker image name as an arg, so that you can:
# - specify it on the command line to docker build
# - specify in docker-compose file
# - specify in GitHub actions file
ARG GSTREAMER_IMAGE=gstreamer-docker:1.18.4-debian

# pull the gstreamer-docker image as a build step
FROM ${GSTREAMER_IMAGE} AS gst-distro



# build your target image as usual
FROM debian:buster-slim

# NOTE: the X-related packages listed below are needed for GStreamer to work!
# add more packages that your target image needs, but do not remove libx11-6, libxrender1, libxext6.
RUN apt-get update \
	&& apt-get install -y --no-install-recommends \
		ca-certificates \
		xz-utils \
		libx11-6 \
		libxrender1 \
		libxext6 \
	&& rm -rf /var/lib/apt/lists/*

# ... add more setup as needed


# copy GStreamer packages from the gstreamer-docker image
COPY --from=gst-distro /gstreamer-1.0-*.tar.xz /

# setup GStreamer environment
ENV LD_LIBRARY_PATH=/usr/local/lib:/usr/lib
ENV GST_PLUGIN_PATH=/usr/local/lib/gstreamer-1.0
ENV GST_PLUGIN_SCANNER=/usr/local/libexec/gstreamer-1.0/gst-plugin-scanner

# uncomment the following line if you don't need GStreamer development libraries in the target image
# RUN rm /gstreamer-1.0-*-devel.tar.xz

# extract GStreamer files to your target image
# the gst-inspect-1.0 invocation is optional, but it does two good things:
# 1. it shows whether GStreamer installation is usable and what plugins are available,
# 2. it generates plugin cache so that your container wouldn't need to do that on startup.
RUN cd /usr/local/ \
    && tar xf /gstreamer-1.0-*.tar.xz \
    && cd / \
    && rm /gstreamer-1.0-*.tar.xz \
    && gst-inspect-1.0 -b

# ... add more setup for your image as needed


```
