#####
## Image for running the ESGF SLCS server
##
## To avoid including unnecessary build dependencies in the final image, we use
## a builder image to check out the code and build wheels for all our dependencies
#####

ARG ESGF_HUB=esgfhub
ARG ESGF_PREFIX=
ARG ESGF_VERSION=latest

FROM python:2.7-slim AS esgf-slcs-build

# Install apt dependencies required for the build
#   git: for checking out the source code and pulling dependencies
#   build-essential: for gcc, required to build cryptography
#   libssl-dev: openssl header files, required to build cryptography
RUN apt-get update && \
    apt-get install -y git build-essential libssl-dev && \
    rm -rf /var/lib/apt/lists/*

# Clone the github repo and checkout the specified version
ARG ESGF_SLCS_GIT_VERSION=issue/3/remove-django-wsgi
RUN git clone https://github.com/ESGF/esgf-slcs-server.git /application && \
    cd /application && \
    git checkout $ESGF_SLCS_GIT_VERSION && \
    rm -rf .git && \
    cd -

# Create wheels from the requirements
RUN mkdir /pip-wheels && \
    pip wheel --wheel-dir /pip-wheels -r /application/requirements.txt


#############
## Stage 2 ##
#############

FROM ${ESGF_HUB}/${ESGF_PREFIX}django:${ESGF_VERSION}

MAINTAINER Earth System Grid Federation <esgf-devel@lists.llnl.gov>

# Copy the application and wheels from the build stage
COPY --from=esgf-slcs-build /application /application
COPY --from=esgf-slcs-build /pip-wheels  /pip-wheels

# Install the application + wheels (remove the wheels when we are done)
RUN cd /pip-wheels && \
    pip install --no-deps $(find /pip-wheels -type f -name '*.whl') && \
    pip install --no-deps -e /application && \
    rm -rf /pip-wheels

# Install Django settings
COPY conf/settings.py /application/esgf_slcs_server/

# Set the settings module to use
ENV DJANGO_SETTINGS_MODULE "esgf_slcs_server.settings"

# Run as the Django user unless otherwise specified
USER 1001
