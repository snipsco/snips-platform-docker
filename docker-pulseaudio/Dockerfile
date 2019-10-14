FROM ubuntu:18.04

RUN apt-get update \
    && apt-get install -y --force-yes apt-transport-https \
       ca-certificates \
       dirmngr 


ENV UNAME snips

RUN DEBIAN_FRONTEND=noninteractive apt-get install --yes alsa-utils \
	libasound2 \
	libasound2-plugins \
	pulseaudio \
	pulseaudio-utils \
	--no-install-recommends 

RUN bash -c  'echo "deb [trusted=yes] https://debian.snips.ai/stretch stable main" > /etc/apt/sources.list.d/snips.list' 

RUN apt-get update 
RUN apt-get install -y snips-platform-voice snips-watch snips-platform-demo 
RUN apt-get install -y snips-template snips-skill-server

RUN apt-get install -y --force-yes \
       jq \
       supervisor \
       mosquitto \
       mosquitto-clients \
       sox \
       unzip \
       --yes \
       --no-install-recommends \
    && mkdir -p /var/lib/snips \
    && apt-get -y autoclean \
    && apt-get -y autoremove \
    && rm -rf /var/lib/apt/lists/* 

RUN usermod -aG audio,pulse,pulse-access _snips
RUN usermod -aG audio,pulse,pulse-access root

COPY snips-entrypoint.sh /opt/snips/snips-entrypoint.sh
COPY VERSION /VERSION

COPY default.pa /etc/pulse/default.pa
COPY client.conf /etc/pulse/client.conf
COPY daemon.conf /etc/pulse/daemon.conf


ENTRYPOINT ["/opt/snips/snips-entrypoint.sh"]
