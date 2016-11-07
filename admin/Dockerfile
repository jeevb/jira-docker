FROM postgres:9.4

# Install required packages
RUN \
  apt-get update && \
  apt-get -y upgrade && \
  apt-get install -y \
    vim \
    emacs24 \
    nano \
    s3cmd && \
  apt-get autoremove -y && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/*

CMD ['/bin/bash']
