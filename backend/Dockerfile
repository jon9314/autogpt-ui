FROM nikolaik/python-nodejs:python3.11-nodejs18

# Install browsers and all system-level build tools in one layer
RUN apt-get update && apt-get install -y \
    chromium-driver firefox-esr \
    ca-certificates libmagic1 \
    curl jq wget git ffmpeg \
    build-essential \
    python3-dev \
    && rm -rf /var/lib/apt/lists/*

# Set up environment
ENV PATH="$PATH:/root/.local/bin"
ENV PYTHONPATH=/src
ENV PLUGINS_DIR=/plugins

# Configure Poetry
RUN poetry config virtualenvs.create false

# Install Prisma CLI
RUN npm install -g prisma@4.11.0
ENV PRISMA_BINARY_CACHE_DIR=/usr/lib/

# Copy project files and install dependencies
COPY ./pyproject.toml ./poetry.lock* /
WORKDIR /
RUN poetry run pip install --upgrade pip
RUN poetry install --no-root --only main

# Download plugin dependencies
RUN mkdir ${PLUGINS_DIR}
RUN mkdir /src
RUN curl -L -o ${PLUGINS_DIR}/Auto-GPT-Plugins.zip https://github.com/Significant-Gravitas/Auto-GPT-Plugins/archive/refs/heads/master.zip
RUN curl -L -o ${PLUGINS_DIR}/AutoGPT_Slack.zip https://github.com/adithya77/Auto-GPT-slack-plugin/archive/refs/heads/main.zip
RUN curl -L -o ${PLUGINS_DIR}/AutoGPT_YouTube.zip https://github.com/Yolley/AutoGPT-YouTube/archive/refs/heads/master.zip

ARG AUTO_GPT_VERSION
ENV AUTO_GPT_VERSION ${AUTO_GPT_VERSION}

# Download and install the core Auto-GPT engine
RUN curl -L -o auto_gpt.zip https://github.com/Significant-Gravitas/Auto-GPT/archive/refs/tags/v${AUTO_GPT_VERSION}.zip \
    && unzip auto_gpt.zip \
    && cd "AutoGPT-${AUTO_GPT_VERSION}" \
    && mv autogpt /src/ \
    && pip install -r requirements.txt \
    && python scripts/check_requirements.py requirements.txt \
    # This is the final fix: find all plugin requirements and remove the "typing" package before installing
    && find /plugins -name "requirements.txt" -type f -exec sed -i '/^typing/d' {} + \
    && python scripts/install_plugin_deps.py

# Copy remaining application files
COPY ./src/ /src/
COPY ./migrations /migrations
COPY ./prisma_partial /prisma_partial
COPY ./schema.prisma /schema.prisma

COPY entrypoint.sh /entrypoint.sh
COPY gunicorn_conf.py /gunicorn_conf.py
COPY backend_pre_start.py /backend_pre_start.py

# Make entrypoint executable
RUN chmod +x /entrypoint.sh

# Set the entrypoint
ENTRYPOINT ["/entrypoint.sh"]
