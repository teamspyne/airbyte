FROM docker.io/airbyte/source-declarative-manifest:7.10.1@sha256:bd69f6b52bdd92ce06c30786d67546427b828ffb553363c8950b2816c552d6e0

WORKDIR /airbyte/integration_code

COPY manifest.yaml source_declarative_manifest/manifest.yaml
COPY components.py source_declarative_manifest/components.py

ENV AIRBYTE_ENTRYPOINT="python /airbyte/integration_code/main.py"
ENTRYPOINT ["python", "/airbyte/integration_code/main.py"]
