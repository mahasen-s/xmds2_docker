FROM ubuntu:18.04
RUN apt-get update && apt-get install -y build-essential \
                    libopenmpi-dev \
                    openmpi-bin \
                    wget \
                    vim \
                    libhdf5-dev \
                    libfftw3-dev \
                    libatlas-base-dev \
                    liblapack-dev \
                    libblas-dev \
                    libopenmpi-dev

WORKDIR /home
# Install miniconda and necessary python packages
RUN wget https://repo.continuum.io/miniconda/Miniconda2-latest-Linux-x86_64.sh && \
    bash Miniconda2-latest-Linux-x86_64.sh -b -p /home/miniconda2 && \
    rm Miniconda2-latest-Linux-x86_64.sh
ENV PATH="/home/miniconda2/bin:${PATH}"
RUN conda install setuptools cheetah numpy pyparsing lxml mpmath sphinx h5py -y

# Get and configure xmds2
RUN wget https://excellmedia.dl.sourceforge.net/project/xmds/xmds-2.2.3.tar.gz && \
    tar -xvzf xmds-2.2.3.tar.gz && rm xmds-2.2.3.tar.gz
WORKDIR /home/xmds-2.2.3
RUN ./setup.py develop
RUN xmds2 --reconfigure

# Test
WORKDIR /home
RUN xmds2 ./xmds-2.2.3/examples/lorenz.xmds
RUN ./lorenz && rm lorenz* 

# Update path
RUN mkdir -p /dataDir
WORKDIR /dataDir
RUN echo 'export PATH="/home/miniconda2/bin:$PATH"' >> /root/.bashrc 
