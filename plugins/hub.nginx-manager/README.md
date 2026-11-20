# Nginx 管理 (hub.nginx-manager)

v2.0.0 · MIT

配置检查 / 重载 / 状态 / 日志。重载前建议先跑一次配置检查，本插件也提供一键"检查后重载"。

## 权限

- `ssh.exec`
- `ui.page`
- `sftp.read`

## 可执行的白名单命令

插件**不能**拼接任意 shell，只能执行下面这些模板：

- `nginx -t` — 检查配置文件语法
- `nginx -s reload`
- `nginx -v`
- `systemctl is-active nginx`
- `tail -n {{lines}} /var/log/nginx/access.log`

---

本文件由 ServerDeck 仓库的 `tools/build_plugin_hub.dart` 生成，请勿手改。