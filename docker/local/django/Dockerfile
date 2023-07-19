ARG PYTHON_VERSION=3.9.6-bullseye

# The command bellow is used to get the python image from docker hub. The command AS is used to give a name to the image. And the command FROM is used to get the image from docker hub.
FROM python:${PYTHON_VERSION} as python


FROM python as python-buil-stage
ARG BUID_ENVIROMENT=local

# The command bellow apt-get update is used to update the packages. The command apt-get install is used to install the packages. The command --no-instal-recommend is used to not install the recommended packages. The command -y is used to not ask for confirmation. And the command build-essencial is used to install the packages that are needed to build the dependencies of psycopg2. And the command libpq-dev is used to install the dependencies of psycopg2.
RUN apt-get update && apt-get install --no-instal-recommend -y \
    build-essencial \
    libpq-dev

COPY ./requirements .


RUN pip wheel --wheel-dir /usr/src/app/wheels \
    -r ${BUID_ENVIROMENT}.txt

FROM python as python-run-stage
ARG BUID_ENVIROMENT=local

ARG APP_HOME=/app

# The command bellow is used to create the directory /app.
ENV PYTHONDONTWRITEBYTECODE 1

# The command bellow is used to not buffer the output of the python process.
ENV PYTHONUNBUFFERED 1


ENV BUID_ENV ${BUID_ENVIROMENT}

# The command bellow is used to create the directory /app.
WORKDIR ${APP_HOME}

# The command bellows is used to install the dependencies of psycopg2. The command --no-instal-recommend is used to not install the recommended packages. The command -y is used to not ask for confirmation. The command purge is used to remove the dependencies that are not needed anymore. The command --auto-remove is used to remove the dependencies that are not needed anymore. The command -o APT::AutoRemove::RecommendsImportant=false is used to not remove the recommended packages. And the command rm -rf /var/lib/apt/lists/* is used to remove the directory /var/lib/apt/lists/.
RUN apt-get update && apt-get install --no-instal-recommend -y \
    libpq-dev \
    get \
    && apt-get purge -y --auto-remove -o \
    APT::AutoRemove::RecommendsImportant=false \
    && rm -rf /var/lib/apt/lists/*

# The command bellow is used to copy the dependencies from the python-build-stage to the python-run-stage.
COPY --from=python-buil-stage /usr/src/app/wheels /wheels


# The command bellow is used to install the dependencies with pip. The command --no-cache-dir is used to not cache the index locally. The command --no-index is used to not use the PyPI index. The command --find-links is used to find the dependencies in the directory /wheels. The command /wheels/* is used to install all the dependencies in the directory /wheels. And the command rm -rf /wheels is used to remove the directory /wheels.
# We do this to avoid the download of the dependencies every time we build the image. We download the dependencies once and then we use the cache to install them. Is a way to optimize the build process.
RUN pip install --no-cache-dir --no-index --find-links=/wheels/ /wheels/* \
    && rm -rf /wheels

COPY .docker/local/django/entrypoint /entrypoint

# The command bellow sed -i is used to edit the file /entrypoint. The command 's/\r$//g' is used to remove the \r from the file /entrypoint. The s/\r$//g is a regular expression that means: replace \r with nothing.
RUN sed -i 's/\r$//g' /entrypoint

# The command bellow is used to give permission to the file /entrypoint. +x means that we are giving permission to execute the file /entrypoint.
RUN chmod +x /entrypoint

# The command COPY .docker/local/django/start /start is used to copy the file .docker/local/django/start to the directory /start. 
# The command RUN sed -i 's/\r$//g' /start is used to edit the file /start. The command 's/\r$//g' is used to remove the \r from the file /start. The s/\r$//g is a regular expression that means: replace \r with nothing.
# The command RUN chmod +x /start is used to give permission to the file /start. +x means that we are giving permission to execute the file /start.
COPY .docker/local/django/start /start
RUN sed -i 's/\r$//g' /start
RUN chmod +x /start

COPY . ${APP_HOME}

ENTRYPOINT [ "/entrypoint" ]