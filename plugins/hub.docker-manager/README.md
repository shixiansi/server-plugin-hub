# Docker 管理 (hub.docker-manager)

v3.1.0 · MIT

常用 Docker 运维动作：容器列表、镜像列表、容器日志，以及左滑直接重启 / 停止 / 删除容器（删除带二次确认）。

## 权限

- `ssh.exec`
- `ui.page`

## 可执行的白名单命令

插件**不能**拼接任意 shell，只能执行下面这些模板：

- `docker ps -a --format json`
- `docker images --format json`
- `docker logs --tail {{lines}} {{container}}`
- `docker stop {{container}}`
- `docker restart {{container}}`
- `docker rm {{container}}`

---

本文件由 ServerDeck 仓库的 `tools/build_plugin_hub.dart` 生成，请勿手改。