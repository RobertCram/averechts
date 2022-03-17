FROM ruby:3.1.0

RUN apt-get update \ 
    && apt-get install -y locales locales-all \
    && apt-get install -y postgresql-client \
    && apt-get install -y libxml2 \
    && apt-get install -y libxml2-dev \ 
    && apt-get install -y libxslt1-dev \ 
    && apt-get install -y nodejs \ 
    && apt-get install -y file

# Install latest version of Chrome
RUN wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb \
    && apt-get install -y ./google-chrome-stable_current_amd64.deb \
    && rm ./google-chrome-stable_current_amd64.deb

ARG USERNAME=postgres
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create a postgres user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    #
    # [Optional] Add sudo support. Omit if you don't need to install software after connecting.
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

RUN apt-get install -y postgresql

RUN mkdir -p /var/postgres/data \
    && mkdir -p /var/log/postgres \
    && chown postgres /var/postgres/data \
    && chown postgres /var/log/postgres

COPY createdb.sh /
RUN chmod +rwx createdb.sh

USER postgres

ENV LANG="C.UTF-8"

RUN ./createdb.sh
    
USER root

RUN gem install bundler:2.3.8

ARG BRANCH=develop

# Invalidates the cache when branch changes
ADD https://api.github.com/repos/RobertCram/averechts/git/refs/heads/$BRANCH version.json

# Clones latest version of the branch
RUN mkdir /workspaces && cd /workspaces && git clone --branch $BRANCH https://github.com/RobertCram/averechts.git averechts

# Ensure gems are cached and only get updated when they change. This will
# drastically increase build times when your gems do not change.
RUN cd /workspaces/averechts && bundle install