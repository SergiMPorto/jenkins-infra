FROM jenkins/jenkins:lts

USER root

# Instalar dependencias básicas
RUN apt-get update && apt-get install -y \
    curl \
    ca-certificates \
    gnupg \
    && rm -rf /var/lib/apt/lists/*

# Añadir clave oficial Docker
RUN install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/debian/gpg \
    -o /etc/apt/keyrings/docker.asc && \
    chmod a+r /etc/apt/keyrings/docker.asc

# Añadir repositorio Docker para Debian Trixie arm64
RUN echo \
  "deb [arch=arm64 signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/debian trixie stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalar Docker CLI
RUN apt-get update && apt-get install -y \
    docker-ce-cli \
    && rm -rf /var/lib/apt/lists/*

# Instalar kubectl para arm64
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/arm64/kubectl" && \
    chmod +x kubectl && \
    mv kubectl /usr/local/bin/kubectl

# Instalar sonar-scanner para arm64
COPY sonar-scanner-cli-6.2.1.4610-linux-aarch64.zip /tmp/
RUN apt-get update && apt-get install -y unzip && \
    unzip /tmp/sonar-scanner-cli-6.2.1.4610-linux-aarch64.zip -d /opt && \
    ln -s /opt/sonar-scanner-6.2.1.4610-linux-aarch64/bin/sonar-scanner /usr/local/bin/sonar-scanner && \
    rm /tmp/sonar-scanner-cli-6.2.1.4610-linux-aarch64.zip && \
    rm -rf /var/lib/apt/lists/*

# Añadir jenkins al grupo docker
RUN groupadd -f -g 984 docker && usermod -aG docker jenkins

USER jenkins