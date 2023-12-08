FROM golang:1.18

LABEL maintainer="Ving <ving@nivin.cn>"

ENV GOPROXY https://goproxy.cn,direct

# 安装必要的软件包和依赖包
USER root
RUN sed -i 's/deb.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list && \
    sed -i 's/security.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list && \
    sed -i 's/security-cdn.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list && \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
    curl \
    zip \
    unzip \
    git \
    vim 

###########################################################################
# Set Timezone
###########################################################################

ARG TZ=PRC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 添加开发账户
USER root
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}
RUN set -xe; \
  groupadd -g ${PGID} developer && \
  useradd -l -u ${PUID} -g developer -m developer && \
  usermod -p "*" developer -s /bin/bash


# 安装 goctl
USER developer
RUN GOPROXY=https://goproxy.cn/,direct

# 安装 protoc
# USER developer
# COPY ./protoc-3.19.1-linux-x86_64.zip /tmp/protoc.zip
# RUN unzip -d /tmp/protoc /tmp/protoc.zip && \
#     mv /tmp/protoc/bin/protoc $GOPATH/bin

# 安装 protoc-gen-go
# USER developer
# RUN go install github.com/golang/protobuf/protoc-gen-go@v1.4.0

# $GOPATH/bin添加到环境变量中
ENV PATH $GOPATH/bin:$PATH

# 清理垃圾
USER root
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

# 设置工作目录
WORKDIR /var/www

EXPOSE 17001
EXPOSE 17002
EXPOSE 17003 