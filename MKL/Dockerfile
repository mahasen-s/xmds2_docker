FROM ubuntu:18.04
RUN apt-get update && apt-get install -y build-essential \
                libhdf5-dev \
                wget \
                vim
# Install miniconda and necessary python packages
WORKDIR /home
RUN wget https://repo.continuum.io/miniconda/Miniconda2-latest-Linux-x86_64.sh && \
    bash Miniconda2-latest-Linux-x86_64.sh -b -p /home/miniconda2 && \
    rm Miniconda2-latest-Linux-x86_64.sh
ENV PATH="/home/miniconda2/bin:${PATH}"
RUN conda install setuptools cheetah numpy pyparsing lxml mpmath sphinx h5py -y
RUN wget https://download.open-mpi.org/release/open-mpi/v3.1/openmpi-3.1.2.tar.bz2 && \
    tar -xvjf openmpi-3.1.2.tar.bz2 && \
    rm openmpi-3.1.2.tar.bz2

# Build openMPI
RUN mkdir lib
RUN mkdir lib/openmpi
WORKDIR /home/openmpi-3.1.2
RUN ./configure --prefix=/home/lib/openmpi
RUN make all -j`nproc` && make install
ENV PATH="/home/lib/openmpi/bin:${PATH}"
# Build FFTW3
WORKDIR /home
RUN mkdir /home/lib/fftw
RUN wget http://www.fftw.org/fftw-3.3.8.tar.gz && \
    tar -xvzf fftw-3.3.8.tar.gz && \
    rm fftw-3.3.8.tar.gz
WORKDIR /home/fftw-3.3.8
RUN ./configure --enable-threads --enable-open --enable-openmp --enable-mpi --enable-sse2 --enable-avx --enable-avx2 --enable-avx512 --enable-avx-128-fma --prefix=/home/lib/fftw LDFLAGS=-L/home/lib/openmpi/lib CPPFLAGS=-I/home/lib/openmpi/include
RUN make -j`nproc` && make install
ENV PATH="/home/lib/fftw/bin:${PATH}"
# Fix OS level throttling by setting performance for each core governor. This may need to be done at a host level too for correct ATLAS compilation.
# --criple-atlas-performance flag will bypass this requirement
RUN apt-get install cpufrequtils gfortran -y
RUN echo 'GOVERNOR="performance"' | tee /etc/default/cpufrequtils && \
    /etc/init.d/cpufrequtils restart

# Build ATLAS. N.B. Compiler and linker flags still need to be set; do not use this block
#WORKDIR /home
#RUN wget https://nchc.dl.sourceforge.net/project/math-atlas/Stable/3.10.3/atlas3.10.3.tar.bz2 && \
#    tar -xvjf atlas3.10.3.tar.bz2 && \
#    rm atlas3.10.3.tar.bz2
#RUN wget http://www.netlib.org/lapack/lapack-3.4.1.tgz
#WORKDIR /home/ATLAS/build
#RUN /home/ATLAS/configure --with-netlib-lapack-tarfile=/home/lapack-3.4.1.tgz --prefix=/home/lib/atlas --cripple-atlas-performance
#RUN make build  && \
#    make check && \
#    make ptcheck && \
#    make time && \
#    make install

# Setup Intel MKL
RUN wget https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS-2019.PUB && \
    apt-key add GPG-PUB-KEY-INTEL-SW-PRODUCTS-2019.PUB
RUN wget https://apt.repos.intel.com/setup/intelproducts.list -O /etc/apt/sources.list.d/intelproducts.list
RUN apt-get update && \
    apt-get install intel-mkl-64bit-2019.0-045 -y
#ENV export CXXFLAGS='-DMKL_ILP64 -m64 -I${MKLROOT}/include'
#ENV export CPPFLAGS='-DMKL_ILP64 -m64 -I${MKLROOT}/include'
#ENV LINKFLAGS='-L${MKLROOT}/lib/intel64 -Wl,--no-as-needed -lmkl_intel_ilp64 -lmkl_gnu_thread -lmkl_core -lgomp -lpthread -lm -ldl'
#ENV LDFLAGS='-L${MKLROOT}/lib/intel64 -Wl,--no-as-needed -lmkl_intel_ilp64 -lmkl_gnu_thread -lmkl_core -lgomp -lpthread -lm -ldl'

# Setup and configure xmds2
WORKDIR /home
RUN wget https://excellmedia.dl.sourceforge.net/project/xmds/xmds-2.2.3.tar.gz && \
    tar -xvzf xmds-2.2.3.tar.gz && rm xmds-2.2.3.tar.gz
WORKDIR /home/xmds-2.2.3
RUN ./setup.py develop
# Compile with ATLAS
#RUN xmds2 --reconfigure --include-path /home/lib/fftw/include/ --include-path /home/lib/atlas/include/ --include-path /home/lib/openmpi/include/ --lib-path /home/lib/fftw/lib/ --lib-path /home/lib/atlas/lib --lib-path /home/lib/openmpi/lib
# Compile with MKL
#RUN /bin/bash -c 'source /opt/intel/bin/compilervars.sh intel64 && xmds2 --reconfigure --include-path /home/lib/fftw/include/ --include-path /home/lib/openmpi/include/ --lib-path /home/lib/fftw/lib/ --lib-path /home/lib/openmpi/lib'
RUN apt-get install libatlas-base-dev libblas-dev liblapack-dev -y
RUN /bin/bash -c 'xmds2 --reconfigure --include-path /home/lib/fftw/include/ --include-path /home/lib/openmpi/include/ --lib-path /home/lib/fftw/lib/ --lib-path /home/lib/openmpi/lib'

# Test XMDS
WORKDIR /home
RUN xmds2 ./xmds-2.2.3/examples/lorenz.xmds
RUN ./lorenz && rm lorenz*

# Update path, setup working dir
RUN mkdir -p /dataDir
WORKDIR /dataDir
RUN echo 'export PATH="/home/miniconda2/bin:$PATH"' >> /root/.bashrc
RUN echo 'export PATH="/home/lib/openmpi/bin:$PATH"' >> /root/.bashrc
RUN echo 'export PATH="/home/lib/fftw/bin:$PATH"' >> /root/.bashrc
