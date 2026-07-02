
FROM xychelsea/anaconda3:v23.1.0-gpu
#FROM continuumio/anaconda3
LABEL description="Anaconda 3 GPU Container w/ Jupyter Notebooks"


ARG JUPYTER_LAB_VERSION="4.0.3"
ARG JUPYTER_HUB_VERSION="4.0.1"
ARG JUPYTER_NB_CK_VERSION="2.3.1"
ARG JUPYTER_IPYWIDGETS_VERSION="8.0.7"
ARG JUPYTER_IPYKERNEL="6.24.0"
ARG CONDA_OVERRIDE_CUDA="12.9"

ENV JUPYTER_LAB_VERSION="${JUPYTER_LAB_VERSION}"
ENV JUPYTER_NB_CK_VERSION="${JUPYTER_NB_CK_VERSION}"
ENV JUPYTER_IPYWIDGETS_VERSION="${JUPYTER_IPYWIDGETS_VERSION}"
ENV JUPYTER_IPYKERNEL="${JUPYTER_IPYKERNEL}"

ENV LD_LIBRARY_PATH=/usr/local/anaconda3/lib/:$LD_LIBRARY_PATH

USER root
# R pre-requisites

RUN apt-get update --yes && \
    apt-get install --yes --no-install-recommends \
    gdebi \
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
    wget && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

RUN wget https://download2.rstudio.org/server/jammy/amd64/rstudio-server-2026.01.1-403-amd64.deb && \
    gdebi -n rstudio-server-2026.01.1-403-amd64.deb && \
    rm -f rstudio-server-2026.01.1-403-amd64.deb

# Switch to user "anaconda"
USER ${ANACONDA_UID}
WORKDIR ${HOME}

# Update Anaconda
RUN conda update -c defaults conda && \
    conda config --add channels conda-forge  && \
    conda config --set channel_priority strict && \
    conda config --append channels nvidia
RUN pip install "setuptools<81"
RUN conda install -c conda-forge ncurses --force-reinstall
# Install Jupyter Notebook
RUN conda install --y -c base -c conda-forge -c nvidia \ 
    jupyterlab=${JUPYTER_LAB_VERSION} \
    jupyterhub=${JUPYTER_HUB_VERSION} \
    nb_conda_kernels=${JUPYTER_NB_CK_VERSION} \
    ipywidgets=${JUPYTER_IPYWIDGETS_VERSION} \
    jupyterlab-nvdashboard \
    mamba \
    cudatoolkit \
    #cudnn \
    #cuda \
    #tensorflow-gpu=2.19.1 \
    ipykernel=${JUPYTER_IPYKERNEL}

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
    'dask==2026.3.0' \
    'rpy2' \
    'jupyterhub' \
    'jupyter-server-proxy' \
    'jupyter-rsession-proxy' \
    'jupyter-vscode-proxy' \
    'pycuda' \
    'jupyterlab-nvdashboard' \
    'code-server' \
    'dask-labextension>=5' \
    'cupy' \
    'unixodbc' && \
     mamba clean --all -f -y

RUN pip3 install dask-cuda

RUN jupyter lab --generate-config
USER root
RUN chown -R ${ANACONDA_UID}:users /home/anaconda 
USER ${ANACONDA_UID}
# Clean Anaconda
RUN conda clean -afy \
    && fix-permissions ${HOME} \
    && fix-permissions ${ANACONDA_PATH}

EXPOSE 8888

# Configure container startup
ENTRYPOINT [ "tini", "-g", "--" ]
CMD [ "jupyter", "lab" ]

# Currently need to have both jupyter_notebook_config and jupyter_server_config to support classic and lab
COPY scripts/jupyter_server_config.py scripts/docker_healthcheck.py /etc/jupyter/

# Fix permissions on /etc/jupyter as root
USER root

# Legacy requirements for jupyter notebooks
RUN sed -re "s/c.ServerApp/c.NotebookApp/g" \
    /etc/jupyter/jupyter_server_config.py > /etc/jupyter/jupyter_notebook_config.py \
    && fix-permissions /etc/jupyter/

# Re-activate user "anaconda"
USER $ANACONDA_UID
WORKDIR $HOME
