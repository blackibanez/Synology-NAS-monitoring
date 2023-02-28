FROM ubuntu:20.04

LABEL maintainer="blackibanez"

ENV DEBIAN_FRONTEND noninteractive
ENV LANG C.UTF-8

# Default versions
ENV TELEGRAF_VERSION 1.52.2
ENV INFLUXDB_VERSION 2.6.1
ENV GRAFANA_VERSION  9.3.6
ENV CHRONOGRAF_VERSION 1.10

ENV GF_DATABASE_TYPE=sqlite3

# Clear previous sources
RUN rm /var/lib/apt/lists/* -vf

# Base dependencies

RUN apt-get -y update && \
 apt-get -y dist-upgrade && \
 apt-get -y install \
  apt-utils \
  ca-certificates \
  curl \
  git \
  vim \
  htop \
  libfontconfig \
  mc \
  net-tools \
  openssh-server \
  supervisor \
  gnupg \
  gnupg2 \
  gnupg1 \
  snmp \
  syslog-ng \
  libnss3-tools \
  libatk-bridge2.0-0 \
  libcups2-dev \
  libdrm2 \
  libxkbcommon-x11-0 \
  libxcomposite-dev \
  libxdamage-dev \
  libxrandr2 \
  libgbm-dev \
  libcairo2 \
  libasound2-dev \
  libxshmfence-dev \
  libpango-1.0-0 \
  libatk1.0-0 \
  snmp-mibs-downloader \
  wget && \
 curl -sL https://deb.nodesource.com/setup_15.x | bash - && \
 apt-get install -y nodejs

# Configure Supervisord, SSH and base env
COPY supervisord/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

WORKDIR /root

RUN mkdir -p /var/log/supervisor && \
    mkdir -p /var/run/sshd && \
    echo 'root:root' | chpasswd && \
    rm -rf .ssh && \
    rm -rf .profile && \
    mkdir .ssh

COPY bash/profile .profile

# Install InfluxDB
#RUN wget https://dl.influxdata.com/influxdb/releases/influxdb2_${INFLUXDB_VERSION}_amd64.deb && \
## dpkg -i influxdb_${INFLUXDB_VERSION}_amd64.deb && rm influxdb2_${INFLUXDB_VERSION}_amd64.deb

# Install InfluxDB2
RUN wget -q https://repos.influxdata.com/influxdata-archive_compat.key 
RUN echo '393e8779c89ac8d958f81f942f9ad7fb82a25e133faddaf92e15b16e6ac9ce4c influxdata-archive_compat.key' | sha256sum -c && cat influxdata-archive_compat.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg > /dev/null
RUN echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/debian stable main' | sudo tee /etc/apt/sources.list.d/influxdata.list
RUN apt-get update && sudo apt-get install influxdb2

# Configure InfluxDB
COPY influxdb/influxdb.conf /etc/influxdb/influxdb.conf
COPY influxdb/init.sh /etc/init.d/influxdb

#install syslog-ng 
COPY syslog/syslog-ng.conf /etc/syslog-ng/syslog-ng.conf

# Install Telegraf
#RUN wget https://dl.influxdata.com/telegraf/releases/telegraf_${TELEGRAF_VERSION}_amd64.deb && \
# dpkg -i telegraf_${TELEGRAF_VERSION}_amd64.deb && rm telegraf_${TELEGRAF_VERSION}_amd64.deb
 
 # influxdata-archive_compat.key GPG fingerprint:
#     9D53 9D90 D332 8DC7 D6C8 D3B9 D8FF 8E1F 7DF8 B07E
RUN wget -q https://repos.influxdata.com/influxdata-archive_compat.key
RUN echo '393e8779c89ac8d958f81f942f9ad7fb82a25e133faddaf92e15b16e6ac9ce4c influxdata-archive_compat.key' | sha256sum -c && cat influxdata-archive_compat.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg > /dev/null
RUN echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/debian stable main' | sudo tee /etc/apt/sources.list.d/influxdata.list
RUN sudo apt-get update && sudo apt-get install telegraf

# Configure Telegraf
RUN mv -f /etc/telegraf/telegraf.conf /etc/telegraf/telegraf.conf.default 
COPY telegraf/telegraf.conf /etc/telegraf/telegraf.conf
COPY telegraf/init.sh /etc/init.d/telegraf

# Install Chronograf
RUN wget https://dl.influxdata.com/chronograf/releases/chronograf_${CHRONOGRAF_VERSION}_amd64.deb && \
 dpkg -i chronograf_${CHRONOGRAF_VERSION}_amd64.deb  && rm chronograf_${CHRONOGRAF_VERSION}_amd64.deb

# Install Grafana
RUN wget https://dl.grafana.com/oss/release/grafana_${GRAFANA_VERSION}_amd64.deb && \
 dpkg -i grafana_${GRAFANA_VERSION}_amd64.deb && rm grafana_${GRAFANA_VERSION}_amd64.deb

# Configure Grafana with provisioning
ADD grafana/provisioning /etc/grafana/provisioning
COPY grafana/grafana.ini /etc/grafana/grafana.ini

# Synology SNMP
COPY synology/* /usr/share/snmp/mibs/
RUN chown root:root /usr/share/snmp/mibs
RUN chmod 755 /usr/share/snmp/mibs


# Install plugins
#COPY rootfs /tmp
#RUN /tmp/grafana-plugins.sh

#

EXPOSE 22/tcp 3003/tcp 8086/tcp 8888/tcp 8125/udp 514/udp
#VOLUME /var/lib/influxdb /var/lib/grafana /var/lib/backups

# Cleanup
RUN apt-get clean && \
 rm -rf /var/lib/apt/lists/*  /var/tmp/*

CMD ["/usr/bin/supervisord"]
