#
# Docker file to create a Compute Server Node Manager
#

FROM ubuntu:20.04 as buildmanager
ARG GRB_VERSION=9.1.2
ARG GRB_SHORT_VERSION=9.1
# install manager package and copy the files
WORKDIR /opt

RUN apt-get update \
    && apt-get install --no-install-recommends -y\
       ca-certificates  \
       wget \
    && update-ca-certificates \
    && wget -v https://packages.gurobi.com/${GRB_SHORT_VERSION}/gurobi_server${GRB_VERSION}_linux64.tar.gz \
    && tar -xvf gurobi_server${GRB_VERSION}_linux64.tar.gz  \
    && rm -f gurobi_server${GRB_VERSION}_linux64.tar.gz \
    && mv -f gurobi_server* gurobi_server \
    && rm -rf gurobi/linux64/docs

# After the file renaming, a clean image is built

FROM ubuntu:20.04 AS packagemanager

ARG GRB_VERSION=9.1.2

LABEL vendor="Gurobi"
LABEL version=${GRB_VERSION}

# update system and certificates
RUN apt-get update \
    && apt-get install --no-install-recommends -y\
       ca-certificates  \
       p7zip-full \
       zip \
    && update-ca-certificates \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /opt/gurobi_server
COPY --from=buildmanager /opt/gurobi_server/linux64/bin/grb_rsm ./linux64/bin/grb_rsm
COPY --from=buildmanager /opt/gurobi_server/linux64/bin/grbcluster ./linux64/bin/grbcluster
COPY --from=buildmanager /opt/gurobi_server/linux64/bin/grb_rsm.cnf ./linux64/bin/grb_rsm.cnf
COPY --from=buildmanager /opt/gurobi_server/linux64/resources/grb_rsm ./linux64/resources/grb_rsm
COPY --from=buildmanager /opt/gurobi_server/linux64/EULA.pdf ./linux64/EULA.pdf

ENV GRB_HOME=/opt/gurobi_server/linux64
ENV PATH $GRB_HOME/bin:$PATH
WORKDIR $GRB_HOME/bin
# changes group permissions to run as a non-root user for better security
RUN chgrp -R 0 /opt/gurobi_server/linux64/bin && \
    chmod -R g=u /opt/gurobi_server/linux64/bin
# User to run the container
USER 1001
# expose command line
ENTRYPOINT ["grb_rsm", "--port=61080","--console-ts"]
EXPOSE 61080

