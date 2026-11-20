# Nginx 管理 (hub.nginx-manager)

v3.1.0 · MIT

配置检查 / 版本 / 状态 / 重载 / 访问日志。重载前请先点一次「检查配置」，配置有语法错误时重载会让服务异常。

## 权限

- `ssh.exec`
- `ui.page`
- `sftp.read`

## 可执行的白名单命令

插件**不能**拼接任意 shell，只能执行下面这些模板：

- `nginx -t` — 检查配置文件语法（结果在 stderr）
- `nginx -s reload`
- `nginx -v`
- `systemctl is-active nginx`
- `tail -n {{lines}} /var/log/nginx/access.log`

---

本文件由 ServerDeck 仓库的 `tools/build_plugin_hub.dart` 生成，请勿手改。