FROM ubuntu:14.04.5
MAINTAINER Daniel Eagle

ENV BIND_USER=bind \
  BIND_VERSION=1:9.9.5 \
  WEBMIN_VERSION=1.831 \
  DATA_DIR=/data

# Install Webmin and Bind
RUN apt-get update \
  && DEBIAN_FRONTEND=noninteractive apt-get install wget -y \
  && rm -rf /etc/apt/apt.conf.d/docker-gzip-indexes \
  && wget http://www.webmin.com/jcameron-key.asc -qO - | apt-key add - \
  && echo "deb http://download.webmin.com/download/repository sarge contrib" >> /etc/apt/sources.list \
  && apt-get update \
  && DEBIAN_FRONTEND=noninteractive apt-get install -y bind9=${BIND_VERSION}* bind9-host=${BIND_VERSION}* webmin=${WEBMIN_VERSION}* \
  && rm -rf /var/lib/apt/lists/*

# Add entrypoint
COPY ./config/entrypoint.sh /sbin/entrypoint.sh
RUN chmod +x /sbin/entrypoint.sh

# Add Tini
ENV TINI_VERSION v0.14.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini

EXPOSE 53/udp 53/tcp 10000/tcp

VOLUME ["${DATA_DIR}"]

ENTRYPOINT ["/tini", "--", "/sbin/entrypoint.sh"]

CMD ["/usr/sbin/named"]
