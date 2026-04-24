FROM ubuntu:24.04

RUN apt-get update && apt-get install -y \
    ansible \
    openssh-client \
    sshpass \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /ansible

CMD ["sleep", "infinity"]