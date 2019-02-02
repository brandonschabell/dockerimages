FROM brandonschabell/cuda-python:cuda10-cudnn7.4-python3.6-ubuntu18.04

LABEL maintainer="Brandon Schabell <brandonschabell@gmail.com>"

ENV TF_CUDA_COMPUTE_CAPABILITIES=3.5,5.2,6.0,6.1,7.0
ENV TF_CUDA_VERSION=10.0
ENV TF_CUDNN_VERSION=7
ENV NCCL_VERSION=2.3.7-1+cuda10.0


RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        cuda-command-line-tools-10-0 \
        cuda-cublas-dev-10-0 \
        cuda-cudart-dev-10-0 \
        cuda-cufft-dev-10-0 \
        cuda-curand-dev-10-0 \
        cuda-cusolver-dev-10-0 \
        cuda-cusparse-dev-10-0 \
	curl \
        libcudnn7=7.4.1.5-1+cuda10.0 \
        libcudnn7-dev=7.4.1.5-1+cuda10.0 \
	libnccl2=${NCCL_VERSION} \
        libnccl-dev=${NCCL_VERSION} \
        libcurl3-dev \
        libfreetype6-dev \
        libhdf5-serial-dev \
        libpng-dev \
        libzmq3-dev \
        pkg-config \
        rsync \
        software-properties-common \
        unzip \
        zip \
        zlib1g-dev \
        wget \
        git \
        && \
    find /usr/local/cuda-10.0/lib64/ -type f -name 'lib*_static.a' -not -name 'libcudart_static.a' -delete && \
    rm /usr/lib/x86_64-linux-gnu/libcudnn_static_v7.a

RUN apt-get update && \
        apt-get install nvinfer-runtime-trt-repo-ubuntu1804-5.0.2-ga-cuda10.0 \
        && apt-get update \
        && apt-get install -y --no-install-recommends libnvinfer-dev=5.0.2-1+cuda10.0 \
        && rm -rf /var/lib/apt/lists/*

RUN pip --no-cache-dir install \
        Pillow \
        h5py \
        ipykernel \
        jupyter \
        keras_applications \
        keras_preprocessing \
        matplotlib \
        mock \
        numpy \
        scipy \
        sklearn \
        pandas \
        && \
    python -m ipykernel.kernelspec

# RUN ln -s -f /usr/bin/python3 /usr/bin/python#

# Set up our notebook config.
COPY jupyter_notebook_config.py /root/.jupyter/

# Jupyter has issues with being run directly:
#   https://github.com/ipython/ipython/issues/7062
# We just add a little wrapper script.
COPY run_jupyter.sh /

# Set up Bazel.

# Running bazel inside a `docker build` command causes trouble, cf:
#   https://github.com/bazelbuild/bazel/issues/134
# The easiest solution is to set up a bazelrc file forcing --batch.
RUN echo "startup --batch" >>/etc/bazel.bazelrc
# Similarly, we need to workaround sandboxing issues:
#   https://github.com/bazelbuild/bazel/issues/418
RUN echo "build --spawn_strategy=standalone --genrule_strategy=standalone" \
    >>/etc/bazel.bazelrc

# Install the most recent bazel release.
ENV BAZEL_VERSION 0.18.1
WORKDIR /
RUN mkdir /bazel && \
    cd /bazel && \
    curl -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36" -fSsL -O https://github.com/bazelbuild/bazel/releases/download/$BAZEL_VERSION/bazel-$BAZEL_VERSION-installer-linux-x86_64.sh && \
    curl -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.133 Safari/537.36" -fSsL -o /bazel/LICENSE.txt https://raw.githubusercontent.com/bazelbuild/bazel/master/LICENSE && \
    chmod +x bazel-*.sh && \
    ./bazel-$BAZEL_VERSION-installer-linux-x86_64.sh && \
    cd / && \
    rm -f /bazel/bazel-$BAZEL_VERSION-installer-linux-x86_64.sh

# Download and build TensorFlow.
WORKDIR /tensorflow
RUN git clone --branch=r1.12 --depth=1 https://github.com/tensorflow/tensorflow.git .

# Configure the build for our CUDA configuration.
ENV CI_BUILD_PYTHON python
ENV LD_LIBRARY_PATH /usr/local/cuda/extras/CUPTI/lib64:$LD_LIBRARY_PATH
ENV TF_NEED_CUDA 1
ENV TF_NEED_TENSORRT 1

RUN cp /tensorflow/tools/bazel.rc /tensorflow/.tf_configure.bazelrc

RUN ln -s /usr/local/cuda/lib64/stubs/libcuda.so /usr/local/cuda/lib64/stubs/libcuda.so.1 && \
    LD_LIBRARY_PATH=/usr/local/cuda/lib64/stubs:${LD_LIBRARY_PATH} \
    tensorflow/tools/ci_build/builds/configured GPU \
    bazel build -c opt --copt=-mavx --config=cuda \
	--cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" \
        tensorflow/tools/pip_package:build_pip_package && \
    rm /usr/local/cuda/lib64/stubs/libcuda.so.1 && \
    bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/pip && \
    pip --no-cache-dir install --upgrade /tmp/pip/tensorflow-*.whl && \
    rm -rf /tmp/pip && \
    rm -rf /root/.cache
# Clean up pip wheel and Bazel cache when done.

WORKDIR /root

# TensorBoard
EXPOSE 6006
# IPython
EXPOSE 8888

WORKDIR "/notebooks"

CMD ["/run_jupyter.sh", "--allow-root"]
