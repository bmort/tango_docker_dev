FROM ubuntu:18.04
LABEL maintainer="Benjmain Mort <ben.mort@gmail.com>"

ENV DEBIAN_FRONTEND=noninteractive
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

# Install tango development packages
RUN apt-get update && \
    apt-get install -y \
        python3 \
        python3-pip \
        python3-pytango \
        libboost-python-dev \
        libtango-dev \
        pkg-config \
        wget \
        openjdk-8-jdk \
        liblog4j1.2-java \
        less

RUN pip3 install -U pip pipenv itango

# Jive installation
RUN wget -q -c https://people.debian.org/~picca/libtango-java_9.2.5a-1_all.deb
RUN dpkg -i ./libtango-java_9.2.5a-1_all.deb

# Create unprivaliged ska user
RUN useradd -ms /bin/bash ska
USER ska
WORKDIR /home/ska

# initialise ipython profile (needed for itango & ipython)
RUN ipython profile create

# Create pipenv installation with pytango and itango
# FIXME(BMo): Installing pytango takes a LONG time. \
#             consider moving this to another layer?
#             http://bit.ly/2FZLV0w
#RUN pipenv install pytango itango

ENTRYPOINT ["/bin/bash"]
CMD []
