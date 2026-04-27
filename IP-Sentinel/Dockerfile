FROM alpine:latest

# 安装必要依赖
RUN apk add --no-cache curl jq sqlite bash procps openssl

# 创建工作目录
WORKDIR /opt/ip_sentinel_master

# 环境变量占位
ENV TG_TOKEN=""
ENV MASTER_DIR="/opt/ip_sentinel_master"
ENV DB_FILE="/opt/ip_sentinel_master/sentinel.db"

# 【终极防崩写法】明确使用 bash 执行复合命令，并强制挂起进程保活
CMD ["/bin/bash", "-c", "curl -sL https://raw.githubusercontent.com/hotyue/IP-Sentinel/main/master/tg_master.sh -o tg_master.sh && chmod +x tg_master.sh && bash tg_master.sh & tail -f /dev/null"]
