# Based on: https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops#linux

FROM ubuntu:bionic-20200807

# To make it easier for build and release pipelines to run apt-get,
# configure apt to not require confirmation (assume the -y argument by default)
ENV DEBIAN_FRONTEND=noninteractive
RUN echo "APT::Get::Assume-Yes \"true\";" > /etc/apt/apt.conf.d/90assumeyes

# https://github.com/phusion/baseimage-docker/issues/319#issuecomment-590534455
SHELL ["/bin/bash", "-c"]

# ppa:deadsnakes/ppa repo contains more recent Python versions packaged for Ubuntu
# curl and gnupg-agent required to add Docker repo
# software-properties-common provides add-apt-repository
# lxml is required to build ansible-lint-junit wheel
RUN apt-get update \
&& apt-get install --no-install-recommends \
        apt-utils 2> >(grep -v 'debconf: delaying package configuration, since apt-utils is not installed' >&2) \
&& apt-get install --no-install-recommends \
        curl \
        gnupg-agent \
        software-properties-common \
&& curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - \
        2> >(grep -v 'Warning: apt-key output should not be parsed (stdout is not a terminal)' >&2) \
&& add-apt-repository \
        ppa:deadsnakes/ppa \
&& add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
&& apt-get update \
&& apt-get install --no-install-recommends \
        ca-certificates \
        curl \
        jq \
        git \
        iputils-ping \
        libcurl4 \
        libicu60 \
        libunwind8 \
        netcat \
        \
        docker-ce-cli \
        python3.7 \
        python3-pip \
        rubygems \
&& apt-get clean \
\
&& python3.7 -m pip install --upgrade \
        lxml pip setuptools wheel \
&& python3.7 -m pip install --upgrade \
        ansible-lint \
        ansible-lint-junit>=0.16 \
        boto3 \
        jinja2 \
        pylint \
        pylint-fail-under \
        pytest \
        python-json-logger \
        ruamel.yaml \
&& gem install \
        rubocop \
        rubocop-junit_formatter

WORKDIR /azp

# By default Agent.ToolsDirectory=/azp/agent/_work/_tool but /azp/agent is removed in ./start.sh so we set custom path.
# https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/tool/use-python-version?view=azure-devops#how-can-i-configure-a-self-hosted-agent-to-use-this-task
ENV AGENT_TOOLSDIRECTORY=/tools

COPY ./start.sh ./configure-tool-cache.sh ./

RUN chmod +x start.sh configure-tool-cache.sh \
&& ./configure-tool-cache.sh

CMD ["./start.sh"]
