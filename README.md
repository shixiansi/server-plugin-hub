# server-plugin-hub

ServerDeck 的官方插件仓库。App 从这个仓库的 `main` 分支拉取 `index.json`：

```
https://raw.githubusercontent.com/shixiansi/server-plugin-hub/main/index.json
```

## 仓库结构

**每个插件一个独立目录**，`index.json` 只做目录（schema 2）：

```
index.json                        # 只列"有哪些插件、文件在哪"，约 1.8 KB
plugins/
  hub.pm2-manager/
    manifest.json                 # 插件清单（权限、白名单命令）
    page.json                     # 页面 DSL
    README.md                     # 这个插件自己的说明
  hub.docker-manager/
    manifest.json
    page.json
    README.md
  hub.nginx-manager/
    manifest.json
    page.json
    README.md
```

### 为什么不再"全部塞一个 index.json"

早期版本（schema 1）把所有插件内容内联在一个大文件里，问题很明显：

| 问题 | 后果 |
|---|---|
| 改一个插件要重写整个大文件 | `git diff` 全是噪音，review 时看不出改了什么 |
| 无法按插件独立校验/签名 | 一个坏插件污染整份索引，全市场都装不了 |
| 增删插件要改公共文件 | 多人协作必然冲突 |
| 客户端要下 27 KB 才能显示列表 | 列表刷新慢，且失败就是全失败 |

现在换成目录式：改插件只动它自己的目录，`index.json` 基本不变。
客户端先拿索引（1.8 KB），再**并发**拉每个插件，单个失败不影响其他。

## index.json 格式

```jsonc
{
  "schema": 2,
  "generatedAt": "2026-11-19T00:00:00Z",
  "plugins": [
    {
      "id": "hub.pm2-manager",          // 插件唯一 id（反向域名）
      "version": "1.0.0",               // 冗余一份，便于列表直接显示
      "name": { "zh-Hans": "PM2 管理", "en": "PM2 Manager" },
      "summary": { "zh-Hans": "...", "en": "..." },
      "manifest": "plugins/hub.pm2-manager/manifest.json",
      "page":     "plugins/hub.pm2-manager/page.json"
    }
  ]
}
```

> 客户端**仍然兼容 schema 1**（内容内联在索引里），迁移期旧仓库不会突然打不开。

## 怎么生成

不要在手机或网页上手改。用 App 仓库里的脚本生成：

```bash
# 在 App 仓库根目录
dart run tools/build_plugin_hub.dart build/plugin-hub

# 校验（会用和 App 完全相同的 ManifestParser + DslValidator 过一遍）
cd app && flutter test test/plugin_hub_validation_test.dart
```

然后把生成的整个目录结构复制到本仓库根目录提交。

插件的**定义**（唯一事实来源）在 App 仓库的 `tools/plugin_hub_fixtures.dart`。

## 当前收录

| id | 名称 | 白名单命令 |
|---|---|---|
| `hub.pm2-manager` | PM2 管理 | 4 |
| `hub.docker-manager` | Docker 管理 | 7 |
| `hub.nginx-manager` | Nginx 管理 | 5 |

危险操作（stop / delete / reload）在 UI 上带**二次确认**。

## 插件的安全模型

插件**永远不写一整条 shell 字符串**。它只声明「白名单命令模板」——
每个词元（token）都必须落在安全字符集里，变量以 `{{name}}` 整体占位，
且变量类型受限（`string` / `integer` / `enumeration` / `path` / `unit`）。

所以 `;`、`|`、`$()`、反引号**从结构上就写不进来**，不存在命令注入。

```json
{
  "id": "pm2_restart",
  "exec": ["pm2", "restart", "{{app}}"],
  "vars": { "app": { "type": "string", "required": true } },
  "timeout": "30s"
}
```

硬性约束（写错会被校验器拒绝，App 里显示「清单无效」）：

- `exec` 第一个词元必须是可执行文件名，不能是变量；
- 每个词元最多 16 个；
- 词元里**不能有花括号、竖线、引号、空格** —— 所以 docker 的
  `--format '{{.ID}}'` 写不进来，只能用 docker 默认输出；
- 变量名匹配 `^[a-z][a-z0-9_]{0,31}$`；
- `timeout` 必须在 1s–120s。

## DSL 页面的坑（都是被校验器拦下来的）

- `text.value` 只接受**字符串或表达式**，不接受 `{zh-Hans, en}` 本地化对象；
- `form.fields[]` 用 `key`（不是 `name`），`label` 必须是**纯字符串**；
- `when` 的 `then` / `else` 必须是**节点数组** `[ ... ]`；
- `row` 的 `wrap: true` 才会自动换行（否则窄屏溢出）；
- 只能出现 schema 声明过的节点与属性。

## 版本与更新

- 改插件内容**必须同时改 `manifest.json` 的 `version`**，
  也要改 `index.json` 里那一行的 `version`（生成脚本会自动同步）；
- 新增权限属于破坏性变更，用户重装时会看到新增权限项；
- 删插件：删掉目录 + 从 `index.json` 的 `plugins` 移除一行。
  已安装的用户本地仍可用，只是市场里不再出现。
