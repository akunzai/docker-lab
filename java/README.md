# Java 開發環境

## 環境需求

- [Docker Engine](https://docs.docker.com/install/)
- [Docker Compose V2](https://docs.docker.com/compose/cli-command/)

## 使用方式

> `docker compose` 指令必須要在 `compose.yml` 所在的目錄下執行
>
> 可透過建立 `compose.override.yml` 來擴展 `compose.yml` 組態
>
> 還可以利用 [COMPOSE_FILE](https://docs.docker.com/compose/reference/envvars/#compose_file) 環境變數指定多個組態來擴展服務配置

```sh
# 啟動並執行完整應用(若配置有異動會自動重建容器)
docker compose up

# 在背景啟動並執行完整應用
docker compose up -d

# 在背景啟動應用時指定服務的執行個數數量
docker compose up -d --scale java=2

# 在背景啟動並執行指定服務
docker compose up -d java

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

- 8080: HTTP

## [啟用 HTTPS 連線](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto.webserver.configure-ssl)

可透過 [mkcert](https://github.com/FiloSottile/mkcert) 建立本機開發用的 TLS 憑證

以網域名稱 `*.dev.local` 為例

```sh
# 安裝本機開發用的憑證簽發證書
mkcert -install

# 產生 TLS 憑證
mkdir -p ../.secrets
mkcert -cert-file ../.secrets/cert.pem -key-file ../.secrets/key.pem '*.dev.local'

# 啟用 TLS 加密連線
COMPOSE_FILE=compose.yml:compose.tls.yml docker compose up -d

# 確認已正確啟用
curl -v 'https://www.dev.local:8443'
```

## 利用容器執行指令

```sh
# 預設執行身分為 root
$ docker compose run --rm java whoami
root

# 指定執行身分為 www-data
$ docker compose run --rm --user www-data java whoami
www-data

# 執行 Bash Shell
$ docker compose run --rm java bash
```

## [自訂和調整](https://learn.microsoft.com/azure/app-service/configure-language-java?pivots=platform-linux#customization-and-tuning)

### 啟動腳本

在本機開發時可以透過 [command](https://docs.docker.com/compose/compose-file/#command) 屬性設定啟動命令

而在 Azure App Service 則可以在組態頁面的一般設定中設定啟動命令

### [設定 Java 執行階段選項](https://learn.microsoft.com/azure/app-service/configure-language-java?pivots=platform-linux#set-java-runtime-options)

如需設定 Java 執行階段選項, 可透過配置 `JAVA_OPTS` 環境變數達成，例如

```sh
JAVA_OPTS=-server -Xmx4g
```

### [Spring Boot 應用程式組態檔配置](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)

如需自訂 Spring Boot 應用程式組態檔位置

```sh
# 透過 Java 屬性自訂組態檔搜尋路徑 (多個路徑以逗號分隔, 後者會優先於前者)
JAVA_OPTS=-Dspring.config.location=classpath:/,file:/home/config/

# 透過環境變數自訂組態檔搜尋路徑 (多個路徑以逗號分隔, 後者會優先於前者)
SPRING_CONFIG_LOCATION=classpath:/,file:/home/config/
```

常用的 Spring Boot 應用程式組態檔配置如下表所示

| 名稱                              | 說明                                                                                                                                |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| spring.config.name                | 組態檔主檔名(用於切換組態, 會嘗試 `.properties` 及 `.yml`,`.yaml` 等副檔案, 預設值: `application`)                                  |
| spring.config.location            | 組態檔搜尋路徑(用於切換組態, 路徑必須以 `/` 結尾, 預設值: `classpath:/,classpath:/config/,file:./,file:./config/*/,file:./config/`) |
| spring.config.additional-location | 額外組態檔的搜尋路徑(用於覆寫組態, 自 2.0 開始支援)                                                                                 |
| spring.config.import              | 額外組態檔的匯入位址(用於覆寫組態, 自 2.4 開始支援)                                                                                 |

## 疑難排解

### [偵錯應用程式](https://www.baeldung.com/spring-debugging)

請配置如下的 `JAVA_OPTS` 環境變數，並開放連接埠 5005 以利偵錯

> 如果 JVM 版本小於 9, address 參數請改為 5005

- JAVA_OPTS: `-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005`

### 如果啟用 HTTPS 後, 如果應用程式無法正確判定 HTTPS 安全連線的話

可以試著在 Spring Boot JAR 應用程式組態檔加入以下配置以支援 [反向代理的 HTTPS 卸載](https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto-use-behind-a-proxy-server)

```ini:application.properties
# before spring-boot 2.2
server.use-forward-headers=true
# since spring-boot 2.2
server.forward-headers-strategy=NATIVE
```

如果你使用的反向代理伺服器並不是使用預設的信任 IP 範圍的話

> Java properties 檔案需要跳脫特殊字元，若使用其它配置形式可將 `\\` 替換為 `\`

```ini:application.properties
server.tomcat.remoteip.trusted-proxies=198\\.19\\.\\d{1,3}\\.\\d{1,3}|35\\.191\\.\\d{1,3}\\.\\d{1,3}
```

### 以非 root 身分執行應用程式

可利用先前提到的自訂啟動腳本，在容器內透過 [gosu](https://github.com/tianon/gosu) 或 [su-exec](https://github.com/ncopa/su-exec) 等工具以非 root 身分執行應用程式

```sh
# for Debian/Ubuntu
apt-get update -qq && apt-get install --no-install-recommends -yqq gosu
gosu www-data java -noverify -Djava.security.egd=file:/dev/./urandom $JAVA_OPTS -jar $JAR_FILE

# for Alpine
apk update && apk add su-exec
su-exec nobody java -noverify -Djava.security.egd=file:/dev/./urandom $JAVA_OPTS -jar $JAR_FILE
```

某些內嵌 Tomcat 應用程式可能會[指定不同的工作目錄](https://github.com/apereo/cas/blob/6.6.x/webapp/cas-server-webapp-resources/src/main/resources/application.properties#L28)
當由 root 身分轉換至非 root 身分時，需一併自動建立這些目錄及設定好權限，否則可能會造成應用程式啟動失敗 (Unable to create the directory [/build/tomcat] to use as the base directory)

```sh
# ensure embedded tomcat working directory exists and fixes permission
mkdir -p /build/tomcat && chown -R www-data:www-data /build/tomcat
```
