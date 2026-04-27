FROM alpine:latest

# 安装必要依赖
RUN apk add --no-cache curl jq sqlite bash procps openssl

# 创建工作目录
WORKDIR /opt/ip_sentinel_master

# 下载核心程序脚本 (对应你脚本中第 4 步的操作)
RUN curl -sL https://raw.githubusercontent.com/hotyue/IP-Sentinel/main/master/tg_master.sh -o tg_master.sh && \
    chmod +x tg_master.sh

# 环境变量占位
ENV TG_TOKEN=""
ENV MASTER_DIR="/opt/ip_sentinel_master"
ENV DB_FILE="/opt/ip_sentinel_master/sentinel.db"

# 启动程序
# 注意：由于没有 systemd，我们直接用 bash 运行守护脚本
CMD ["/bin/bash", "tg_master.sh"]