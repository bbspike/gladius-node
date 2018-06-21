FROM debian:stretch-slim

# install curl to download gladius
RUN apt-get update \
  && apt-get install -y curl \
  && rm -rf /var/lib/apt/lists/*

# create gladius user
RUN adduser --no-create-home --system gladius

# setup gladius key, wallet (controld) and content (networkd) volumes
# inside the docker container we store everything inside GLADIUSBASE
ENV GLADIUSBASE=/gladius
RUN mkdir -p ${GLADIUSBASE}/content \
	&& mkdir -p ${GLADIUSBASE}/wallet \
	&& mkdir -p ${GLADIUSBASE}/keys \
	&& touch ${GLADIUSBASE}/gladius-networkd.toml \
	&& touch ${GLADIUSBASE}/gladius-controld.toml \
  && touch ${GLADIUSBASE}/start.sh \
  && chmod a+x ${GLADIUSBASE}/start.sh

# download and install gladius binaries
RUN curl -L \
		https://gladius-node-hackathon.nyc3.digitaloceanspaces.com/gladius-hackathon-linux-amd64.tar.gz \
		-o /tmp/gladius.tar.gz \
  && tar -xzf /tmp/gladius.tar.gz -C /usr/local/bin/ --strip-components 1 \
  && rm -f /tmp/gladius.tar.gz
# download the gladius content file
RUN curl -L https://gladius-node-hackathon.nyc3.digitaloceanspaces.com/content.tar.gz \
         -o /tmp/content.tar.gz \
  && tar -xzf /tmp/content.tar.gz -C ${GLADIUSBASE}/content/ --strip-components 1 \
  && rm -f /tmp/content.tar.gz
# make sure all data inside the base directory belongs to the gladius user
RUN chown -R gladius ${GLADIUSBASE}

# be sure to expose volumes for the different directories
# inside gladius base
VOLUME ${GLADIUSBASE}/content
VOLUME ${GLADIUSBASE}/wallet
VOLUME ${GLADIUSBASE}/keys

RUN echo '#!/bin/sh' >> ${GLADIUSBASE}/start.sh
RUN echo 'echo IP IS: $GLADIUS_HOSTIP' >> ${GLADIUSBASE}/start.sh
RUN echo 'gladius create wallet' >> ${GLADIUSBASE}/start.sh
RUN echo 'account=$(gladius account)' >> ${GLADIUSBASE}/start.sh
RUN echo 'echo Account is: $account' >> ${GLADIUSBASE}/start.sh
RUN echo "curl -s -X \"POST\" \"http://162.243.175.169/transfer/\$account\" \
     -H \"Content-Type: application/json\" \
     -d '{\"amount\": \"0.05\"}'" >> ${GLADIUSBASE}/start.sh
RUN echo sleep 1m >> ${GLADIUSBASE}/start.sh
RUN echo 'gladius create node "Node $(hostname)" "null@null.com"' >> ${GLADIUSBASE}/start.sh
RUN echo sleep 1m >> ${GLADIUSBASE}/start.sh
RUN echo 'gladius apply 0xEC377B425c452B72a048d38378C6aDb160648d02' >> ${GLADIUSBASE}/start.sh

# expose the ports used by the different services
# controld
EXPOSE 3001
# networkd
EXPOSE 5000
EXPOSE 8080

USER gladius
