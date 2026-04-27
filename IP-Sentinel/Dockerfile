FROM alpine:latest

# 安装必要依赖
RUN apk add --no-cache curl jq sqlite bash procps openssl

# 创建工作目录
WORKDIR /opt/ip_sentinel_master

# 去掉硬编码的版本号，保留核心环境变量
ENV TG_TOKEN=""
ENV MASTER_DIR="/opt/ip_sentinel_master"
ENV DB_FILE="/opt/ip_sentinel_master/sentinel.db"
ENV IS_OFFICIAL_GATEWAY="false"
ENV ENABLE_MASTER_OTA="false"

# 编写启动包裹脚本 (动态获取版本 -> 生成配置 -> 建表 -> 拉代码 -> 运行)
RUN cat << 'EOF' > /start.sh
#!/bin/bash

echo "🔍 正在从云端动态获取最新版本号..."
LATEST_VERSION=$(curl -sL "https://raw.githubusercontent.com/hotyue/IP-Sentinel/main/version.txt" | grep "^MASTER_VERSION=" | cut -d'=' -f2 | tr -d '[:space:]')
LATEST_VERSION=${LATEST_VERSION:-"4.0.0"}
echo "✅ 当前最新版本号为: v${LATEST_VERSION}"

echo "⏳ 正在生成 master.conf 配置文件..."
cat > /opt/ip_sentinel_master/master.conf << CONF
MASTER_VERSION="${LATEST_VERSION}"
TG_TOKEN="${TG_TOKEN}"
DB_FILE="${DB_FILE}"
MASTER_DIR="${MASTER_DIR}"
IS_OFFICIAL_GATEWAY="${IS_OFFICIAL_GATEWAY}"
ENABLE_MASTER_OTA="${ENABLE_MASTER_OTA}"
CONF
chmod 600 /opt/ip_sentinel_master/master.conf

echo "📦 正在初始化 SQLite 数据库表结构..."
sqlite3 /opt/ip_sentinel_master/sentinel.db <<'SQL'
CREATE TABLE IF NOT EXISTS nodes (
    chat_id TEXT, node_name TEXT, agent_ip TEXT, agent_port TEXT,
    last_seen DATETIME DEFAULT CURRENT_TIMESTAMP, region TEXT DEFAULT 'UNKNOWN',
    node_alias TEXT, enable_google TEXT DEFAULT 'true',
    enable_trust TEXT DEFAULT 'true', enable_ota TEXT DEFAULT 'false',
    PRIMARY KEY(chat_id, node_name)
);
CREATE TABLE IF NOT EXISTS ip_trend_log (
    id INTEGER PRIMARY KEY AUTOINCREMENT, node_name TEXT, check_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    scam_score INTEGER, goog_status TEXT, nf_status TEXT, gpt_status TEXT
);
SQL

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
