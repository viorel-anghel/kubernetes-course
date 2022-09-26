# dummy Dockerfile
FROM ubuntu

RUN DEBIAN_FRONTEND=noninteractive apt-get update -q --fix-missing && \
  apt-get -y upgrade && apt-get -y install apt-utils

RUN DEBIAN_FRONTEND=noninteractive apt-get -y install \
  telnet iputils-ping net-tools netbase \
  netcat-openbsd rsync tcpdump procps grep coreutils diffutils \
  bind9-host wget curl socat vim-nox less

RUN DEBIAN_FRONTEND=noninteractive apt-get -y install mini-httpd

#RUN DEBIAN_FRONTEND=noninteractive apt-get -y install stress
RUN DEBIAN_FRONTEND=noninteractive apt-get -y install mysql-client
RUN DEBIAN_FRONTEND=noninteractive apt-get -y install postgresql-client
RUN DEBIAN_FRONTEND=noninteractive apt-get -y install redis-tools

RUN rm /var/www/html/index.mini-httpd.html && echo ok >/var/www/html/index.html
RUN mkdir -p /var/www/html/cgi-bin
COPY cgi-bin/ /var/www/html/cgi-bin
RUN chmod 755 /var/www/html/cgi-bin/*

USER root
WORKDIR /root

ENTRYPOINT ["/usr/sbin/mini_httpd", "-D", "-d", "/var/www/html", "-c", "cgi-bin/*", "-u", "www-data"]
