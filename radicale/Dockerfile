# Based on https://github.com/jcsaaddupuy/docker-radicale/blob/master/Dockerfile
FROM mail-base
MAINTAINER Steve Farthing <me@stevefar.com>

ENV DEBIAN_FRONTEND noninteractive

## Base packages
RUN apt-get update -qq && apt-get upgrade -y -qq
RUN apt-get install -y python2.7 python-pip
######

## Radicale installation
RUN pip install radicale
######

# Create some folder
RUN mkdir -p /data/radicale
RUN mkdir -p /root/.config
RUN ln -s /data/radicale /root/.config/radicale
######

 
EXPOSE 5232

# Fix empty $HOME
ENV HOME /root
# WORKDIR /root
CMD ["radicale"]