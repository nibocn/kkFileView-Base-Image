# kkFileView 基础镜像

`nibo/kkfileview-base` 是为 kkFileView 文档预览场景准备的运行环境基础镜像。

镜像基于 Ubuntu 26.04，并使用其默认 APT 软件源。镜像中已包含：

- Java 21 运行时（headless JRE）
- LibreOffice（`nogui`）
- `curl`
- `zh_CN.UTF-8` Locale 和 `Asia/Shanghai` 时区
- Noto CJK、文泉驿字体，以及 Caladea、Carlito、Liberation 等可开源使用的替代字体

本镜像**不包含** kkFileView 应用，也不定义应用启动入口。请在基于此镜像构建的业务镜像中复制 kkFileView 产物并自行设置 `ENTRYPOINT` 或 `CMD`。

## 本地构建与验证

构建镜像：

```bash
docker build -t nibo/kkfileview-base:local .
```

验证 Java、LibreOffice、Locale、时区和字体替代映射：

```bash
docker run --rm nibo/kkfileview-base:local sh -c \
  'java -version && libreoffice --version && locale && cat /etc/timezone && fc-match SimSun && fc-match "WenQuanYi Zen Hei"'
```

## 发布

GitHub Actions 仅在推送 `v*` 标签时触发，构建并发布 `linux/amd64` 与 `linux/arm64/v8` 多架构镜像。

标签必须符合 `vX.Y.Z`，或在其后附加预发布后缀（例如 `v26.04.0-rc.1`）。每个通过校验的版本标签都会发布同名镜像标签：

```bash
git tag v26.04.0
git push origin v26.04.0
```

稳定版 `vX.Y.Z` 在发布其精确版本后，还会将 `nibo/kkfileview-base:latest` 更新到该版本；预发布标签不会更新 `latest`。

工作流需要在 GitHub 仓库中配置以下 Secrets：

- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`
