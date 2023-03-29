ARG PARENT_DOCKER

ARG NB_USER="jovyan"
ARG NB_UID=1000
ARG NB_GID=100

FROM "$PARENT_DOCKER"

ENV DEBIAN_FRONTEND noninteractive

USER root
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
    software-properties-common \
    apt-transport-https \
    python3-pip && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen


########### custom software installation

# Update Ubuntu Software repository
RUN pip install nextflow
RUN pip uninstall GTFProcessing -y
RUN pip install git+https://github.com/LucasSilvaFerreira/GTFProcessing.git
RUN pip install gtfparse==1.3.0
RUN pip install git+https://github.com/LucasSilvaFerreira/Perturb_Loader.git
RUN pip install --quiet kb-python
RUN conda install -c conda-forge mamba -y
RUN mamba install -c bioconda nextflow -y
RUN mamba install -c bioconda kallisto -y
RUN mamba install -c anaconda openpyxl -y
RUN mamba install -c conda-forge r-base -y
RUN mamba install -c conda-forge r-gert -y
RUN mamba install -c conda-forge r-ragg -y
RUN mamba install -c conda-forge r-ggplot2
RUN Rscript -e 'install.packages("BiocManager", repos = "http://cran.us.r-project.org")'
RUN Rscript -e 'BiocManager::install("Rhdf5lib")'
RUN Rscript -e 'BiocManager::install("rhdf5")'
RUN Rscript -e 'BiocManager::install("ShortRead")'
RUN Rscript -e 'install.packages("doParallel", repos = "http://cran.us.r-project.org")'
RUN Rscript -e 'install.packages("devtools", repos = "http://cran.us.r-project.org")'
RUN Rscript -e 'devtools::install_github("katsevich-lab/sceptre")'
RUN Rscript -e 'devtools::install_github("chris-mcginnis-ucsf/MULTI-seq")'
RUN pip install muon
RUN pip install deeptools

RUN apt-get update -y && apt-get install -y curl gpg-agent
RUN echo "deb https://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
RUN curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
RUN apt-get update -y && apt-get install -y google-cloud-sdk

################################################


ENV HOME /home/$NB_USER

RUN echo 'root:jamboree'|chpasswd && \
    echo 'jovyan:jamboree'|chpasswd && \
    sed -i.bak -e 's/#%admin/%admin/' /etc/sudoers && \
    sed -i.bak -e 's/#%sudo/%sudo/' /etc/sudoers && \
    usermod -aG sudo jovyan && \
    chmod 0440 /etc/sudoers && \
    chmod -R 755 /home/$NB_USER
