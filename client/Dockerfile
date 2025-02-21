FROM ubuntu:20.04 AS base

# Install basic dependencies for analysis 
RUN apt-get update \
  && apt-get install -y software-properties-common wget \
  && apt-get install --no-install-recommends -y \
       clang \
       clang-tidy \
       cppcheck \
       g++-10 \
       gcc-10 \
       cmake \
       build-essential \
       curl \
       gcc-multilib \
       git \
       python3-pip \
       python3-dev \
       python3-venv \
       python3-setuptools \
       libsasl2-dev \
       python-dev \
       libldap2-dev \
       libssl-dev \
       nodejs \
       npm \
 && pip3 install --no-cache-dir wheel virtualenv \
 && pip3 install codechecker \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# Install latest Node.js for codechecker usage
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - && \
    apt-get install -y nodejs
   
# Verify Node.js and npm versions
RUN node -v && npm -v

# Change the compiler versions
RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 100
RUN update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-10 100

# Copy the settings
RUN mkdir codechecker\
WORKDIR /opt/codechecker
COPY /settings /opt/codechecker

# Clone the repository for analysis
FROM base AS clone_repo
RUN git clone -b feature/dune-emperor https://github.com/elcapitanu/dune.git /opt/dune/source 

WORKDIR /opt/dune 
RUN mkdir /opt/dune/build && \
    cd /opt/dune/build && \
    cmake ../source && \ 
    CodeChecker log --build "make -j8" --output /opt/codechecker/compile_commands.json
 
WORKDIR /opt/codechecker
RUN CodeChecker analyze compile_commands.json --config client_config.json -o output || true

FROM clone_repo AS pub_data

CMD ["CodeChecker", "store", "output", "--name=", "--url=http://10.0.0.15:8001/Default"]
