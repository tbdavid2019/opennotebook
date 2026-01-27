# Open WebUI Nginx 設定指南

本文件說明如何為 Open WebUI 配置 Nginx，包含 WebSocket 支援和檔案上傳大小限制。

## 問題背景

Open WebUI 需要以下 Nginx 配置才能正常運作：
1. **WebSocket 支援**：用於即時通訊功能
2. **檔案上傳大小限制**：預設 Nginx 只允許 1MB，需要調整以支援大檔案上傳

如果沒有正確配置，可能會遇到：
- `Unexpected token '<', "<html> <h"... is not valid JSON` 錯誤
- HTTP 413 Request Entity Too Large 錯誤
- WebSocket 連線失敗

---

## 配置步驟

### 1. 編輯全域 Nginx 配置

編輯 `/etc/nginx/nginx.conf`，在 `http` 區塊內加入 WebSocket 映射：

```nginx
http {
    # ... 現有配置 ...
    
    # WebSocket upgrade mapping
    map $http_upgrade $connection_upgrade {
        default upgrade;
        ''      close;
    }
    
    # ... 其他配置 ...
}
```

**位置建議**：放在 `include /etc/nginx/mime.types;` 之前

### 2. 編輯站點配置檔案

編輯你的站點配置檔（例如 `/etc/nginx/conf.d/gpt.conf`）：

```nginx
server {
    server_name your-domain.com;
    
    # 設定檔案上傳大小限制（根據需求調整）
    client_max_body_size 250M;

    location / {
        proxy_pass http://127.0.0.1:8888;  # Open WebUI 的 port
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket 支援
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        
        # SSE (Server-Sent Events) 支援
        proxy_buffering off;
        proxy_cache off;
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
        
        # 確保不會中斷長連接
        keepalive_timeout 86400s;
    }

    # SSL 配置（如果有使用 Certbot）
    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}

# HTTP to HTTPS 重定向
server {
    if ($host = your-domain.com) {
        return 301 https://$host$request_uri;
    }

    listen 80;
    server_name your-domain.com;
    return 404;
}
```

### 3. 驗證並重新載入配置

```bash
# 測試配置是否正確
sudo nginx -t

# 重新載入 Nginx
sudo systemctl reload nginx
```

---

## 重要注意事項

### ⚠️ client_max_body_size 位置

`client_max_body_size` **必須放在 `server` 區塊內**，不能放在外面，否則會出現 "duplicate directive" 錯誤。

**正確：**
```nginx
server {
    client_max_body_size 250M;  # ✅ 正確位置
    location / { ... }
}
```

**錯誤：**
```nginx
client_max_body_size 250M;  # ❌ 錯誤：在 server 區塊外
server {
    location / { ... }
}
```

### 📏 檔案大小限制建議

- **100MB**：一般用途（文件、圖片、PDF）
- **250MB**：支援較大檔案（簡報、壓縮檔）
- **500MB - 1GB**：專業用途（影片、大型資料集）

**考量因素：**
1. **磁碟空間**：Nginx 會將上傳檔案暫存到 `/var/lib/nginx/tmp/client_body/`
2. **記憶體使用**：大檔案上傳會佔用記憶體
3. **安全性**：過大的限制可能導致 DoS 攻擊風險
4. **後端限制**：確保 Open WebUI 也支援相同大小

### 🔌 WebSocket 配置說明

使用 `map` 指令的好處：
- 只在需要時才升級連線（當 `Upgrade` header 存在時）
- 沒有 `Upgrade` header 時會正常關閉連線
- 這是 Nginx 官方推薦的做法

### ⏱️ 超時設定

```nginx
proxy_read_timeout 86400s;   # 24 小時
proxy_send_timeout 86400s;   # 24 小時
keepalive_timeout 86400s;    # 24 小時
```

這些設定確保長時間的 AI 對話不會被中斷。如果你的使用情境不需要這麼長，可以調整為：
- 一般用途：`3600s`（1 小時）
- 短時間互動：`600s`（10 分鐘）

---

## 驗證配置

### 檢查 WebSocket 連線

在瀏覽器開發者工具的 Network 標籤中，應該看到：
- `/ws/socket.io/` 請求
- Status: `101 Switching Protocols`

### 檢查檔案上傳

嘗試上傳檔案，確認：
- 不會出現 413 錯誤
- 不會出現 JSON 解析錯誤

### 檢查 Nginx 日誌

```bash
# 查看錯誤日誌
sudo tail -f /var/log/nginx/error.log

# 查看訪問日誌
sudo tail -f /var/log/nginx/access.log
```

---

## 常見問題排查

### 問題 1: nginx -t 失敗，顯示 "duplicate directive"

**原因**：`client_max_body_size` 放在錯誤位置或重複定義

**解決方法**：
```bash
# 檢查是否有重複
grep -n "client_max_body_size" /etc/nginx/conf.d/*.conf

# 刪除所有舊的設定
sudo sed -i '/client_max_body_size/d' /etc/nginx/conf.d/your-site.conf

# 重新加入到正確位置（server 區塊內第一行）
```

### 問題 2: WebSocket 連線失敗

**檢查項目**：
1. 確認 `map $http_upgrade $connection_upgrade` 在 `nginx.conf` 的 `http` 區塊內
2. 確認站點配置有 `proxy_set_header Connection $connection_upgrade;`
3. 確認後端服務（Open WebUI）正在運行

### 問題 3: 仍然出現 413 錯誤

**可能原因**：
1. 配置沒有重新載入：`sudo systemctl reload nginx`
2. `client_max_body_size` 設定太小
3. 後端應用也有檔案大小限制

---

## 快速部署指令

如果你要在新機器上快速部署，可以使用以下指令：

```bash
# 1. 備份現有配置
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup
sudo cp /etc/nginx/conf.d/your-site.conf /etc/nginx/conf.d/your-site.conf.backup

# 2. 在 nginx.conf 的 http 區塊加入 map（手動編輯或使用 sed）
sudo sed -i '/types_hash_max_size 4096;/a \    # WebSocket upgrade mapping\n    map $http_upgrade $connection_upgrade {\n        default upgrade;\n        '\''      close;\n    }' /etc/nginx/nginx.conf

# 3. 更新站點配置中的 Connection header
sudo sed -i 's/proxy_set_header Connection "upgrade";/proxy_set_header Connection $connection_upgrade;/g' /etc/nginx/conf.d/your-site.conf

# 4. 測試並重新載入
sudo nginx -t && sudo systemctl reload nginx
```

---

## 參考資料

- [Nginx WebSocket Proxying](http://nginx.org/en/docs/http/websocket.html)
- [Nginx client_max_body_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size)
- [Open WebUI Documentation](https://docs.openwebui.com/)

---

**最後更新**：2026-01-27  
**適用版本**：Nginx 1.18+, Open WebUI (Docker)
