# Use NVIDIA's CUDA base image as the foundation
FROM --platform=linux/amd64 quay.io/jupyter/pytorch-notebook@sha256:b09fff154a0b8dfcf9641e669f32bb4fe0f301b6be5da3fa18ecafeb06a23876

# Set user arguments from second image
ARG NB_USER="jovyan"
ARG NB_UID=1000
ARG NB_GID=100

# Combine environment variables from both images
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility
ENV PATH /usr/local/nvidia/bin:/usr/local/cuda/bin:$PATH
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib:/usr/local/nvidia/lib64:$LD_LIBRARY_PATH
ENV DEBIAN_FRONTEND noninteractive
ENV MICROMAMBA_ROOT_PREFIX="/opt/micromamba"

# Combine system package installations
USER root
RUN mkdir -p /opt/micromamba /data && \
    chmod -R 777 /opt && \
    chmod 775 /data && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    libhdf5-dev \
    gcc \
    g++ \
    libc6-dev \
    python3-dev \
    pkg-config \
    ca-certificates \
    fonts-liberation \
    locales \
    sudo \
    wget \
    build-essential \
    git \
    curl \
    ssh \
    vim \
    ncurses-dev \
    software-properties-common \
    apt-transport-https \
    python3-pip \
    gnupg2 \
    dirmngr \
    libxml2-dev \
    libssl-dev \
    zlib1g-dev \
    libgit2-dev \
    libtbb-dev \
    libcurl4-openssl-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# Install micromamba and configure
RUN conda install -y conda-forge::micromamba && \
    micromamba config append channels conda-forge && \
    micromamba config append channels bioconda && \
    micromamba config append channels defaults

# Install Python and R through micromamba
RUN micromamba install -r ${MICROMAMBA_ROOT_PREFIX} -n base -y \
    python=3.10.14 \
    r-base=4.4.0 \
    nextflow \
    numpy=1.26.4 \
    pandas=2.2.2 \
    pip=24.2 \
    kallisto=0.50.1 \
    bustools=0.44.0 \
    anndata=0.10.9 \
    gmm-demux=0.2.2.3 \
    scanpy=1.10.4 \
    matplotlib=3.10.0 \
    ipykernel \
    muon=0.1.6 \
    mudata=0.3.1 \
    seaborn=0.13.2 \
    scrublet=0.2.3 \
    networkx=3.4.2 \
    umap-learn=0.5.7 \
    cmdstanpy=1.2.5 \
    scikit-learn=1.6.1 \
    gtfparse=2.5.0

# Install h5py with specific build
RUN pip install --upgrade pip && \
    pip install setuptools==72.2.0 && \
    pip install --no-binary=h5py h5py==3.11.0

# Install remaining Python packages
RUN pip install argparse openpyxl seqspec GMM_Demux adjustText igraph && \
    pip install git+https://github.com/LucasSilvaFerreira/GTFProcessing.git && \
    pip install --no-deps --no-cache-dir "git+https://github.com/jiangsizhu1201/CLEANSER"

# R environment setup
COPY renv.lock /tmp/renv.lock
RUN micromamba run -r ${MICROMAMBA_ROOT_PREFIX} -n base Rscript -e \
    'install.packages("renv", repos = "http://cran.us.r-project.org")' && \
    micromamba run -r ${MICROMAMBA_ROOT_PREFIX} -n base Rscript -e \
    'renv::restore(lockfile = "/tmp/renv.lock")'

# Install perturbo package
WORKDIR /app
COPY perturbo-0.0.1-py3-none-any.whl /app/
RUN pip install perturbo-0.0.1-py3-none-any.whl

# User permissions and final setup
RUN echo 'root:jamboree' | chpasswd && \
    echo 'jovyan:jamboree' | chpasswd && \
    sed -i.bak -e 's/#%admin/%admin/' /etc/sudoers && \
    sed -i.bak -e 's/#%sudo/%sudo/' /etc/sudoers && \
    usermod -aG sudo jovyan && \
    chmod 0440 /etc/sudoers && \
    chmod -R 755 /home/$NB_USER

ENV HOME /home/$NB_USER
WORKDIR $HOME
