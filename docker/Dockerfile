FROM openjdk:11.0.8-jre-slim-buster
ENV ZK_USER=zookeeper \
ZK_DATA_DIR=/var/lib/zookeeper/data \
ZK_DATA_LOG_DIR=/var/lib/zookeeper/log \
ZK_LOG_DIR=/var/log/zookeeper \
ZK_DOWNLOAD_URL=https://downloads.apache.org/zookeeper \
APACHE_KEYS="68E327C1 6E1CC7A4 8EE2F25C D2C80E32 820F225C 92BC2F2B ADAFD097 \
  B5E9BD2D 767E7473 41CF31A9 173C31A2 15DFA1BA A615D22C 3B5B5147 8F6E35FC"

RUN set -x \
    && apt-get update \
    && apt-get -qq -y --no-install-recommends install \
      gnupg \
      wget \
      netcat-openbsd \
    && rm -rf /var/lib/apt/lists/*

RUN for key in $APACHE_KEYS; do \
      gpg --keyserver ha.pool.sks-keyservers.net --recv-key "$key" || \
      gpg --keyserver keyserver.ubuntu.com --recv-key "$key"; \
    done

ARG ZK_DIST=zookeeper-3.4.14
ARG ZK_PACKAGE=$ZK_DIST
RUN wget -q "$ZK_DOWNLOAD_URL/$ZK_DIST/$ZK_PACKAGE.tar.gz" \
    && wget -q "$ZK_DOWNLOAD_URL/$ZK_DIST/$ZK_PACKAGE.tar.gz.asc" \
    && apt-get autoremove -y wget \
    && gpg --batch --verify "$ZK_PACKAGE.tar.gz.asc" "$ZK_PACKAGE.tar.gz" \
    && tar -xzf "$ZK_PACKAGE.tar.gz" -C /opt \
    && rm -r "$ZK_PACKAGE.tar.gz" "$ZK_PACKAGE.tar.gz.asc" \
    && ln -s /opt/$ZK_PACKAGE /opt/zookeeper \
    && rm -rf /opt/zookeeper/CHANGES.txt \
      /opt/zookeeper/README.txt \
      /opt/zookeeper/NOTICE.txt \
      /opt/zookeeper/CHANGES.txt \
      /opt/zookeeper/README_packaging.txt \
      /opt/zookeeper/build.xml \
      /opt/zookeeper/config \
      /opt/zookeeper/contrib \
      /opt/zookeeper/dist-maven \
      /opt/zookeeper/docs \
      /opt/zookeeper/ivy.xml \
      /opt/zookeeper/ivysettings.xml \
      /opt/zookeeper/recipes \
      /opt/zookeeper/src \
      /opt/zookeeper/$ZK_PACKAGE.jar.asc \
      /opt/zookeeper/$ZK_PACKAGE.jar.md5 \
      /opt/zookeeper/$ZK_PACKAGE.jar.sha1

#Copy configuration generator script to bin
COPY scripts /opt/zookeeper/bin/

# Create a user for the zookeeper process and configure file system ownership
# for nessecary directories and symlink the distribution as a user executable
RUN set -x \
  && useradd $ZK_USER \
  && [ `id -u $ZK_USER` -eq 1000 ] \
  && [ `id -g $ZK_USER` -eq 1000 ] \
  && mkdir -p $ZK_DATA_DIR $ZK_DATA_LOG_DIR $ZK_LOG_DIR \
    /usr/share/zookeeper /tmp/zookeeper /usr/etc/ \
  && chown -R "$ZK_USER:$ZK_USER" /opt/$ZK_PACKAGE $ZK_DATA_DIR \
    $ZK_LOG_DIR $ZK_DATA_LOG_DIR /tmp/zookeeper \
  && ln -s /opt/zookeeper/conf/ /usr/etc/zookeeper \
  && ln -s /opt/zookeeper/bin/* /usr/bin \
  && ln -s /opt/zookeeper/$ZK_PACKAGE.jar /usr/share/zookeeper/ \
  && ln -s /opt/zookeeper/lib/* /usr/share/zookeeper
