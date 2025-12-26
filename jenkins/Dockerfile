FROM jenkins/jenkins:lts-jdk17

USER root

# Install dependencies
RUN apt-get update && apt-get install -y \
    ca-certificates curl gnupg lsb-release git && \
    rm -rf /var/lib/apt/lists/*

# Install kubectl
RUN curl -fsSL https://dl.k8s.io/release/stable.txt -o /tmp/k8s_version && \
    curl -fsSL "https://dl.k8s.io/release/$(cat /tmp/k8s_version)/bin/linux/amd64/kubectl" -o /usr/local/bin/kubectl && \
    chmod +x /usr/local/bin/kubectl

# Install docker CLI
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker.gpg && \
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/debian bookworm stable" \
      > /etc/apt/sources.list.d/docker.list && \
    apt-get update && apt-get install -y docker-ce-cli && \
    rm -rf /var/lib/apt/lists/*

USER jenkins
