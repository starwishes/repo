FROM alpine:latest

# 安装必要依赖
RUN apk add --no-cache curl jq sqlite bash procps openssl

# 创建工作目录
WORKDIR /opt/ip_sentinel_master

# 环境变量占位
ENV TG_TOKEN=""
ENV MASTER_DIR="/opt/ip_sentinel_master"
ENV DB_FILE="/opt/ip_sentinel_master/sentinel.db"

# 【核心改变】将下载命令移到 CMD 中
# 这样每次容器启动，都会先拉取最新脚本，然后再运行
CMD curl -sL https://raw.githubusercontent.com/hotyue/IP-Sentinel/main/master/tg_master.sh -o tg_master.sh && \
    chmod +x tg_master.sh && \
    /bin/bash tg_master.sh
