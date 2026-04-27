FROM alpine:latest

# 1. 安装系统必要依赖
RUN apk add --no-cache curl jq sqlite bash procps openssl

# 2. 设置工作目录
WORKDIR /opt/ip_sentinel_master

# 3. 定义环境变量默认值（具体值请在 Dokploy 面板的 Environment 中配置）
ENV TG_TOKEN=""
ENV MASTER_DIR="/opt/ip_sentinel_master"
ENV DB_FILE="/opt/ip_sentinel_master/sentinel.db"
ENV IS_OFFICIAL_GATEWAY="false"
ENV ENABLE_MASTER_OTA="false"

# 4. 编写集成启动脚本 (start.sh)
RUN cat << 'EOF' > /start.sh
#!/bin/bash
set -e

echo "🔍 正在从云端动态获取最新版本号..."
LATEST_VERSION=$(curl -sL "https://raw.githubusercontent.com/hotyue/IP-Sentinel/main/version.txt" | grep "^MASTER_VERSION=" | cut -d'=' -f2 | tr -d '[:space:]')
LATEST_VERSION=${LATEST_VERSION:-"4.0.5"}
echo "✅ 当前版本将对齐为: v${LATEST_VERSION}"

echo "⏳ 正在生成/更新 master.conf 配置文件..."
cat > /opt/ip_sentinel_master/master.conf << CONF
MASTER_VERSION="${LATEST_VERSION}"
TG_TOKEN="${TG_TOKEN}"
DB_FILE="${DB_FILE}"
MASTER_DIR="${MASTER_DIR}"
IS_OFFICIAL_GATEWAY="${IS_OFFICIAL_GATEWAY}"
ENABLE_MASTER_OTA="${ENABLE_MASTER_OTA}"
CONF
chmod 600 /opt/ip_sentinel_master/master.conf

echo "📦 检查并初始化数据库..."
if [ ! -f "$DB_FILE" ]; then
    echo "⚠️ 未发现现有数据库，正在执行初始化建表..."
    sqlite3 "$DB_FILE" <<'SQL'
CREATE TABLE IF NOT EXISTS nodes (
    chat_id TEXT, node_name TEXT, agent_ip TEXT, agent_port TEXT,
    last_seen DATETIME DEFAULT CURRENT_TIMESTAMP, region TEXT DEFAULT 'UNKNOWN',
    node_alias TEXT, enable_google TEXT DEFAULT 'true',
    enable_trust TEXT DEFAULT 'true', enable_ota TEXT DEFAULT 'false',
    PRIMARY KEY(chat_id, node_name)
);
CREATE TABLE IF NOT EXISTS ip_trend_log (
    id INTEGER PRIMARY KEY AUTOINCREMENT, node_name TEXT, check_time DATETIME DEFAULT CURRENT_TIMESTAMP, scam_score INTEGER, goog_status TEXT, nf_status TEXT, gpt_status TEXT
);
SQL
    echo "✅ 数据库表结构初始化完成。"
else
    echo "✅ 发现现有数据库文件，跳过初始化，保持数据持久化。"
fi

# 修复可能存在的权限问题
chmod -R 777 /opt/ip_sentinel_master

echo "⬇️ 正在拉取最新的 tg_master.sh 核心代码..."
curl -sL https://raw.githubusercontent.com/hotyue/IP-Sentinel/main/master/tg_master.sh -o /opt/ip_sentinel_master/tg_master.sh
chmod +x /opt/ip_sentinel_master/tg_master.sh

echo "🚀 启动 IP-Sentinel 控制中枢..."
exec /bin/bash /opt/ip_sentinel_master/tg_master.sh

# 权限双重保险：确保中枢有权写入挂载目录
echo "🛡️ 正在确保数据库目录权限..."
chmod -R 777 /opt/ip_sentinel_master

# 环境清理：删除可能导致启动失败的旧日志锁文件
rm -f /opt/ip_sentinel_master/*.log.lock

echo "🚀 启动 IP-Sentinel 控制中枢..."

# 智能启动
# 使用 exec 确保日志直接输出到控制台（Dokploy 面板）
# 同时加上 -u 参数可以强制刷新缓冲区，防止日志显示延迟
exec /bin/bash /opt/ip_sentinel_master/tg_master.sh

EOF

# 5. 赋予启动脚本执行权限
RUN chmod +x /start.sh

# 6. 容器启动入口
CMD ["/bin/bash", "/start.sh"]
