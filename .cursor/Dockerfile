FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    docker.io \
    docker-compose \
    curl \
    git \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js (replace 20 with your required version)
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

RUN curl -fsSL https://github.com/supabase/cli/releases/latest/download/supabase_linux_amd64.tar.gz \
    | tar -xz -C /usr/local/bin