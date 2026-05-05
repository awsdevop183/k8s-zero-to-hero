FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    curl wget dnsutils iproute2 net-tools \
    tcpdump traceroute mtr netcat-openbsd telnet \
    jq vim less htop iputils-ping procps lsof iperf3 openssl \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Install kubectl
RUN curl -LO https://dl.k8s.io/release/v1.29.0/bin/linux/amd64/kubectl && \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && \
    rm kubectl
CMD ["sleep", "infinity"]