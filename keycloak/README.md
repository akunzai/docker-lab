# [Keycloak](https://www.keycloak.org/) 身分認證與存取管理系統

## 環境需求

- [Docker Engine](https://docs.docker.com/install/)
- [Docker Compose V2](https://docs.docker.com/compose/cli-command/)

## 使用方式

> `docker compose` 指令必須要在 `docker-compose.yml` 所在的目錄下執行
>
> 可透過建立 `docker-compose.override.yml` 來擴展 `docker-compose.yml` 組態
>
> 還可以利用 [COMPOSE_FILE](https://docs.docker.com/compose/reference/envvars/#compose_file) 環境變數指定多個組態來擴展服務配置

```sh
# 啟動並執行完整應用
docker compose up

# 在背景啟動並執行完整應用
docker compose up -d

# 在背景啟動並執行指定服務
docker compose up -d keycloak

# 在背景啟動並擴展指定服務的容器個數
docker compose up -d --scale keycloak=2

# 顯示記錄
docker compose logs

# 持續顯示記錄
docker compose logs -f

# 關閉應用
docker compose down

# 顯示所有啟動中的容器
docker ps
```

## 連線埠配置

啟動環境後預設會開始監聽本機的以下連線埠

- 80: HTTP
- 443: HTTPS
- 9090: Traefik 負載平衡器管理後台

## [啟用 HTTPS 連線](https://doc.traefik.io/traefik/https/tls/)

### [使用 Let's Encrypt 自動產生憑證](https://doc.traefik.io/traefik/https/acme/)

### 建立本機開發用的 TLS 憑證

```sh
# 設定本機開發用的主機名稱及 TLS 憑證
./init.sh auth.dev.local
```

## 匯入與匯出

如果要匯出現有 Keycloak 的所有配置的話

```sh
docker compose exec keycloak bin/kc.sh export --dir /opt/keycloak/data/export/
```

如果要匯入 Keycloak 配置的話, 請將檔案放置在專案目錄的 [./import](./import/) 目錄下，在啟動 keycloak 容器時便會自動匯入(會略過已存在的 realm)