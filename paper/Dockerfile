FROM gradle:7.3.2-alpine as build-banstick

# certain packages require git to shadowJar
RUN apk add --no-cache curl git

# Github Package Repository requires authentication even when pulling from public repositories
# https://github.community/t/download-from-github-package-registry-without-authentication/14407
ARG GITHUB_ACTOR
ARG GITHUB_TOKEN
# make sure args are set
RUN test -n "$GITHUB_ACTOR" && test -n "$GITHUB_TOKEN"

WORKDIR /src
ARG BANSTICK_VERSION="v2.0.0-SNAPSHOT-2"
ARG BANSTICK_SHA256="cbde8d74dd2621191d5ff905413dc731f3c01074479826d0ddf2afab68283515  banstick.tar.gz"
RUN curl -L https://github.com/CivMC/BanStick/archive/refs/tags/"$BANSTICK_VERSION".tar.gz > banstick.tar.gz \
    && echo "$BANSTICK_SHA256" | sha256sum -c \
    && tar -xz --strip-components=1 < banstick.tar.gz \
    && rm banstick.tar.gz \
    && gradle shadowJar

FROM itzg/minecraft-server

# https://github.com/itzg/docker-minecraft-server/blob/master/README.md#versions
ENV VERSION=1.18.2

# https://github.com/itzg/docker-minecraft-server#optional-plugins-mods-and-config-attach-points
COPY plugins /plugins
COPY --from=build-banstick /src/paper/build/libs/BanStick-paper-2.0.0-SNAPSHOT-dev-all.jar /plugins/BanStick-paper-2.0.0-SNAPSHOT.jar
ENV REMOVE_OLD_MODS=TRUE

# https://github.com/itzg/docker-minecraft-server#optional-plugins-mods-and-config-attach-points
COPY config /config
ENV COPY_CONFIG_DEST=/data
ENV SYNC_SKIP_NEWER_IN_DESTINATION=false

# https://github.com/itzg/docker-minecraft-server#running-a-paper-server
ENV TYPE=PAPER
ENV EULA=TRUE
# https://github.com/itzg/docker-minecraft-server#replacing-variables-inside-configs
ENV REPLACE_ENV_DURING_SYNC=TRUE
ENV REPLACE_ENV_VARIABLE_PREFIX=CIV
