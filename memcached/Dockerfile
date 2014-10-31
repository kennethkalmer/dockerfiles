FROM ubuntu:14.04
MAINTAINER Kenneth Kalmer "https://github.com/kennethkalmer/dockerfiles"

# Install packages
RUN DEBIAN_FRONTEND=noninteractive apt-get update &&\
    apt-get -y install memcached

# memcached public variable
ENV MAX_MEM 64
ENV BIND 0.0.0.0
ENV MAX_CONN 1024

EXPOSE 11211

CMD ["/usr/bin/memcached", "-u", "root", "-v", "-m", "$MAX_MEM", "-l", "$BIND", "-c", "$MAX_CONN"]
