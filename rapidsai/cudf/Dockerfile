FROM brandonschabell/tensorflow-cuda:python3.6

WORKDIR /rapidsai

# Install cmake 3.13
RUN version=3.13 && \
	build=3 && \
	mkdir ~/temp && \
	cd ~/temp && \
	wget https://cmake.org/files/v$version/cmake-$version.$build.tar.gz && \
	tar -xzvf cmake-$version.$build.tar.gz && \
	cd cmake-$version.$build/ && \
	./bootstrap && \
	make -j4 && \
	make install

# Install libboost
RUN apt-get update && \
	apt-get install -y libboost-all-dev

RUN git clone --recurse-submodules https://github.com/rapidsai/cudf.git
WORKDIR /rapidsai/cudf/cpp

ADD https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh /miniconda.sh
RUN sh /miniconda.sh -b -p /conda && /conda/bin/conda update -n base conda
ENV PATH=${PATH}:/conda/bin

RUN conda install -c nvidia nvstrings

RUN mkdir /rapidsai/cudf/cpp/build && \
	cd /rapidsai/cudf/cpp/build && \
	cmake .. -DCMAKE_INSTALL_PREFIX=/install/path && \
	make -j && \
	make install
