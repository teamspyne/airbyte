FROM docker.io/airbyte/python-connector-base:4.1.1@sha256:a268b44c733ae699a60f5fbc06a324945dba98945c6e2ab7f8609f2f895b0d28

COPY . ./airbyte/integration_code
# README.md may be a symlink to a file outside the build context; poetry requires it to exist.
RUN rm -f ./airbyte/integration_code/README.md && touch ./airbyte/integration_code/README.md
RUN pip install ./airbyte/integration_code

# The entrypoint and default env vars are already set in the base image
ENV AIRBYTE_ENTRYPOINT="python /airbyte/integration_code/main.py"
ENTRYPOINT ["python", "/airbyte/integration_code/main.py"]