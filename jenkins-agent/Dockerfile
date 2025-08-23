# Stage 1: Installer
FROM amazonlinux:2 as installer

# Install AWS CLI
RUN yum update -y \
    && yum install -y unzip curl \
    && curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip \
    && unzip awscliv2.zip \
    && ./aws/install --bin-dir /aws-cli-bin/

# Install Snyk CLI
RUN mkdir /snyk && cd /snyk \
    && curl https://static.snyk.io/cli/v1.666.0/snyk-linux -o snyk \
    && chmod +x ./snyk

# Stage 2: Jenkins agent
FROM jenkins/agent

# Switch to root to install packages
USER root

# Install Docker client, Python3, pip3
RUN apt-get update && \
    apt-get install -y docker.io python3 python3-pip && \
    rm -rf /var/lib/apt/lists/*

# Copy AWS CLI and Snyk from installer stage
COPY --from=installer /usr/local/aws-cli/ /usr/local/aws-cli/
COPY --from=installer /aws-cli-bin/ /usr/local/bin/
COPY --from=installer /snyk/snyk /usr/local/bin/

# Switch back to Jenkins user
USER jenkins

