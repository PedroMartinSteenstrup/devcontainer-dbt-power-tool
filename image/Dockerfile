##
#  Generic dockerfile for dbt image building.
#  See README for operational details
###

# Top level build args
ARG build_for=linux/amd64

##
# base image (abstract)
##
FROM --platform=$build_for python:3.11.7-slim-bullseye as builder

# Copy requirements
COPY setup/ /setup/

# System setup
RUN apt-get update \
  && apt-get dist-upgrade -y \
  && xargs apt-get install -y --no-install-recommends </setup/os_packages.txt \
  && apt-get clean \
  && rm -rf \
    /var/lib/apt/lists/* \
    /tmp/* \
    /var/tmp/*

# Env vars
ENV PYTHONIOENCODING=utf-8
ENV LANG=C.UTF-8

# Update python
RUN python -m pip install --upgrade pip setuptools wheel --no-cache-dir

##
# dbt-snowflake
##
FROM builder as dbt-snowflake
# install additional requirements
RUN pip3 install --no-cache-dir -r /setup/requirements.txt

# Set user and group
ARG USERNAME=dbtuser
ARG USER_UID=1000
ARG USER_GID=$USER_UID
RUN groupadd --gid $USER_GID ${USERNAME}
RUN useradd --uid $USER_UID --gid $USER_GID -m ${USERNAME}

# creating the file to write XComs to, transferring ownership to ${user}
RUN mkdir -p /airflow/xcom
RUN echo "" > /airflow/xcom/return.json
RUN chown -R ${USER_UID}:${USER_GID} /airflow
# grant RW on airflow directory and its subtree
RUN chmod -R 755 /airflow

ENV DBT_BASE_DIR=/opt/dbt
RUN mkdir ${DBT_BASE_DIR} && \
    chown -R ${USERNAME}:${USER_GID} /home/${USERNAME} && \
    chown -R ${USERNAME}:${USER_GID} ${DBT_BASE_DIR} && \
    chmod -R 766 /home/${USERNAME} && \
    chmod -R 766 ${DBT_BASE_DIR}
ENV PYTHONPATH "${PYTHONPATH}:/code"

# Final build
FROM dbt-snowflake as base

# Switch to user
USER $USERNAME
WORKDIR ${DBT_BASE_DIR}

ENTRYPOINT ["dbt"]