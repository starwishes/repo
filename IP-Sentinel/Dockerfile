FROM alpine:latest

# 安装必要依赖
RUN apk add --no-cache curl jq sqlite bash procps openssl

# 创建工作目录
WORKDIR /opt/ip_sentinel_master

# 定义环境变量默认值（真实值在 Dokploy 面板填）
ENV TG_TOKEN=""
ENV MASTER_DIR="/opt/ip_sentinel_master"
ENV DB_FILE="/opt/ip_sentinel_master/sentinel.db"
ENV MASTER_VERSION="4.0.0"
ENV IS_OFFICIAL_GATEWAY="false"
ENV ENABLE_MASTER_OTA="false"

# 编写启动包裹脚本 (生成配置 -> 下载最新代码 -> 运行)
RUN cat << 'EOF' > /start.sh
#!/bin/bash
echo "⏳ 正在生成 master.conf 配置文件..."
cat > /opt/ip_sentinel_master/master.conf << CONF
MASTER_VERSION="${MASTER_VERSION}"
TG_TOKEN="${TG_TOKEN}"
DB_FILE="${DB_FILE}"
MASTER_DIR="${MASTER_DIR}"
IS_OFFICIAL_GATEWAY="${IS_OFFICIAL_GATEWAY}"
ENABLE_MASTER_OTA="${ENABLE_MASTER_OTA}"
CONF
chmod 600 /opt/ip_sentinel_master/master.conf

echo "⬇️ 正在拉取最新中枢代码..."
curl -sL https://raw.githubusercontent.com/hotyue/IP-Sentinel/main/master/tg_master.sh -o /opt/ip_sentinel_master/tg_master.sh
chmod +x /opt/ip_sentinel_master/tg_master.sh

echo "🚀 启动控制中枢..."
exec bash /opt/ip_sentinel_master/tg_master.sh
EOF

# 赋予执行权限
RUN chmod +x /start.sh

# 容器启动命令
CMD ["/start.sh"]
