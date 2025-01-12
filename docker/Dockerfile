# Base image must at least have pytorch and CUDA installed.
# We are using NVIDIA NGC's PyTorch image here, see: https://ngc.nvidia.com/catalog/containers/nvidia:pytorch for latest version
ARG BASE_IMAGE=nvcr.io/nvidia/pytorch:19.09-py3
# ARG BASE_IMAGE=nvcr.io/nvidia/pytorch:20.01-py3
FROM $BASE_IMAGE
ARG BASE_IMAGE

# Installing NVIDIA Apex for distributed computing
RUN echo "Installing Apex on top of ${BASE_IMAGE}"
# make sure we don't overwrite some existing directory called "apex"
WORKDIR /tmp/unique_for_apex
# uninstall Apex if present, twice to make absolutely sure :)
RUN pip uninstall -y apex || :
RUN pip uninstall -y apex || :
# SHA is something the user can touch to force recreation of this Docker layer,
# and therefore force cloning of the latest version of Apex
RUN SHA=ToUcHMe git clone https://github.com/NVIDIA/apex.git
WORKDIR /tmp/unique_for_apex/apex
RUN pip install -v --no-cache-dir --global-option="--cpp_ext" --global-option="--cuda_ext" .

# See http://bugs.python.org/issue19846
ENV LANG C.UTF-8

# Install additional programs
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    htop vim tmux sudo \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Update pip
# SHA is something the user can touch to force recreation of this Docker layer,
RUN SHA=ToUcHMe1 which python3
RUN SHA=ToUcHMe2 python3 -m pip install --upgrade pip

# Set path to CUDA
ENV CUDA_HOME=/usr/local/cuda

# Install dependencies
RUN python3 -m pip install transformers
RUN python3 -m pip install numpy
RUN python3 -m pip install -U spacy
RUN python3 -m spacy download en_core_web_sm
RUN python3 -m spacy download de_core_news_sm
RUN python3 -m pip install matplotlib
RUN python3 -m pip install pandas
RUN python3 -m pip install scipy
RUN python3 -m pip install scikit-learn
RUN python3 -m pip install seaborn
RUN python3 -m pip install tqdm

# Install additional stuff
RUN python3 -m pip install tensorboardX
RUN python3 -m pip install tensorboard
RUN python3 -m pip install wandb
RUN python3 -m pip install jsonlines
RUN python3 -m pip install pytorch_revgrad

# Install additional stuff (for VS Code dev)
RUN python3 -m pip install autopep8
RUN conda install pylint

# Specify a new user (USER_NAME and USER_UID are specified via --build-arg)
ARG USER_UID
ARG USER_NAME
ENV USER_GID=$USER_UID
ENV USER_GROUP="users"

# Create the user
RUN mkdir /home/$USER_NAME
RUN useradd -l -d /home/$USER_NAME -u $USER_UID -g $USER_GROUP $USER_NAME
RUN mkdir /home/$USER_NAME/.vscode-server
RUN mkdir /home/$USER_NAME/.vscode-server-insiders

# Change owner of home dir
RUN chown -R ${USER_UID}:${USER_GID} /home/$USER_NAME/

# Set workdir when starting container
WORKDIR /deep_crispr

# Add workdir to PYTHONPATH
RUN export PYTHONPATH="$PYTHONPATH:/deep_crispr"

# If you have git submodules, add their path to PYTHONPATH
RUN export PYTHONPATH="$PYTHONPATH:/deep_crispr/spacyface"
RUN export PYTHONPATH="$PYTHONPATH:/deep_crispr/structural-probes"