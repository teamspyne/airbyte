FROM docker.io/airbyte/source-declarative-manifest:7.37.1@sha256:9ef4125a09cb4ce3a3ad12ffd326817ad6ddadb2957cced297bd731e3bc6b0e4

WORKDIR /airbyte/integration_code

COPY manifest.yaml source_declarative_manifest/manifest.yaml
COPY components.py source_declarative_manifest/components.py

ENV AIRBYTE_ENTRYPOINT="python /airbyte/integration_code/main.py"
ENTRYPOINT ["python", "/airbyte/integration_code/main.py"]
