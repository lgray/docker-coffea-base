FROM continuumio/miniconda3:latest as builder

RUN apt-get update \
     && apt-get install -yq --no-install-recommends build-essential autoconf automake libtool git g++ gfortran swig libmpfr-dev libboost-dev \
     && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN conda install --yes --freeze-installed -c conda-forge conda-build python==3.8.* \
     && conda clean --all -f -y \
     && conda build purge-all

ARG FASTJET_VERSION=3.3.4.0rc4

RUN git clone https://github.com/scikit-hep/fastjet.git -b v$FASTJET_VERSION --recursive \
     && cd fastjet \
     && pip install . -vv

RUN mkdir -p /artifacts \
    && cp -rp $(python -c 'import fastjet; import pathlib; print(pathlib.Path(fastjet.__file__).parent)')* /artifacts \
    && ls /artifacts

FROM continuumio/miniconda3:latest

RUN apt-get update \
     && apt-get install -yq --no-install-recommends libarchive-dev \
     && apt-get clean && rm -rf /var/lib/apt/lists/*

# XRootD + CA + VOMS + Coffea
RUN conda install --yes --freeze-installed \
      -c conda-forge \
      conda-build \
      voms \
      jupyterlab \
      dask_labextension \
      python==3.8.* \
      ca-policy-lcg \
      xrootd==5.2.0 \
      "uproot>=4.0.8" \
      coffea==0.7.5 \
      lz4 python-xxhash zstandard \
     && conda clean --all -f -y \
     && conda build purge-all \
     && pip install vector \
     && pip cache purge

# This is very delicate but it keeps the image size reasonable...
COPY --from=builder /artifacts /opt/conda/lib/python3.8/site-packages

# Make a symbolic link between installation /opt/conda/etc/grid-security and actual directory /etc/grid-security
RUN ln -s /opt/conda/etc/grid-security /etc/grid-security
