FROM jenkins/jenkins:lts

USER root

# GID del grupo docker del HOST (se pasa desde docker-compose)
ARG DOCKER_GID=984
ARG SONAR_SCANNER_VERSION=6.2.1.4610

# Dependencias básicas
RUN apt-get update && apt-get install -y \
    curl ca-certificates gnupg unzip \
    && rm -rf /var/lib/apt/lists/*

# Repositorio oficial de Docker para la arquitectura de la máquina
RUN install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc && \
    chmod a+r /etc/apt/keyrings/docker.asc && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/debian $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
    > /etc/apt/sources.list.d/docker.list

# Docker CLI + buildx (necesario para construir imágenes arm64 desde x86)
RUN apt-get update && apt-get install -y \
    docker-ce-cli docker-buildx-plugin \
    && rm -rf /var/lib/apt/lists/*

# kubectl para la arquitectura de la máquina
RUN ARCH=$(dpkg --print-architecture) && \
    curl -fsSLo /usr/local/bin/kubectl \
      "https://dl.k8s.io/release/$(curl -fsSL https://dl.k8s.io/release/stable.txt)/bin/linux/${ARCH}/kubectl" && \
    chmod +x /usr/local/bin/kubectl

# sonar-scanner: Sonar llama "x64" a amd64 y "aarch64" a arm64
RUN case "$(dpkg --print-architecture)" in \
      amd64) SONAR_ARCH=x64 ;; \
      arm64) SONAR_ARCH=aarch64 ;; \
    esac && \
    curl -fsSLo /tmp/sonar.zip \
      "https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-${SONAR_SCANNER_VERSION}-linux-${SONAR_ARCH}.zip" && \
    unzip -q /tmp/sonar.zip -d /opt && \
    ln -s /opt/sonar-scanner-${SONAR_SCANNER_VERSION}-linux-${SONAR_ARCH} /opt/sonar-scanner && \
    ln -s /opt/sonar-scanner/bin/sonar-scanner /usr/local/bin/sonar-scanner && \
    rm /tmp/sonar.zip

# Helm (el script oficial detecta la arquitectura solo)
RUN curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Grupo docker con el MISMO GID que en el host, para poder usar /var/run/docker.sock
RUN groupadd -f -g ${DOCKER_GID} docker && usermod -aG docker jenkins

USER jenkins
