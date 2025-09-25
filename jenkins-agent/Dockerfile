# Stage 1: Installer (Debian-based for glibc >= 2.28)
FROM debian:bullseye as installer

# Install base dependencies
RUN apt-get update && apt-get install -y \
    unzip curl tar gzip git python3 python3-pip \
    && rm -rf /var/lib/apt/lists/*

# Install AWS CLI v2
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip \
    && unzip awscliv2.zip \
    && ./aws/install

# Install Snyk CLI
RUN curl -fsSL https://static.snyk.io/cli/v1.666.0/snyk-linux -o /usr/local/bin/snyk \
    && chmod +x /usr/local/bin/snyk

# Install Node.js 18 + npm
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*


# Stage 2: Jenkins Agent
FROM jenkins/agent:latest

# Switch to root to install packages
USER root

# Install Docker client + Python3
RUN apt-get update && \
    apt-get install -y docker.io python3 python3-pip && \
    rm -rf /var/lib/apt/lists/*

# Copy AWS CLI and Snyk from installer stage
COPY --from=installer /usr/local/aws-cli/ /usr/local/aws-cli/
COPY --from=installer /usr/local/bin/aws /usr/local/bin/aws
COPY --from=installer /usr/local/bin/snyk /usr/local/bin/snyk

# Copy Node.js and npm
COPY --from=installer /usr/bin/node /usr/bin/node
COPY --from=installer /usr/bin/npm /usr/bin/npm

# Switch back to Jenkins user
USER jenkins
