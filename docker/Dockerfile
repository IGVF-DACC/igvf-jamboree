#ARG PARENT_DOCKER

ARG NB_USER="jovyan"
ARG NB_UID=1000
ARG NB_GID=100

FROM jupyter/base-notebook@sha256:0ec801994602a53d3917d0f07ccf4474a1ddf5ae419483b24681a3c42abc4ce5

ENV DEBIAN_FRONTEND noninteractive

ENV MICROMAMBA_ROOT_PREFIX="/opt/micromamba"

USER root
RUN mkdir /opt/micromamba
RUN chmod -R 777 /opt && mkdir /data && chmod 775 /data
RUN APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=1 && \
    apt-get update && \
    apt-get install --yes --no-install-recommends \
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
    python3-pip && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen


########### custom software installation
RUN conda install conda-forge::micromamba
# Add necessary channels in the correct order of priority
RUN micromamba config append channels conda-forge && \
    micromamba config append channels bioconda && \
    micromamba config append channels defaults

#Install Python-related conda dependencies (including R base)
RUN micromamba install -r ${MICROMAMBA_ROOT_PREFIX} -n base -y \
    python=3.10.14 \
    r-base=4.4.0 

   
# Install system-level dependencies
USER root

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    gnupg2 \
    dirmngr \
    libxml2-dev \
    libssl-dev \
    zlib1g-dev \
    libgit2-dev \
    libtbb-dev  \
    libcurl4-openssl-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*


# # R setup for installing renv and restoring R environment from lock file
COPY renv.lock /tmp/renv.lock
RUN micromamba run -r ${MICROMAMBA_ROOT_PREFIX} -n base Rscript -e 'install.packages("renv", repos = "http://cran.us.r-project.org")' 

RUN micromamba run -r ${MICROMAMBA_ROOT_PREFIX} -n base Rscript -e 'renv::restore(lockfile = "/tmp/renv.lock")'

# # Install additional pip dependencies
RUN pip install openpyxl seqspec GMM_Demux adjustText igraph && \
    pip install git+https://github.com/LucasSilvaFerreira/GTFProcessing.git

    #Install Python-related conda dependencies (including R base)
RUN micromamba install -r ${MICROMAMBA_ROOT_PREFIX} -n base -y \
    nextflow \
    numpy=1.26.4 \
    pandas=2.2.2 \
    pip=24.2 \
    kallisto=0.50.1 \
    bustools=0.44.0 \
    anndata=0.10.9 \
    scanpy \
    matplotlib \
    ipykernel \
    muon \
    mudata \
    seaborn \
    scrublet \
    networkx \
    umap-learn \
    cmdstanpy

# Install CLEANSER and its dependencies
RUN micromamba run -r ${MICROMAMBA_ROOT_PREFIX} -n base pip install kb-python==0.28.2
RUN micromamba run -r ${MICROMAMBA_ROOT_PREFIX} -n base pip install --no-deps --no-cache-dir "git+https://github.com/jiangsizhu1201/CLEANSER" 

ENV PATH="$MICROMAMBA_ROOT_PREFIX/bin:$PATH"

ENV HOME /home/$NB_USER

RUN echo 'root:jamboree'|chpasswd && \
    echo 'jovyan:jamboree'|chpasswd && \
    sed -i.bak -e 's/#%admin/%admin/' /etc/sudoers && \
    sed -i.bak -e 's/#%sudo/%sudo/' /etc/sudoers && \
    usermod -aG sudo jovyan && \
    chmod 0440 /etc/sudoers && \
    chmod -R 755 /home/$NB_USER