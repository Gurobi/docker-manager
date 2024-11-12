#
# Docker file to create a Compute Server Node Manager
#

FROM ubuntu:20.04 as buildmanager
ARG GRB_VERSION=12.0.0
ARG GRB_SHORT_VERSION=12.0
ARG TARGETPLATFORM

RUN apt-get update \
    && apt-get install --no-install-recommends -y \
       ca-certificates \
       wget \
    && update-ca-certificates

# install manager package and copy the files
WORKDIR /opt

RUN if [ "$TARGETPLATFORM" = "linux/arm64" ]; then \
        export GRB_PLATFORM="armlinux64"; \
      else \
        export GRB_PLATFORM="linux64"; \
    fi \
    && wget -v https://packages.gurobi.com/${GRB_SHORT_VERSION}/gurobi_server${GRB_VERSION}_$GRB_PLATFORM.tar.gz \
    && tar -xvf gurobi_server${GRB_VERSION}_$GRB_PLATFORM.tar.gz  \
    && rm *.tar.gz \
    && mv -f gurobi_server* gurobi_server \
    && mv -f gurobi_server/$GRB_PLATFORM*  gurobi_server/linux
# After the file renaming, a clean image is built

FROM ubuntu:20.04 AS packagemanager

ARG GRB_VERSION=12.0.0

LABEL vendor="Gurobi"
LABEL version=${GRB_VERSION}

# update system and certificates
RUN apt-get update -o Debug::Acquire::http=true \
    && apt-get install --no-install-recommends -y\
       ca-certificates  \
       p7zip-full \
       zip \
    && update-ca-certificates \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /opt/gurobi_server
COPY --from=buildmanager /opt/gurobi_server/linux/bin/grb_rsm ./linux/bin/grb_rsm
COPY --from=buildmanager /opt/gurobi_server/linux/bin/grbcluster ./linux/bin/grbcluster
COPY --from=buildmanager /opt/gurobi_server/linux/bin/grb_rsm.cnf ./linux/bin/grb_rsm.cnf
COPY --from=buildmanager /opt/gurobi_server/linux/resources/grb_rsm ./linux/resources/grb_rsm
COPY --from=buildmanager /opt/gurobi_server/linux/EULA.pdf ./linux/EULA.pdf

ENV GRB_HOME=/opt/gurobi_server/linux
ENV PATH $GRB_HOME/bin:$PATH
WORKDIR $GRB_HOME/bin
# changes group permissions to run as a non-root user for better security
RUN chgrp -R 0 /opt/gurobi_server/linux/bin && \
    chmod -R g=u /opt/gurobi_server/linux/bin
# User to run the container
USER 1001
# expose command line
ENTRYPOINT ["grb_rsm", "--port=61080","--console-ts"]
EXPOSE 61080

