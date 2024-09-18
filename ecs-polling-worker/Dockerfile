FROM mcr.microsoft.com/dotnet/runtime-deps:6.0

ARG BUILD_NUMBER
ARG BUILD_DATE

# Install pre-req tooling
RUN apt-get update && \
    apt-get install -y \
        curl \
        dos2unix \
        jq \
        sudo \
        gpg \
        unzip \
        wget \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /tmp

# Copy scripts to configure tentacle
COPY --from=octopusdeploy/tentacle:latest /scripts/* /scripts/
RUN chmod +x /scripts/*.sh

# Install Octopus tentacle
# https://octopus.com/downloads/tentacle#linux
RUN curl https://apt.octopus.com/public.key | gpg --dearmor --yes -o /usr/share/keyrings/octopus.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/octopus.gpg] https://apt.octopus.com/ stretch main" | tee /etc/apt/sources.list.d/octopus.list && \
    apt-get update && apt-get install tentacle -y && \
    ln -s /opt/octopus/tentacle/Tentacle /usr/bin/tentacle

# Powershell
# https://learn.microsoft.com/en-us/powershell/scripting/install/install-debian?view=powershell-7.4
RUN wget https://github.com/PowerShell/PowerShell/releases/download/v7.4.5/powershell_7.4.5-1.deb_amd64.deb&& \
    dpkg -i powershell_7.4.5-1.deb_amd64.deb && \
    apt-get install -f && rm powershell_7.4.5-1.deb_amd64.deb

# Get AWS CLI
# https://docs.aws.amazon.com/cli/latest/userguide/install-linux.html#install-linux-awscli
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install && \
    rm awscliv2.zip && \
    rm -rf ./aws

# Get ECS CLI
# https://github.com/aws/amazon-ecs-cli
RUN curl --silent --location "https://amazon-ecs-cli.s3.amazonaws.com/ecs-cli-linux-amd64-latest" -o /usr/local/bin/ecs-cli && \
    chmod +x /usr/local/bin/ecs-cli

# GET eksctl
# https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html
RUN curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp && \
    mv /tmp/eksctl /usr/local/bin

# We know this won't reduce the image size at all. It's just to make the filesystem a little tidier.
RUN rm -rf /tmp/*

# Replicate the environment variables from the tentacle image
ENV OCTOPUS_RUNNING_IN_CONTAINER=Y
ENV ACCEPT_EULA=N
ENV AsKubernetesTentacle=""
ENV BearerToken=""
ENV CustomPublicHostName=""
ENV DISABLE_DIND=N
ENV ListeningPort=""
ENV MachinePolicy="Default Machine Policy"
ENV PublicHostNameConfiguration="ComputerName"
ENV ProxyName=""
ENV ServerApiKey=""
ENV ServerPassword=""
ENV ServerCommsAddress=""
ENV ServerPort=""
ENV ServerUrl=""
ENV ServerUsername=""
ENV Space="Default"
ENV TargetEnvironment=""
ENV TargetName=""
ENV TargetRole=""
ENV TargetTenant=""
ENV TargetTenantTag=""
ENV TargetTenantedDeploymentParticipation=""
ENV TargetWorkerPool=""

ENTRYPOINT [ "/scripts/configure-and-run.sh" ]

LABEL \
    org.label-schema.schema-version="1.0" \
    org.label-schema.name="Octopus ECS Polling Worker" \
    org.label-schema.vendor="harrisonmeister" \
    org.label-schema.url="https://github.com/harrisonmeister" \
    org.label-schema.vcs-url="https://github.com/harrisonmeister/tentacle-images" \
    org.label-schema.license="Apache"  \
    org.label-schema.description="Octopus Tentacle instance configured as a polling worker for use in Amazon ECS" \
    org.label-schema.version=${BUILD_NUMBER} \
    org.label-schema.build-date=${BUILD_DATE}