ARG CUDA_VERSION=10.1
ARG CUDNN=7
FROM nvidia/cuda:$CUDA_VERSION-cudnn$CUDNN-devel-ubuntu18.04

RUN apt-get update -qq && apt-get install -y -q \
        build-essential \
        pkg-config \
        software-properties-common \
        curl \
        git \
        unzip \
        zlib1g-dev \
        locales \
    && apt-get clean -qq && rm -rf /var/lib/apt/lists/*

RUN locale-gen en_US.UTF-8
ENV LANG=en_US.UTF-8 LANGUAGE=en_US.en LC_ALL=en_US.UTF-8

ENV PYTHONPATH="${PYTHONPATH}:/src"
ENV PATH=/opt/conda/bin:$PATH
ENV PYTHONVERSION=3.6.9

# conda needs an untainted base environment to function properly
# that's why a new separate conda environment is created
RUN curl "https://repo.anaconda.com/miniconda/Miniconda3-4.7.12.1-Linux-x86_64.sh" --output ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p /opt/conda && \
    rm -rf ~/.cache ~/miniconda.sh

# split the conda installations because the dev boxes have limited memory
RUN /opt/conda/bin/conda create -n env -c conda-forge python=$PYTHONVERSION pip && \
    /opt/conda/bin/conda clean -a && \
    ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" > ~/.env && \
    echo "conda activate env" >> ~/.env && \
    echo "source ~/.env" >> ~/.bashrc

ENV BASH_ENV=~/.env
SHELL ["/bin/bash", "-c"]

COPY pkg/workloads/cortex/serve/requirements.txt /src/cortex/serve/requirements.txt
RUN pip install --no-cache-dir -r \
    /src/cortex/serve/requirements.txt

ARG SLIM="false"
RUN test "${SLIM}" = "true" || ( \
        # libsm6, libxext6, and libxrender-dev are required for opencv
        # libpng-dev is required for some image processing libaries
        # libsndfile1 is required for some audio processing libaries
        # libgl1-mesa-glx to libxtst6 are for qt (for GUI packages): https://docs.anaconda.com/anaconda/install/linux/
        # libnvinfer is required for TensorRT
        apt-get update -qq && apt-get install -y -q \
            libsm6 \
            libxext6 \
            libxrender-dev \
            libpng-dev \
            libsndfile1 \
            libgl1-mesa-glx \
            libegl1-mesa \
            libxrandr2 \
            libxss1 \
            libxcursor1 \
            libxcomposite1 \
            libasound2 \
            libxi6 \
            libxtst6 \
            libnvinfer6=6.0.1-1+cuda10.1 \
            libnvinfer-plugin6=6.0.1-1+cuda10.1 \
        && apt-get clean -qq && rm -rf /var/lib/apt/lists/* \
        && pip install --no-cache-dir --find-links https://download.pytorch.org/whl/torch_stable.html \
            cloudpickle==1.6.0 \
            Cython==0.29.21 \
            joblib==0.16.0 \
            Keras==2.4.3 \
            nltk==3.5 \
            np-utils==0.5.12.1 \
            opencv-python==4.4.0.42 \
            pandas==1.1.1 \
            Pillow==7.2.0 \
            scikit-image==0.17.2 \
            scikit-learn==0.23.2 \
            scipy==1.5.2 \
            six==1.15.0 \
            statsmodels==0.12.0 \
            sympy==1.6.2 \
            tensorflow-gpu==2.3.0 \
            tensorflow-hub==0.9.0 \
            torch==1.6.0+cu101 \
            torchvision==0.7.0+cu101 \
            xgboost==1.2.0 \
    )

COPY pkg/workloads/cortex/consts.py /src/cortex
COPY pkg/workloads/cortex/lib /src/cortex/lib
COPY pkg/workloads/cortex/serve /src/cortex/serve

ENTRYPOINT ["/src/cortex/serve/run.sh"]
