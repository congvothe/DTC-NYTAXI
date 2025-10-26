#Build args
ARG build_for=linux/arm64 

ARG py_version=3.12.10

FROM --platform=$build_for python:$py_version-slim-bullseye AS base

ARG dbt_core_ref=dbt-core@v1.0.1
ARG dbt_postgres_ref=dbt-postgres@v1.0.1
ARG dbt_reddshift_ref=dbt-redshift@v1.0.1
ARG dbt_snowflake_ref=dbt-snowflake@v1.0.1
ARG dbt_bigquery_ref=dbt-bigquery@v1.0.1
ARG dbt_spark_ref=dbt-spark@v1.0.1
#other args
ARG dbt_spark_version=all
ARG dbt_third_party

# Systen setup
RUN apt-get update \
  && apt-get dist-upgrade -y \
  && apt-get install -y --no-install-recommends \
    git \
    ssh-client \
    software-properties-common \
    build-essential \
    ca-certificates \
    libpq-dev \
    make \
    # Additional dependencies for building git from source
    wget \
    libcurl4-gnutls-dev \
    libexpat1-dev \
    libssl-dev \
    zlib1g-dev \
  # Build and install git from source to get 2.50.1 since debian only ships with 2.47 for now
  # && wget https://github.com/git/git/archive/refs/tags/v2.50.1.tar.gz \
  # && tar -xf v2.50.1.tar.gz \
  # && cd git-2.50.1 \
  # && make prefix=/usr/local NO_TCLTK=1 NO_GETTEXT=1 all \
  # && make prefix=/usr/local NO_TCLTK=1 NO_GETTEXT=1 install \
  # && cd .. \
  # && rm -rf git-2.50.1 v2.50.1.tar.gz \
  # Remove build dependencies to keep image slim
  # && apt-get remove -y \
  #   libcurl4-gnutls-dev \
  #   libexpat1-dev \
  #   libssl-dev \
  #   wget \
  #   zlib1g-dev \
  # && apt-get autoremove -y \
  # && apt-get clean \
  && rm -rf \
    /var/lib/apt/lists/* \
    /tmp/* \
    /var/tmp/*


#Setup environment  
ENV PYTHONIOENCODING=utf-8
ENV LANG=C.UTF-8

#Update python
RUN python -m pip install --upgrade "pip==24.0" "setuptools==69.2.0" "wheel==0.43.0" --no-cache-dir


#ARG commit_ref=main
#HEALTHCHECK CMD dbt --version || exit 1

# Setup docker basics
WORKDIR /usr/app/dbt/
VOLUME /usr/app
ENTRYPOINT ["dbt"]

# dbt-core
FROM base AS dbt-core
RUN python -m pip install --no-cache-dir "dbt-core @ git+https://github.com/dbt-labs/${dbt_core_ref}#egg=dbt-core&subdirectory=core"

# dbt-postgres
FROM base AS dbt-postgres
RUN python -m pip install --no-cache-dir "dbt-postgres @ git+https://github.com/dbt-labs/dbt-core@${commit_ref}#subdirectory=plugins/postgres"

# dbt-redshift
FROM base AS dbt-redshift
RUN python -m pip install --no-cache-dir "dbt-redshift @ git+https://github.com/dbt-labs/${dbt_redshift_ref}#egg=dbt-redshift"

# dbt-snowflake
FROM base AS dbt-snowflake
RUN python -m pip install --no-cache-dir "dbt-snowflake @ git+https://github.com/dbt-labs/${dbt_snowflake_ref}#egg=dbt-snowflake"

# dbt-bigquery
FROM base AS dbt-bigquery
RUN python -m pip install --no-cache-dir "dbt-bigquery @ git+https://github.com/dbt-labs/${dbt_bigquery_ref}#egg=dbt-bigquery"

# dbt-spark
FROM base AS dbt-spark
RUN apt-get update \
    && apt-get dist-upgrade -y \
    && apt-get install -y --no-install-recommends \
      python-dev \
      libsas12-dev \ 
      gcc \
      unixodbc-dev \
    && apt-get clean \
    && rm -rf \
      /var/lib/apt/lists/* \
      /tmp/* \
      /var/tmp/*
RUN python -m pip install --no-cache-dir "dbt-spark @ git+https://github.com/dbt-labs/${dbt_spark_ref}#egg=dbt-spark[${dbt_spark_version}]"

# dbt-third-party
FROM dbt-core AS dbt-third-party
RUN python -m pip install --no-cache-dir "${dbt_third+party}"

# dbt-all
FROM dbt-core AS dbt-all
RUN apt-get update \
  && apt-get dist-upgrade -y \
  && apt-get install -y --no-install-recommends \
    python-dev \
    libsas12-dev \ 
    gcc \ 
    unixodbc-dev \
  && apt-get clean \
  && rm -rf \
    /var/lib/apt/lists/* \
    /tmp/* \
    /var/tmp/*
  RUN python -m pip install --no-cache "git+https://github.com/dbt-labs/${dbt_redshift_ref}#egg=dbt-redshift"
  RUN python -m pip install --no-cache "git+https://github.com/dbt-labs/${dbt_bigquery_ref}#egg=dbt-bigquery"
  RUN python -m pip install --no-cache "git+https://github.com/dbt-labs/${dbt_snowflake_ref}#egg=dbt-snowflake"
  RUN python -m pip install --no-cache "git+https://github.com/dbt-labs/${dbt_spark_ref}#egg=dbt-spark[${dbt_spark_version}]"
  RUN python -m pip install --no-cache "git+https://github.com/dbt-labs/${dbt_postgres_ref}#egg=dbt-postgres&subdirectory=plugins/postgres"  