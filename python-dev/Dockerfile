FROM ubuntu:18.04
LABEL maintainer="Benjmain Mort <ben.mort@gmail.com>"

ENV DEBIAN_FRONTEND=noninteractive
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

# Install tango development packages
RUN apt-get update && apt-get install -y \
        pkg-config \
        libboost-python-dev \
        python3 \
        python3-pip \
        python3-pytango \
        python3-itango \
        less \
    && rm -rf /var/lib/apt/lists/*

# Create unprivaliged ska user
RUN useradd -ms /bin/bash ska
USER ska
WORKDIR /home/ska

ENTRYPOINT ["/bin/bash"]
CMD []
