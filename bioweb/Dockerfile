FROM ubuntu:16.04

# Install dependencies
RUN apt-get update && apt-get install -y \
        bsdtar \
        curl \
        g++ \
        git \
        libpq-dev \
        lighttpd \
        mercurial \
        nodejs \
        nodejs-legacy \
        npm \
        postgresql \
        python3-dev \
        python3-pip \
        scons \
        sudo \
    && rm -rf /var/lib/apt/lists/*

# Add a user
RUN groupadd -r developer \
    && useradd \
        --gid developer \
        --create-home \
        --no-log-init \
        --groups sudo \
        --key MAIL_DIR=/dev/null \
        --shell /bin/bash \
        developer \
    && passwd -d developer

WORKDIR /opt

# Download boost
ARG boost_url="https://dl.bintray.com/boostorg/release/1.63.0/source/boost_1_63_0.tar.gz"

RUN mkdir boost \
    && curl \
        --location \
        $boost_url \
    | tar \
        --directory boost \
        --extract \
        --gzip \
        --strip-components 1

WORKDIR /opt/boost

# Build and install boost
ARG boost_build_threads=1

RUN ./bootstrap.sh \
        --prefix=/usr/local/ \
        --with-python=$(which python3) \
    && ./b2 \
        -j$boost_build_threads \
    && ./b2 install

ENV LD_LIBRARY_PATH=/usr/local/lib

WORKDIR /opt

# Remove boost sources
RUN rm -rf boost

# Download mt4cpp
ARG mt4cpp_url="https://downloads.sourceforge.net/project/mt4cpp/0.03/mt4cpp_0.3.76-1.tar.gz"

RUN mkdir mt4cpp \
    && curl \
        --location \
        $mt4cpp_url \
    | tar \
        --directory mt4cpp \
        --extract \
        --gzip \
        --strip-components 1

# Install mt4cpp
RUN mv mt4cpp /usr/local/include

USER developer
WORKDIR /home/developer

# Download bioweb
ARG bioweb_url="https://downloads.sourceforge.net/project/bioweb/0.09/bioweb_0.09.105_1_django_angular.zip"

RUN mkdir bioweb \
    && curl \
        --location \
        $bioweb_url \
    | bsdtar \
        --directory bioweb \
        --extract \
        --file -

# Install Python dependencies
ENV PATH="/home/developer/.local/bin:$PATH"

RUN pip3 install \
        --user \
        -r ~/bioweb/pip_dev.requirements

# Install Node dependencies
ENV PATH="/home/developer/.npm-global/bin:$PATH"
ENV NPM_CONFIG_PREFIX="/home/developer/.npm-global"
RUN mkdir ~/.npm-global \
    && npm install -g bower

# Fix and install local Node dependencies
WORKDIR /home/developer/bioweb/client

RUN sed -i 's/"angular": "",/"angular": "1.4.3",/g' bower.json \
    && sed -i 's/"angular-resource" : "",/"angular-resource" : "1.4.3",/g' bower.json \
    && sed -i 's/"angular-route" : "",/"angular-route" : "1.4.3",/g' bower.json \
    && sed -i 's/"angular-translate" : "",/"angular-translate" : "2.18.1",/g' bower.json \
    && sed -i 's/"angular-translate-loader-static-files" : "",/"angular-translate-loader-static-files" : "2.18.1",/g' bower.json \
    && sed -i 's/"jquery": "",/"jquery": "3.4.1",/g' bower.json \
    && sed -i 's/"qunit": "",/"qunit": "2.9.2",/g' bower.json \
    && sed -i 's/"blanket": ""/"blanket": "1.1.7"/g' bower.json \
    && ~/.npm-global/bin/bower install

# Fix python executable name in SConstruct
WORKDIR /home/developer/bioweb

RUN sed -i "s/'python /'python3 /g" SConstruct

# Build the bioweb project
RUN cd ~/bioweb \
    && scons

# Configure the database
RUN sudo sed -i 's/local   all             all                                     peer/local   all             all                                     md5/g' /etc/postgresql/9.5/main/pg_hba.conf \
    && sudo service postgresql start \
    && sudo -u postgres createuser --no-superuser --createdb --no-createrole mydb \
    && sudo -u postgres createdb -O mydb mydb \
    && sudo -u postgres psql -c "alter user mydb with encrypted password 'mydb';" \
    && cd ~/bioweb/build_web \
    && python3 manage.py migrate
    
RUN printf '#!/bin/bash\n\nset -ex\nsudo service postgresql start\nscons $*' > /home/developer/entrypoint \
    && chmod +x /home/developer/entrypoint

EXPOSE 9000

ENTRYPOINT ["/home/developer/entrypoint"]
CMD ["-h"]
