FROM ubuntu:24.04

ENV LISTEN_PORT=5000
EXPOSE 5000

ARG DEBIAN_FRONTEND=noninteractive
# Use BuildKit's built-in ARGs
ARG TARGETARCH
ARG TARGETPLATFORM

RUN apt-get -y update && apt-get install -y \
    build-essential \
    autotools-dev \
    automake \
    autoconf \
    libbz2-dev \
    libboost-all-dev \
    # python3.9 \
    # python3-pip \
    sqlite3 \
    wget \
    curl \
    gzip \
    libssl-dev \
    libio-socket-ssl-perl \
    cpanminus \
    libxml-simple-perl \ 
    libwww-perl \
    libnet-perl \
    golang \
    checkinstall \
    libreadline-dev \
    libncursesw5-dev \
    libsqlite3-dev \
    tk-dev \
    libgdbm-dev \
    libc6-dev \
    dpkg-dev \
    # python-dev \
    python3-dev \
    libncursesw5-dev \
    libffi-dev \
    zlib1g-dev \
    libjpeg-dev \
    libtiff-dev \
    libpq-dev \
    libxml2-dev \
    libxslt1-dev \
    libsdl2-dev \
    libgstreamer-plugins-base1.0-dev \
    libnotify-dev \
    freeglut3-dev \
    libsm-dev \
    libgtk-3-dev \
    libxtst-dev


WORKDIR /app

RUN wget https://www.python.org/ftp/python/3.13.2/Python-3.13.2.tgz ;\
    tar -xzvf Python-3.13.2.tgz
WORKDIR /app/Python-3.13.2
RUN ./configure --enable-optimizations --with-lto ;\
    make ;\
    make test ;\
    make install

WORKDIR /app

RUN pip3.13 install flask gemmi biopython rich dataclasses-json scipy numpy seaborn


# Get blast
RUN set -eux; \
    if [ "$TARGETPLATFORM" = "linux/amd64" ]; then \
        echo "Installing x64 binaries" ; \
        wget "https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.16.0/ncbi-blast-2.16.0+-x64-linux.tar.gz"; \
        tar -zxvf ncbi-blast-2.16.0+-x64-linux.tar.gz; \
    elif [ "$TARGETPLATFORM" = "linux/arm64" ]; then \
        echo "Installing ARM64 binaries" ; \
        wget "https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.16.0/ncbi-blast-2.16.0+-aarch64-linux.tar.gz"; \
        tar -zxvf ncbi-blast-2.16.0+-aarch64-linux.tar.gz; \
    else \
        echo "Unsupported architecture: $TARGETPLATFORM" ; \
        exit 1 ; \
    fi


# Get blast
# RUN wget "https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/ncbi-blast-2.16.0+-x64-linux.tar.gz" ;\
#     tar -zxvf ncbi-blast-2.16.0+-x64-linux.tar.gz


# RUN wget "https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.16.0/ncbi-blast-2.16.0+-src.tar.gz" ;\
#     tar -zxvf ncbi-blast-2.16.0+-src.tar.gz
# WORKDIR /app/ncbi-blast-2.16.0+-src/c++
# RUN ./configure ;\
#     make ;\
#     make install
# WORKDIR /app 
# RUN rm -r ncbi-blast-2.16.0+-src.tar.gz

# build DSSP
RUN wget "https://github.com/cmbi/dssp/archive/refs/tags/2.3.0.tar.gz" ;\
    tar -zxvf 2.3.0.tar.gz
WORKDIR /app/dssp-2.3.0
RUN ./autogen.sh ;\
    ./configure CXXFLAGS="-std=c++14" ;\
    make ;\
    make install
WORKDIR /app
RUN rm -r dssp-2.3.0 2.3.0.tar.gz

# build TMalign
RUN wget "https://zhanglab.ccmb.med.umich.edu/TM-align/TMalign.f" ;\
    gfortran -static -O3 -ffast-math -lm -o TMalign TMalign.f ;\
    mv TMalign /usr/local/bin ;\
    rm TMalign.f

# get mysql2sqlite converter to convert IEDB's mySQL DB
RUN wget "https://raw.githubusercontent.com/dumblob/mysql2sqlite/d14d22ad7029cdf4d11825ee3c96922e8fbb0122/mysql2sqlite" ;\
    chmod +x mysql2sqlite ;\
    mv mysql2sqlite /usr/local/bin

# get mmseqs2

RUN set -eux; \
    if [ "$TARGETPLATFORM" = "linux/amd64" ]; then \
        echo "Installing x64 binaries" ; \
        wget "https://mmseqs.com/latest/mmseqs-linux-avx2.tar.gz"; \
        tar -zxvf mmseqs-linux-avx2.tar.gz; \
    elif [ "$TARGETPLATFORM" = "linux/arm64" ]; then \
        echo "Installing ARM64 binaries" ; \
        wget "https://dev.mmseqs.com/latest/mmseqs-linux-arm64.tar.gz"; \
        tar -zxvf mmseqs-linux-arm64.tar.gz; \
    else \
        echo "Unsupported architecture: $TARGETPLATFORM" ; \
        exit 1 ; \
    fi
# RUN wget https://mmseqs.com/latest/mmseqs-linux-avx2.tar.gz ;\
#     tar xvfz mmseqs-linux-avx2.tar.gz ;\
#     rm mmseqs-linux-avx2.tar.gz


RUN curl -s "ftp://ftp.ncbi.nlm.nih.gov/entrez/entrezdirect/edirect.tar.gz" -o "edirect.tar.gz" ;\
    tar -xf edirect.tar.gz ;\
    rm edirect.tar.gz

WORKDIR /app/edirect

RUN wget ftp.ncbi.nlm.nih.gov/entrez/entrezdirect/xtract.Linux.gz ;\
    gunzip -f xtract.Linux.gz ;\
    chmod +x xtract.Linux;\
    wget ftp.ncbi.nlm.nih.gov/entrez/entrezdirect/transmute.Linux.gz ;\
    gunzip -f transmute.Linux.gz ;\
    chmod +x transmute.Linux ;\
    wget ftp.ncbi.nlm.nih.gov/entrez/entrezdirect/rchive.Linux.gz ;\
    gunzip -f rchive.Linux.gz ;\
    chmod +x rchive.Linux 

WORKDIR /app
COPY . /app


RUN python3.13 -m ensurepip && \
    python3.13 -m pip install --upgrade pip setuptools
RUN python3.13 setup.py build ;python3.13 setup.py install
# RUN python3.10 setup.py develop
# RUN chmod +x docker/run_epitopedia.py ; chmod +x docker/generate_database.py

ENV PATH=/app/edirect:/app/mmseqs/bin/:/app/ncbi-blast-2.16.0+/bin:/app/:$PATH

ENV FLASK_DEBUG=false
ENV FLASK_APP=/app/epitopedia/viz/serve.py


