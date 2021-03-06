############################################################
# Dockerfile to build OpenZWave Library container images
# Based on CentOS7
############################################################

# Set the base image to Alpine
FROM debian:bullseye-slim as builder

# File Author / Maintainer
LABEL maintainer="justin@dynam.ac"

WORKDIR /opt

#RUN echo 'APT::Default-Release "stable";' > /etc/apt/apt.conf.d/99defaultrelease \
#	&& echo 'deb     http://ftp.debian.org/debian/    unstable main contrib non-free' > /etc/apt/sources.list.d/unstable.list \
#	&& apt update
RUN apt update
RUN apt-get -y install rapidjson-dev git cmake make pkgconf bash python wget joe mc libunwind-dev libcurl4-openssl-dev 
RUN apt-get -y install g++
RUN apt-get -y install qt5-default libqt5websockets5-dev qtbase5-private-dev libqt5remoteobjects5-dev libqt5remoteobjects5-bin
#&& apt-get -y -t unstable install qtbase5-private-dev && apt-get -y -t unstable install libqt5remoteobjects5-dev libqt5remoteobjects5-bin 

ENV PATH=$PATH:/opt/depot_tools/
ENV DEPOT_TOOLS_UPDATE=0
RUN git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git \
	&& cd depot_tools && git checkout 464e9ff && cd .. \
	&& mkdir breakpad \
	&& cd breakpad \
	&& fetch breakpad \
	&& cd src \
	&& ./configure --disable-processor \
	&& make \
	&& make install
RUN git clone https://github.com/qt/qtmqtt.git \
	&& cd qtmqtt \
	&& git checkout 5.12 \
	&& /usr/lib/qt5/bin/qmake QT_BUILD_PARTS="libs tools" \
	&& make \
	&& make install
RUN git clone https://github.com/OpenZWave/open-zwave.git \
	&& cd open-zwave \
	&& make -j4 \
	&& make install
COPY . /opt/qt-openzwave/
ARG BP_CLIENTID
ARG BP_CLIENTKEY
ARG BUILDNUMBER=0
RUN cd qt-openzwave \
	&& if [ -f Makefile ]; then /usr/lib/qt5/bin/qmake -r; make distclean; fi \
	&& /usr/lib/qt5/bin/qmake -r "BP_CLIENTID=$BP_CLIENTID" "BP_CLIENTKEY=$BP_CLIENTKEY" "BUILDNUMBER=$BUILDNUMBER" \
	&&  make -j4 \
	&& make install

FROM debian:bullseye-slim

LABEL maintainer="justin@dynam.ac"

WORKDIR /opt

#RUN echo 'APT::Default-Release "stable";' > /etc/apt/apt.conf.d/99defaultrelease \
#    && echo 'deb http://ftp.debian.org/debian/ unstable main contrib non-free' > /etc/apt/sources.list.d/unstable.list \
RUN apt update \
    && apt-get -y install libunwind8 libcurl4 binutils \
    && apt-get -y install libqt5core5a libqt5network5 libqt5websockets5 libqt5remoteobjects5 \
    && strip --remove-section=.note.ABI-tag /usr/lib/*/libQt5Core.so.5 \
    && apt-get -y purge binutils \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && mkdir -p /opt/ozw/config/crashes/

COPY --from=builder /usr/local/bin/* /usr/local/bin/
COPY --from=builder /usr/lib/*/libqt-* /usr/local/lib/
COPY --from=builder /usr/local/lib*/libopenzwave* /usr/local/lib/
COPY --from=builder /usr/lib/*/libQt5Mqtt* /usr/local/lib/
COPY --from=builder /opt/qt-openzwave/tools/* /usr/local/bin/
COPY --from=builder /usr/share/qt5/qt-openzwavedatabase.rcc /usr/share/qt5/
COPY Docker/startozwdaemon.sh /
RUN mkdir -p /opt/ozw/config/crashes/
ENV LD_LIBRARY_PATH="/usr/local/lib:/usr/local/lib64:$LD_LIBRARY_PATH"
ENV USB_PATH="/dev/ttyUSB0"
ENV MQTT_SERVER="localhost"
ENV MQTT_PORT="1883"
ENV OZW_INSTANCE="1"
ENV BP_DB_PATH="/opt/ozw/config/crashes/"
WORKDIR /opt/ozw/
EXPOSE 1983
VOLUME ["/opt/ozw/config/"]
ENTRYPOINT ["/startozwdaemon.sh"]
