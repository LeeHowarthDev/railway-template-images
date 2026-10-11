
# ByteGraffiti HQ - Paperclip + Hermes
# Based on the official Paperclip image.
# Preserves the Railway bootstrap and startup scripts.

ARG PAPERCLIP_VERSION=latest
FROM ghcr.io/paperclipai/paperclip:${PAPERCLIP_VERSION}

USER root

# System dependencies
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       git curl ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Install uv, the Python environment manager
ENV UV_INSTALL_DIR=/usr/local/bin
ENV UV_PYTHON_INSTALL_DIR=/opt/uv-python
RUN curl -LsSf https://astral.sh/uv/install.sh | sh \
    && uv --version

# Install Hermes from source with its Git submodules
RUN git clone --depth 1 --recurse-submodules \
       --shallow-submodules \
       https://github.com/NousResearch/hermes-agent.git \
       /opt/hermes/source \
    && uv venv /opt/hermes/venv --python 3.14 \
    && cd /opt/hermes/source \
    && VIRTUAL_ENV=/opt/hermes/venv uv pip install -e "." \
    && ln -s /opt/hermes/venv/bin/hermes /usr/local/bin/hermes \
    && chmod -R a+rX /opt/hermes \
    && hermes --help

# Prepare Hermes configuration for Paperclip's node user
RUN mkdir -p /paperclip/.hermes \
    && chown -R node:node /paperclip/.hermes

# Preserve the existing Railway bootstrap
COPY --chown=node:node railway-bootstrap.mjs /app/server/railway-bootstrap.mjs
COPY railway-start.sh /usr/local/bin/railway-start.sh

RUN chmod 755 /usr/local/bin/railway-start.sh

# Run Paperclip as its normal user
USER node

# Keep the original Paperclip entrypoint
CMD ["railway-start.sh"]
