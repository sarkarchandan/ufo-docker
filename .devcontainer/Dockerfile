FROM nvcr.io/nvidia/cuda:12.4.1-devel-ubuntu20.04

ENV G_MESSAGES_DEBUG=all

# Install dependencies
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update \
	&& apt-get install -y software-properties-common \
	&& add-apt-repository -y ppa:cnugteren/clblast \
	&& apt-get update \
	&& apt-get install -y apt-utils build-essential meson ninja-build git gobject-introspection \
		ca-certificates cmake liblapack-dev libjpeg-dev libtiff-dev libtiff5-dev libglib2.0-dev libjson-glib-dev \
		libopenmpi-dev libhdf5-dev libclfft-dev libgsl-dev libgirepository1.0-dev qtbase5-dev python3 \
		python3-dev python3-gi python-gi-dev python3-sphinx python3-sphinxcontrib.bibtex python3-pip \
		python3-cairo python3-gi-cairo python3-pyqt5 python3-pyqtgraph pkg-config ocl-icd-libopencl1 \
		ocl-icd-opencl-dev fftw3-dev opencl-headers clinfo zlib1g-dev gtk-doc-tools asciidoc vim \
		libpangocairo-1.0-0 libpython3-dev libopencv-dev python3-opencv libzmq3-dev libclblast-dev python3-jinja2 \
	&& rm -rf /var/lib/apt/lists/*

RUN pip3 install --upgrade pip \
	&& pip3 install setuptools tifffile scikit-image numpy==1.22.0

# Configure compute environment
ENV LD_LIBRARY_PATH /usr/local/lib/:${LD_LIBRARY_PATH}
ENV GI_TYPELIB_PATH=/usr/local/lib/girepository-1.0:$GI_TYPELIB_PATH
ENV PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH

RUN mkdir -p /etc/OpenCL/vendors \
    && echo "libnvidia-opencl.so.1" > /etc/OpenCL/vendors/nvidia.icd
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility

# Clone repositories and build
RUN git clone https://github.com/ufo-kit/ufo-core.git \
    && git clone -b meson_deps https://github.com/ufo-kit/ufo-filters.git \
    && git clone https://github.com/ufo-kit/tofu.git

RUN cd /ufo-core \
	&& meson build --libdir=lib -Dbashcompletiondir=$HOME/.local/share/bash-completion/completions \
	&& cd build \
	&& ninja install
RUN cd /ufo-core/python \
	&& python3 setup.py install
RUN cd /ufo-filters \
	&& meson build --libdir=lib -Dcontrib_filters=True \
	&& cd build \
	&& ninja install
RUN cd /tofu \
	&& pip3 install -r requirements-flow.txt \
	&& pip3 install -r requirements-flow.txt \
	&& python3 setup.py install
RUN rm -rf /ufo-core /ufo-filters /tofu
