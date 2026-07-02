ARG BASE_IMAGE=quay.io/jupyter/pytorch-notebook:cuda12-python-3.13.14
#ARG BASE_IMAGE=quay.io/jupyter/base-notebook
FROM $BASE_IMAGE


RUN mamba install --yes 'flake8' && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

# Install from the requirements.txt file
COPY --chown=${NB_UID}:${NB_GID} mamba_requirements.txt /tmp/
RUN mamba install --yes --file /tmp/mamba_requirements.txt && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

RUN pip3 install dask_cuda
USER root
RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    wget \
    sudo \
    libssl-dev \
    libreadline-dev \
    zlib1g-dev \
    libbz2-dev \
    liblzma-dev \
    libpng-dev \
    libjpeg-dev \
    libtiff5-dev \
    libxt-dev \
    libtirpc-dev \
    libclang-dev \
    lsb-release \
    libstdc++6 \
    libc++-dev \
    libc++abi-dev \
    psmisc \
    gdebi && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

RUN wget https://download2.rstudio.org/server/jammy/amd64/rstudio-server-2026.01.1-403-amd64.deb && \
    gdebi -n rstudio-server-2026.01.1-403-amd64.deb && \
    rm -f rstudio-server-2026.01.1-403-amd64.deb
