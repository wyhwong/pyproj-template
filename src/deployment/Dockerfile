# STAGE 1 - Export dependencies
FROM python:3.11-slim-buster AS base

# Convert poetry.lock to requirements.txt
RUN pip3 install poetry poetry-plugin-export
COPY ./pyproject.toml ./poetry.lock /
RUN poetry export -f requirements.txt -o requirements.txt \
    --without-hashes \
    --without dev
RUN poetry export -f requirements.txt -o requirements-dev.txt \
    --without-hashes \
    --only dev

# STAGE 2 - Build main image
FROM python:3.11-slim-buster AS main

# Set version for analyticks installation
ARG VERSION
ENV VERSION=${VERSION}

# Set working directory and freeze scripts
COPY --from=base /requirements.txt /tmp/requirements.txt
RUN python3 -m pip install --no-cache-dir -r /tmp/requirements.txt

# Get args
ARG USERNAME
ARG USER_ID
ARG GROUP_ID

# Create user
RUN groupadd --gid ${GROUP_ID} ${USERNAME} && \
    adduser --disabled-password --gecos '' --uid ${USER_ID} --gid ${GROUP_ID} ${USERNAME}
USER ${USERNAME}

# Set working directory
WORKDIR /home/${USERNAME}
COPY ./app /home/${USERNAME}/app

# [Optional] Setup test environment
FROM main AS test

# Install dev dependencies and copy test cases into the image
# NOTE: Here we created a file /.dockertest for the test cases to
#       be able to recognize the infrastructure correctly.
USER root
RUN touch /.dockertest

USER ${USERNAME}
COPY --from=base /requirements-dev.txt /tmp/requirements-dev.txt
RUN python3 -m pip install --no-cache-dir -r /tmp/requirements-dev.txt
COPY ./tests/ /home/${USERNAME}/tests/
