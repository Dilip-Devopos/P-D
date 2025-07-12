# Use Ubuntu as the base image
FROM ubuntu:latest

# Switch to root user to install dependencies
USER root

# Install dependencies, including Git, OpenSSH, Docker, Maven, and others
RUN apt-get update -qq && \
    DEBIAN_FRONTEND=noninteractive apt-get install -yq \
    curl wget unzip maven gnupg ca-certificates docker.io git openssh-client \
    cmake ninja-build clang g++ pkg-config libgtk-3-dev libatk-bridge2.0-dev libgl1-mesa-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Amazon Corretto 17 (Java)
RUN wget https://corretto.aws/downloads/latest/amazon-corretto-17-x64-linux-jdk.tar.gz && \
    mkdir -p /usr/lib/jvm/amazon-corretto-17 && \
    tar -xzf amazon-corretto-17-x64-linux-jdk.tar.gz -C /usr/lib/jvm/amazon-corretto-17 --strip-components=1 && \
    rm amazon-corretto-17-x64-linux-jdk.tar.gz

# Set JAVA_HOME and update PATH
ENV JAVA_HOME=/usr/lib/jvm/amazon-corretto-17
ENV PATH="$JAVA_HOME/bin:$PATH"

# Install Node.js & npm (LTS version)
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | bash - && \
    apt-get install -y nodejs && \
    npm install -g npm@latest

# Create Jenkins user
RUN useradd -m -s /bin/bash jenkins

# Switch to Jenkins user
USER jenkins

# Install Trivy (Security Scanning)
USER root
RUN OS_CODENAME=$(grep "VERSION_CODENAME" /etc/os-release | cut -d'=' -f2) && \
    curl -fsSL https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor -o /usr/share/keyrings/trivy-archive-keyring.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/trivy-archive-keyring.gpg] https://aquasecurity.github.io/trivy-repo/deb $OS_CODENAME main" | tee /etc/apt/sources.list.d/trivy.list && \
    apt update && apt install -y trivy && \
    rm -rf /var/lib/apt/lists/*

# Install SonarQube CLI (sonar-scanner)
RUN wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-linux.zip && \
    unzip sonar-scanner-cli-4.8.0.2856-linux.zip && \
    mv sonar-scanner-4.8.0.2856-linux /opt/sonar-scanner && \
    rm sonar-scanner-cli-4.8.0.2856-linux.zip

# Add sonar-scanner to PATH
ENV PATH="/opt/sonar-scanner/bin:$PATH"    

# Install Dependency-Check Tool (from GitHub)
RUN wget https://github.com/jeremylong/DependencyCheck/releases/download/v12.1.0/dependency-check-12.1.0-release.zip && \
    unzip dependency-check-12.1.0-release.zip -d /opt/ && \
    rm dependency-check-12.1.0-release.zip
    
# Add Dependency-Check to PATH
ENV PATH="/opt/dependency-check/bin:$PATH"

# Add Jenkins user to the Docker group and update group membership
RUN usermod -aG docker jenkins && newgrp docker

RUN npm install -g snyk

# Switch back to Jenkins user
USER jenkins

RUN mkdir -p /home/jenkins/download_artifact

# Set the correct permissions for the directory
RUN chown -R jenkins:jenkins /home/jenkins/download_artifact
