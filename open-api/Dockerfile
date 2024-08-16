FROM ubuntu:22.04
RUN apt update && \
  apt install make wget patch openjdk-17-jre -y && \
  wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
RUN . ~/.bashrc && nvm install v20.16.0
ENV PATH="$PATH:/root/.nvm/versions/node/v20.16.0/bin"
WORKDIR /usr/src
