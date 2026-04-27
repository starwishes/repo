FROM alpine:latest

# 1. 安装核心依赖
RUN apk add --no-cache curl jq sqlite bash procps openssl

# 2. 工作目录
WORKDIR /opt/ip_sentinel_master

# 3. 环境变量 (安全性核心)
# IS_OFFICIAL_GATEWAY 设置为 false 允许私有部署，但保留 HMAC 签名校验
ENV TG_TOKEN=""
ENV MASTER_DIR="/opt/ip_sentinel_master"
ENV DB_FILE="/opt/ip_sentinel_master/sentinel.db"
ENV IS_OFFICIAL_GATEWAY="false"
ENV ENABLE_MASTER_OTA="true"

# 4. 启动脚本
RUN cat << 'EOF' > /start.sh
#!/bin/bash
set -e
cd /opt/ip_sentinel_master

echo "🔍 安全对齐：同步云端版本与 master.conf..."
LATEST_VERSION=$(curl -sL "https://raw.githubusercontent.com/hotyue/IP-Sentinel/main/version.txt" | grep "^MASTER_VERSION=" | cut -d'=' -f2 | tr -d '[:space:]')
LATEST_VERSION=${LATEST_VERSION:-"4.0.5"}

cat > /opt/ip_sentinel_master/master.conf << CONF
MASTER_VERSION="${LATEST_VERSION}"
TG_TOKEN="${TG_TOKEN}"
DB_FILE="/opt/ip_sentinel_master/sentinel.db"
MASTER_DIR="/opt/ip_sentinel_master"
IS_OFFICIAL_GATEWAY="false"
ENABLE_MASTER_OTA="true"
CONF

# 修复目录权限，确保 SQLite 能够安全地创建日志锁
chmod -R 777 /opt/ip_sentinel_master

echo "⬇️ 同步主程序逻辑..."
curl -sL https://raw.githubusercontent.com/hotyue/IP-Sentinel/main/master/tg_master.sh -o /opt/ip_sentinel_master/tg_master.sh
chmod +x /opt/ip_sentinel_master/tg_master.sh

echo "🚀 司令部正在安全启动..."
exec /bin/bash /opt/ip_sentinel_master/tg_master.sh
EOF

RUN chmod +x /start.sh
CMD ["/bin/bash", "/start.sh"]
