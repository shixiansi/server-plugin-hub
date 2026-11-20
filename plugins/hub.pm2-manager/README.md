# PM2 管理 (hub.pm2-manager)

v3.0.0 · MIT

管理 Node.js 常驻进程：列表、重启、停止、删除。所有命令都落在白名单模板里，且危险操作默认带二次确认。

## 权限

- `ssh.exec`
- `ui.page`
- `ui.homeWidget`

## 可执行的白名单命令

插件**不能**拼接任意 shell，只能执行下面这些模板：

- `pm2 jlist` — 全部 PM2 应用的 JSON（客户端解析）
- `pm2 restart {{app}}`
- `pm2 stop {{app}}`
- `pm2 delete {{app}}`

---

本文件由 ServerDeck 仓库的 `tools/build_plugin_hub.dart` 生成，请勿手改。