ARG REGISTRY=quay.io
ARG OWNER=jupyter
ARG BASE_IMAGE=$REGISTRY/$OWNER/scipy-notebook
FROM $BASE_IMAGE

LABEL maintainer="Nathan LeSueur <nathanle@akamai.com>"

# Fix: https://github.com/hadolint/hadolint/wiki/DL4006
# Fix: https://github.com/koalaman/shellcheck/wiki/SC3014
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

# R pre-requisites
RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    fonts-dejavu \
    gfortran \
    wget \
    gdebi \
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
    psmisc \
    gcc && \
    wget https://download2.rstudio.org/server/jammy/amd64/rstudio-server-2026.01.1-403-amd64.deb && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

#Rstudio install and remove
RUN gdebi -n rstudio-server-2026.01.1-403-amd64.deb && \
    rm -f rstudio-server-2026.01.1-403-amd64.deb 

# macOS Rosetta virtualization creates junk directory which gets owned by root further up.
# It'll get re-created, but as USER runner after the next directive so hopefully should not cause permission issues.
#
# More info: https://github.com/jupyter/docker-stacks/issues/2296
# hadolint ignore=DL3059
RUN rm -rf "/home/${NB_USER}/.cache/"

USER ${NB_UID}

# R packages including IRKernel which gets installed globally.
# r-e1071: dependency of the caret R package
# Proxies for code-server and Rstudio
RUN mamba install --yes \
    'r-base' \
    'r-caret' \
    'r-crayon' \
    'r-devtools' \
    'r-e1071' \
    'r-forecast' \
    'r-hexbin' \
    'r-htmltools' \
    'r-htmlwidgets' \
    'r-irkernel' \
    'r-nycflights13' \
    'r-randomforest' \
    'r-rcurl' \
    'r-rmarkdown' \
    'r-rodbc' \
    'r-rsqlite' \
    'r-shiny' \
    'r-tidymodels' \
    'r-tidyverse' \
    'rpy2' \
    'jupyter-server-proxy' \
    'jupyter-rsession-proxy' \
    'jupyter-vscode-proxy' \
    'code-server' \
    'unixodbc' && \
    mamba clean --all -f -y && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

