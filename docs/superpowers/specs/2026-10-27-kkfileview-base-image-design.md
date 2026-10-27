# kkFileView Base 镜像设计

## 目标

将系统依赖从 kkFileView 应用镜像中拆分为独立的 Docker Base 镜像，避免每次应用构建重复安装 Java、LibreOffice 与字体。Base 镜像发布到 Docker Hub 的 `nibo/kkfileview-base`。

## 镜像边界

Base 镜像基于 `ubuntu:26.04`，保留 Ubuntu 官方默认 apt 源，安装：OpenJDK 21 运行时、LibreOffice 无图形界面版本、fontconfig、开源中文字体（Noto CJK、文泉驿）、替代常用办公字体（Caladea、Carlito、Liberation）、`curl`、中文 locale 与时区数据。镜像生成 `zh_CN.UTF-8` locale 并刷新字体缓存。

Base 镜像不包含 kkFileView 的 tar 包、配置、运行数据目录或应用 `ENTRYPOINT`。应用镜像负责复制发行包、创建 `file/demo` 并提供启动命令。

## 发布与版本

在 `kkFileView-Base-Image` 仓库推送 `v*` Git tag 时触发 GitHub Actions。工作流以 Buildx 构建并推送 `linux/amd64`、`linux/arm64/v8`，使用 Docker Hub Secrets `DOCKERHUB_USERNAME`、`DOCKERHUB_TOKEN` 登录。

每次发布推送两个标签：`nibo/kkfileview-base:latest` 和 `nibo/kkfileview-base:<git-tag>`。主工程默认引用确定的版本标签，并允许通过 `BASE_IMAGE_TAG` 构建参数覆盖；生产构建不依赖可变的 `latest`。

## 主工程改动

根目录 Dockerfile 使用 `ARG BASE_IMAGE_TAG` 与 `FROM nibo/kkfileview-base:${BASE_IMAGE_TAG}`。删除 apt 安装、locale、字体和 LibreOffice 配置层，保留 kkFileView tar 包解压、`/opt/kkFileView-5.0.3/file/demo` 创建、端口与 Java 启动配置。

## 验证

Base 镜像验证 Java、LibreOffice、curl、`zh_CN.UTF-8` 和字体匹配；主工程镜像验证应用启动、`/actuator/health` 与首页读取空 `demo` 目录。
