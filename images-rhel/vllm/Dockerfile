# Dockerfile.vllm
# Thin layer on top of RHOAI vLLM serving runtime for Slurm workloads.
# Base image includes vLLM, CUDA, and all inference dependencies.

FROM registry.redhat.io/rhoai/odh-vllm-cuda-rhel9:v2.25.3

LABEL org.opencontainers.image.title="vllm" \
      org.opencontainers.image.description="RHOAI vLLM serving runtime configured for Slurm"

USER root

# Create model cache directory
RUN mkdir -p /opt/models \
    && chown 1001:0 /opt/models \
    && chmod 0775 /opt/models

ENV HF_HOME="/opt/models" \
    VLLM_WORKER_MULTIPROC_METHOD="spawn"

USER 1001

EXPOSE 8000

ENTRYPOINT ["python3", "-m", "vllm.entrypoints.openai.api_server"]
CMD ["--host", "0.0.0.0", "--port", "8000"]
