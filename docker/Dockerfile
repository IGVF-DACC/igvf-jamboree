# Download base image ubuntu 22.04
FROM continuumio/miniconda3:latest

# LABEL about the custom image
LABEL maintainer="bioinfolucas@gmail"
LABEL version="0.1"
LABEL description="First test"

# Update Ubuntu Software repository
RUN conda install -c conda-forge mamba -y
RUN pip install nextflow
RUN pip uninstall GTFProcessing -y
RUN pip install git+https://github.com/LucasSilvaFerreira/GTFProcessing.git
RUN pip install gtfparse==1.3.0
RUN pip install git+https://github.com/LucasSilvaFerreira/Perturb_Loader.git
RUN mamba install -c bioconda nextflow -y
RUN mamba install -c bioconda kallisto -y
RUN pip install --quiet kb-python
RUN mamba install -c anaconda openpyxl -y
RUN mamba install -c conda-forge r-base -y
RUN Rscript -e 'install.packages("BiocManager", repos = "http://cran.us.r-project.org")'
RUN Rscript -e 'BiocManager::install("Rhdf5lib")'
RUN Rscript -e 'BiocManager::install("rhdf5")'
RUN Rscript -e 'install.packages("doParallel", repos = "http://cran.us.r-project.org")'
RUN Rscript -e 'install.packages("devtools", repos = "http://cran.us.r-project.org")'
RUN mamba install -c conda-forge r-gert -y
RUN mamba install -c conda-forge r-ragg -y
RUN Rscript -e 'install.packages("devtools", repos = "http://cran.us.r-project.org")'
RUN mamba install -c conda-forge r-ggplot2
RUN Rscript -e 'devtools::install_github("katsevich-lab/sceptre")'
RUN Rscript -e 'BiocManager::install("ShortRead")'
RUN Rscript -e 'devtools::install_github("chris-mcginnis-ucsf/MULTI-seq")'
RUN mamba install -c bioconda ucsc-blat -y
EXPOSE 80 443 8080

