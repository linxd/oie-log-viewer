# OIE 日志查看器

在管理界面中查看、分页浏览、搜索和下载引擎的 `mirth.log` 及其轮转归档文件，无需访问引擎主机的操作系统。同时支持 Swing 管理界面和 OIE Web 管理界面。

<img src="https://raw.githubusercontent.com/wiki/diridium-com/oie-log-viewer/images/1.png" alt="The Swing viewer: the file list, a page of mirth.log.8.zip with the search match highlighted, and the search results by file" width="800">

<img src="https://raw.githubusercontent.com/wiki/diridium-com/oie-log-viewer/images/2.png" alt="The web administrator's Log Files view: a page of mirth.log.8.zip with the search match highlighted, and the search results" width="800">

## 功能特性

- **自动发现日志** — 根据引擎自身的 log4j 配置查找日志：当前活跃的 `mirth.log` and
  及其轮转的 `.zip` or `.gz` 归档文件，按最新优先排序，包括按日期命名的文件夹中的归档。无需手动输入路径，引擎也不会接受客户端提供的路径。
- **每页 1,000 行** — 带行号和位置提示，如 "Lines 2,001-3,000 of
  11,905"。活跃文件打开时显示最新行，归档文件从第一行开始。
- **原始行尾** LF、CRLF 和 HL7 段落之间的单独 CR 各自作为行尾，段落不会连在一起。一键切换自动换行和特殊字符视图，控制字符和不可读字节显示为可见符号。从查看器复制时保留原始行尾。
- **搜索所有文件** — 先统计每个文件中的匹配行数，然后打开文件's 时每次显示 1,000 行。支持纯文本或 Java 正则表达式，在引擎上逐行匹配，匹配项在当前页面上高亮显示。
- **当前页查找** — 在当前屏幕显示的行中快速查找。
- **下载** — 按磁盘上的原始内容下载任何文件；归档文件以其 zip 格式下载。
- **引擎时间** 时间以引擎所在时区显示，查看器会说明使用的是哪个时区。
- **轮转透明** 活跃文件是一个快照，直到您加载最新行。如果轮转重命名或移除了您正在阅读的文件，查看器会告知您，而不是静默显示不同的文件。

## 在哪里找到

- Swing 管理界面： **Other** 任务面板，**查看日志文件**。
- Web 管理界面： **Log Files** 监控下的 **日志文件**。

## 权限与审计

日志文件是服务器级别的，可能包含来自任何通道的患者数据，因此此插件拥有自己的权限：

|权限|允许操作|
|---|---|
|查看日志文件|列出、读取和搜索日志文件|
|下载日志文件|下载完整日志文件|

On a stock OIE install the default authorization controller allows every operation for every
authenticated user, so both are effectively granted to everyone, as with the rest of the
Administrator. With a role-based authorization controller, such as
[Role Based Access Control](https://github.com/diridium-com/role-based-access-control), grant them
per role: both administrators hide the entry from roles without View Log Files and the Download
button from roles without Download Log Files, and the engine refuses the operations either way.
Roles limited to specific channels are refused outright, because a log file cannot be filtered by
channel reliably.

Every request is recorded in the engine's event log with its parameters (the file, the position,
the search text). Search text is recorded word for word, so anyone who can view events can read
it, and it travels in the request URL, where an HTTP access log on a proxy in front of the engine
would record it too.

## 限制

Searches run on the engine within fixed bounds, so a broad or badly written pattern cannot tie it
up: at most two searches and four page reads at once across the engine, 15 seconds per request (a
count stopped by the limit offers "Count the rest"), and 1,000 matches per batch. A page is at most 256 KB, so a
stretch of very long lines makes a shorter page. Line numbers are given for pages within the first
64 MB of a log file, and the total line count for files up to 64 MB; an archive is numbered
throughout.

## 系统要求

|属性|值|
|-----------|-------|
| OIE version | 4.6.0 (the engine loads an extension only on the version it was built for) |
| Java | 17 |
| Web Administrator | The [Web Support](https://github.com/gibson9583/oie-web-support-plugin) plugin, which serves extension UIs from the engine |

## 安装

从社区商店安装，或从
[Releases](https://github.com/diridium-com/oie-log-viewer/releases) 页面下载 zip，通过管理界面的'扩展视图安装。重启引擎以激活。

## 设计说明

[docs/design-notes.md](docs/design-notes.md) 解释了插件各部分的设计原因，包括一些看似缺失的设计选择。

## 从源码构建

```
./scripts/install-engine-jars.sh
mvn -B -ntp clean package
```

The script installs the four OIE 4.6.0 engine jars the plugin compiles against into your local
Maven repository, taken from the published OIE 4.6.0 distribution and cached under
`~/.cache/oie-dist`. The first build downloads Node.js to build the web administrator UI. The
installable zip is `package/target/oie-log-viewer-<version>.zip`.

## 致谢

Web UI 基于 Chris Gibson 开发的 OIE Web 管理界面及其插件框架构建。

## 许可证

[MPL-2.0](LICENSE). Copyright (c) 2026 Diridium Technologies Inc. 在 Finnegan 狗的精神支持下开发。
