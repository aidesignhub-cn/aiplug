# Aiplug CLI

AI 设计插件管理工具，用于 AI 设计插件的搜索、查看和安装。

## 安装

```bash
npm i -g @aidesignhub/aiplug                                       # 装成全局命令 aiplug
npx @aidesignhub/aiplug search "把 Figma 设计稿还原成 React 代码"   # 或不装，直接用
```

需要 Node ≥ 22.6。

## 使用

常用命令：

```bash
aiplug search <任务或关键词> [--kind K] [--host H] [--interface I] [--limit 1-20] [--page-token T] [--json]
aiplug show <地址> [--json]                  # 详情；登录后含许可、核验证据与装法；套包地址列出条目
aiplug install <地址> [--host claude-code|codex] # 给出经核验的安装命令（只给出，不执行）；两边通用时可省 --host
aiplug install <地址> [--host H] --dry-run   # 本地计划：不联网、不读凭据、不写文件、不扣额度
aiplug install <地址> --apply                 # 仅 Figma 开发版与 Cursor 项目规则：校验固定提交后写入
# 地址：完整 ID（github:owner/repo…）· owner/repo/name；套包 owner/repo
aiplug login [--email E] [--code-stdin]      # 邮箱验证码登录
aiplug logout [--all] · aiplug whoami · aiplug quota
aiplug contract [--remote] [--json]          # 本机的接口约定；--remote 取服务端的
aiplug list [--json]                         # 本机已安装记录（只读）
aiplug remove <完整 ID> --host H --dry-run # 列出将撤回的路径（只支持 --dry-run，不删文件）
aiplug doctor [<地址>] [--host H] [--project DIR] [--json] # 只读体检；给了地址才联网
aiplug help [<命令>] · aiplug <命令> --help · aiplug --version # 人读，恒为 exit 0，不联网
```

`install --apply` 只放行两类已经核验的原生内容：Figma 开发版保存到用户目录下的 `.aiplug/figma/`，Cursor
项目规则写到当前项目的 `.cursor/rules/`。两类都只从固定提交下载并校验文件，不构建、不运行作者脚本、
不覆盖已有内容；其他条目仍 exit 2（`APPLY_NOT_ENABLED`）。`remove --apply` 尚未开放，恒 exit 2。
`list`、`remove` 与 `doctor` 的记录一节读的是本机安装记录；`install` 目前只给出命令、不执行，所以记录恒为空。
`doctor <地址>` 不靠记录，直接到项目级与用户级的 Claude Code / Codex skill 目录里按条目名查找。

`--format text|json` 是全局开关，可写在子命令前或后；`--json` 是 `--format json` 的简写，二者不能同给（exit 2）。
机读模式下 stdout 恰好一个 JSON 文档，提示走 stderr。

### 给 Agent 用

推荐从标准输入传请求、加 `--json` 取结果：

```bash
echo '{"schemaVersion":"aiplug.discovery.request.v1","task":"设计系统 token 管理","limit":5}' \
  | aiplug search --input - --json
```

`aiplug contract --json` 返回机器可读的接口约定（版本号、退出码、限额、可用的环境变量——只报名字、作用与是否已设，不显示值）；`--remote` 取服务端的约定，主版本不同 exit 3，次版本不同在 stderr 提示。Agent 应据此判断，而不是解析给人看的文字。

### 三条约定

- **默认只给出安装命令。** 只有显式 `install --apply` 且条目属于上述 Figma/Cursor 白名单时，CLI 才写入经校验的文件；其他安装动作不替你执行。
- **`--dry-run` 是本地计划。** 不发任何请求、不扣额度，只说明真正安装会经过哪些步骤、最多扣几个额度；要联网才知道的值（安装命令、本月是否已扣过）标为未知。
- **匿名可用，登录解锁更多。** 未登录能检索；许可详情、核验证据与安装命令需要登录，每月 50 个不同条目。

### 退出码

| 码 | 含义 | 码 | 含义 |
|---|---|---|---|
| 0 | 成功（含零结果） | 5 | 需要登录 |
| 1 | 内部错误 | 6 | 本月额度已用完 |
| 2 | 请求无效 | 7 | 暂无经核验的安装命令 |
| 3 | 服务或服务端目录不可用 | 8 | 请求过于频繁（稍后重试） |
| 4 | 未找到 | | |

机读时看 `error.code`；当前 exit 6 对应 `QUOTA_EXHAUSTED`。

### 配置

| 环境变量 | 作用 |
|---|---|
| `AIPLUG_API_URL` | 覆盖服务地址（默认 `https://api.aiplug.cc/api/v1`） |
| `AIPLUG_CONFIG_DIR` | 覆盖凭据存放目录 |

凭据只存在本机，权限 600；验证码请走 `--code-stdin` 从标准输入给，不要写进命令行参数（会进 shell 历史）。

## 文档

https://aiplug.cc/cli

## License

MIT © 元模因 AI 设计实验室

## 反馈

问题与建议请在 [Issues](https://github.com/aidesignhub-cn/aiplug/issues) 提交。
