ARG FROM_IMAGE=nvidia/cuda
ARG CUDA_VER=10.2
ARG IMAGE_TYPE=devel
ARG LINUX_VER=ubuntu18.04
FROM ${FROM_IMAGE}:${CUDA_VER}-${IMAGE_TYPE}-${LINUX_VER}

# Pull argument from build args
ARG FULL_CUDA_VER
ARG LINUX_VER

# Define versions and download locations
ARG CONDA_VER=4.7.12
ARG TINI_VER=v0.18.0
ARG MINICONDA_URL=http://repo.anaconda.com/miniconda/Miniconda3-${CONDA_VER}-Linux-x86_64.sh
ARG TINI_URL=https://github.com/krallin/tini/releases/download/${TINI_VER}/tini

# Set environment
ENV PATH=/opt/conda/bin:${PATH}
ENV DEBIAN_FRONTEND=noninteractive

# Set CUDA_VERSION as in some 'nvidia/cuda' images this is not set
## A lot of scripts and conda recipes depend on this env var
ENV CUDA_VERSION=${FULL_CUDA_VER}

# Enables "source activate conda"
SHELL ["/bin/bash", "-c"]

# Update and add pkgs for Ubuntu
RUN if [ "${LINUX_VER}" != "centos7" ] ; then \
      apt-get update \
      && apt-get install -y wget bzip2 ca-certificates curl tzdata patch unzip automake autoconf git \
      && apt-get clean \
      && rm -rf /var/lib/apt/lists/* ; \
    else \
      echo -e "\n\n>>>> SKIPPING: LINUX_VER is not 'ubuntu16.04' or 'ubuntu18.04'\n\n"; \
    fi

# Update and add pkgs for centOS
RUN if [ "${LINUX_VER}" == "centos7" ] ; then \
      yum-config-manager --disable cuda \
      yum -y update \
      && yum remove -y bind-license \
      && yum -y install wget bzip2 ca-certificates curl which patch unzip make automake autoconf git \
      && yum clean all ; \
    else \
      echo -e "\n\n>>>> SKIPPING: LINUX_VER is not 'centos7'\n\n"; \
    fi

# Install miniconda and configure
RUN wget --quiet ${MINICONDA_URL} -O /miniconda.sh \
    && /bin/bash /miniconda.sh -b -p /opt/conda \
    && rm -f /miniconda.sh \
    && echo "conda ${CONDA_VER}" >> /opt/conda/conda-meta/pinned \
    && /opt/conda/bin/conda clean -afy \
    && ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh \
    && echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc \
    && echo "conda activate base" >> ~/.bashrc \
    && chmod -R ugo+w /opt/conda \
    && ln -s /opt/conda /conda

# Install tini for init
RUN wget --quiet ${TINI_URL} -O /usr/bin/tini \
    && chmod +x /usr/bin/tini

ENTRYPOINT [ "/usr/bin/tini", "--" ]
CMD [ "/bin/bash" ]
