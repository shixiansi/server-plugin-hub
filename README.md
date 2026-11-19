# server-plugin-hub

ServerDeck 的官方插件仓库。App 会从这个仓库的 `main` 分支拉取 `index.json`：

```
https://raw.githubusercontent.com/shixiansi/server-plugin-hub/main/index.json
```

## 这个仓库里要放什么

只需要一个文件：**`index.json`**（放进仓库根目录即可）。

```jsonc
{
  "schema": 1,
  "generatedAt": "2026-11-19T00:00:00Z",
  "plugins": [
    {
      "manifest": { /* 插件清单，见下 */ },
      "page":     { /* DSL 页面（dsl/1） */ },
      "summary":  { "zh-Hans": "一句话卖点", "en": "..." }
    }
  ]
}
```

`summary` 可选，只用于市场列表的副标题；插件的正式描述在 `manifest.description`。

## 怎么生成（推荐）

不要在手机上/网页里手改这段 JSON。它由 App 仓库里的一个脚本生成，
脚本同时被单元测试用来校验清单合法性：

```bash
# 在 App 仓库根目录
dart run tools/build_plugin_hub.dart build/plugin-hub
# → build/plugin-hub/index.json
```

然后把 `build/plugin-hub/index.json` 复制到本仓库根目录提交即可。

插件的**定义**在 `tools/plugin_hub_fixtures.dart`，是整个仓库的唯一事实来源：
生成 index 和跑测试用的是同一份代码，不会漂移。

## 当前收录的插件

| id | 名称 | 说明 | 白名单命令 |
|---|---|---|---|
| `hub.pm2-manager` | PM2 管理 | 列表 / 重启 / 停止 / 删除 | 4 |
| `hub.docker-manager` | Docker 管理 | 容器与镜像列表 / 日志 / 启停 / 重启 / 删除 | 7 |
| `hub.nginx-manager` | Nginx 管理 | 配置检查 / 重载 / 版本 / 状态 / 访问日志 | 5 |

危险操作（stop / delete / reload）在 UI 上都带**二次确认**。

## 插件的安全模型（为什么这些命令是安全的）

插件**永远不写一整条 shell 字符串**。它只声明「白名单命令模板」——
每个词元（token）都必须落在安全字符集里，变量以 `{{name}}` 整体占位，
且变量的取值类型受限（`string` / `integer` / `enumeration` / `path` / `unit`）。

所以 `;`、`|`、`$()`、反引号这些**从结构上就写不进来**，不存在命令注入。

一个模板长这样：

```json
{
  "id": "pm2_restart",
  "exec": ["pm2", "restart", "{{app}}"],
  "vars": { "app": { "type": "string", "required": true } },
  "timeout": "30s"
}
```

几条硬性约束（写错会被校验器拒绝，App 里显示「清单无效」）：

- `exec` 的第一个词元必须是可执行文件名，不能是变量；
- 每个词元最多 16 个；
- 词元里**不能有花括号、竖线、引号、空格**——所以 docker 的
  `--format '{{.ID}}'` 这种 go-template 写不进来，只能用 docker 默认输出；
- 变量名必须匹配 `^[a-z][a-z0-9_]{0,31}$`；
- `timeout` 必须在 1s–120s。

## DSL 页面的几条坑

这些是实际踩过、并且被 `DslValidator` 拦下来的：

- `text.value` 只接受**字符串或表达式**，**不接受** `{zh-Hans, en}` 本地化对象；
- `form.fields[]` 用 `key`（不是 `name`），且 `label` 必须是**纯字符串**；
- `when` 的 `then` / `else` 必须是**节点数组** `[ ... ]`，不是单个节点；
- 节点里只能出现 schema 声明过的属性，多写一个都会被判错。

改完务必跑一遍校验（在 App 仓库里）：

```bash
cd app && flutter test test/plugin_hub_validation_test.dart
```

这条测试会用和 App 完全相同的 `ManifestParser` + `DslValidator`
把 `index.json` 里的每个插件过一遍，比推上去再回手机上试快得多。

## 版本与更新

- 改插件内容时**必须同时改 `manifest.version`**，App 用版本号判断是否需要更新；
- 新增权限属于**破坏性变更**，用户重装时会看到新增的权限项；
- 删插件时直接从 `index.json` 的 `plugins` 里移除即可，
  已安装的用户本地仍然能用（只是市场里不再出现）。
