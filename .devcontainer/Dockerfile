# syntax=docker/dockerfile:1.4

FROM ubuntu:25.04

ENV DEBIAN_FRONTEND=noninteractive \
    SHELL=/bin/bash \
    DOCKER_BUILDKIT=1

# Create user 'codespace'
ARG USERNAME=codespace
ARG USER_UID=1000
ARG USER_GID=1000

RUN apt-get update && \
    apt-get install -y sudo && \
    groupadd --gid ${USER_GID} ${USERNAME} && \
    useradd --uid ${USER_UID} --gid ${USER_GID} -m ${USERNAME} && \
    echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/90-${USERNAME} && \
    chmod 0440 /etc/sudoers.d/90-${USERNAME}

# Merge layers to reduce image size
RUN apt-get update && \
    yes | unminimize 2>&1 && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
        make unzip build-essential swig3.0 unixodbc-dev libpq-dev \
        default-libmysqlclient-dev moreutils rsync zip libgdiplus jq \
        python3-pip libc6 libgcc1 libgssapi-krb5-2 libncurses6 liblttng-ust1 \
        libssl-dev libstdc++6 zlib1g libuuid1 libunwind8 sqlite3 \
        libsqlite3-dev software-properties-common tk-dev uuid-dev \
        curl gettext inotify-tools cmake cppcheck valgrind clang lldb llvm gdb \
        python3-dev vim vim-doc xtail libsecret-1-dev fish \
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/*

# Fish shell prompt customization
RUN printf "function fish_prompt\n  set_color green\n  echo -n (whoami)\n  set_color normal\n  echo -n \":\"\n  set_color blue\n  echo -n (pwd)\n  set_color normal\n  echo -n \"> \"\nend\n" > /etc/fish/functions/fish_prompt.fish && \
    printf "if type code-insiders > /dev/null 2>&1; and not type code > /dev/null 2>&1\n  alias code=code-insiders\nend\n" > /etc/fish/conf.d/code_alias.fish

# Install libssl1.1 for oryx compatibility based on architecture
RUN ARCH=$(uname -m) && \
    if [ "$ARCH" = "x86_64" ] || [ "$ARCH" = "amd64" ]; then \
        curl -fsSL -o libssl1.1.deb http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb && \
        dpkg -i libssl1.1.deb && rm libssl1.1.deb; \
    elif [ "$ARCH" = "aarch64" ] || [ "$ARCH" = "arm64" ]; then \
        curl -fsSL -o libssl1.1.deb http://ports.ubuntu.com/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2_arm64.deb && \
        dpkg -i libssl1.1.deb && rm libssl1.1.deb; \
    else \
        echo "Unsupported architecture: $ARCH" && exit 1; \
    fi

# Enable Docker volume for Docker-in-Docker
VOLUME ["/var/lib/docker"]

# Default container command
CMD ["sleep", "infinity"]
