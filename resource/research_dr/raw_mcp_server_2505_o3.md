以下列表整理了**当前全球范围内最常被用来发现 MCP（模型上下文协议，Model Context Protocol）服务器的 10 个线上平台 / 注册中心 / 目录**，并按照综合热度（托管/列出的服务器数量、GitHub Stars、社区讨论量、更新频率等指标）由高到低排序。开头先给你一个快速结论：**如果你只想找“最多、最新、社区最活跃”的 MCP 服务器，直接去 PulseMCP；若想看「官方+社区」参考实现，GitHub 上的 modelcontextprotocol/servers 和官方文档站的 Example Servers 页面是最权威的入口；在命令行里安装/升级，mcp-get CLI 是事实上的“npm/pip”式仓库；剩下的平台则各有侧重，适合进阶筛选或自动化集成。**

---

## 1. PulseMCP (pulsemcp.com)

* **规模 & 活跃度**：截至 2025-05-12 收录 **4258 个公开 MCP 服务器** ，并每天增量抓取更新。页面自带「趋势、分类、远程可用」等过滤器，对新手非常友好。
* **为何重要**：目前已成为开发者在博客、教程中最常贴出的“综合目录”，也是多数演示文章的首选数据源。
* **加分点**：内置 Discord 社区、周榜/月榜、服务器健康检查，可快速看到哪台服务器失联。

---

## 2. GitHub · modelcontextprotocol/servers 仓库

* **规模 & 认可度**：官方 & 社区共同维护的“参考实现和索引”，**45.4 k Stars / 5.1 k Forks**，活跃 Issue 超 270 条。
* **为何重要**：对任何想自己部署或学习 MCP 的人而言，这里是“源代码、脚手架、PR 讨论”三位一体的中心；几乎所有其它目录都会反向引用此仓库。

---

## 3. 官方文档站 Example Servers 页面

* **内容**：将官方参考服务器、各大厂商“官方集成”以及精选社区服务器做了分类展示（数据 / 开发工具 / 浏览器自动化等）。
* **为何重要**：官方文档域名，信息更新与协议版本同步；每个条目直接链到对应 GitHub 项目或 npm / PyPI 包，方便跟进代码。

---

## 4. mcp-get CLI + Registry

* **定位**：类似于 Homebrew / npm 的多语言包管理器；既是 CLI，也托管**集中式注册表**（mcp-get.com）。
* **人气指标**：仓库 **401 Stars**，超过 400 次提交；所有包同步到网页可视化目录。([github.com][1])
* **亮点**：一句 `npx @michaellatman/mcp-get install <server>` 即可下载并配置环境变量，适合自动化流水线。

---

## 5. Awesome MCP Servers（wong2/awesome-mcp-servers）

* **定位**：纯手工维护的“精选 + 说明 + 标签”列表，**1.1 k Stars**。
* **优势**：条目带简评，删除了长期无人维护的仓库；适合快速了解“哪些服务器最被点赞”。

---

## 6. MCP Compass（mcphub.io / GitHub liuyoshio/mcp-compass）

* **功能**：自然语言搜索 + 推荐引擎，可输入“浏览器自动化”就返回对应服务器。
* **社区指标**：**93 Stars**，网站演示版提供实时检索。
* **适用场景**：当你只知道想做的任务，却不知道服务器名字时，用它比纯目录更高效。

---

## 7. MCP Registry Service（modelcontextprotocol/registry）

* **定位**：官方团队打造的 **RESTful API** 注册服务，目标是成为“机器可读”的中央注册表。当前 **105 Stars**，仍在早期迭代。
* **意义**：未来可能被各大 IDE、云服务用来自动发现 MCP 工具；想在后端集成可预先关注。

---

## 8. MCP.so （“Featured MCP Servers” 站点）

* **简介**：较早期的可视化展示站，提供“EdgeOne Pages MCP”“Playwright MCP”等精选示例。
* **现状**：近期访问偶有波动（Cloudflare 错误），但在论坛/博客仍常被引用。

---

## 9. MCP Server Finder (mcpserverfinder.com)

* **特色**：定位为“一站式搜索 + 能力标签 + 安全说明”的目录，主打易用 UI。
* **不足**：目前条目数不足千级，且缺少社区热度排序。适合作为补充。

---

## 10. Reddit / 社区贴合集

* **形式**：在 r/ClaudeAI 等子版块长期置顶/讨论的“公共服务器清单”“自荐贴”等。
* **优势**：能第一时间看到作者自荐或新服务器测试链接；但信息噪声也相对多，需要自行甄别。

---

### 选用建议一览

| 使用场景          | 推荐入口                                           |
| ------------- | ---------------------------------------------- |
| 想看**最多服务器**   | PulseMCP                                       |
| 想找**官方/权威参考** | Example Servers + modelcontextprotocol/servers |
| **命令行一键安装**   | mcp-get CLI                                    |
| 需要**自然语言搜索**  | MCP Compass                                    |
| 集成到后端 / 自动发现  | MCP Registry Service                           |
| 想跟进**社区新玩具**  | Awesome MCP Servers / Reddit 汇总                |

> **提示**：如果你在企业内网或私有云里使用 MCP，除了公开目录，还可以把自己的服务器元数据推送到 MCP Registry，或在 mcp-get 上发布私有包，以便团队内部自动发现。

希望这份排序清单能帮你快速锁定最合适的 MCP 服务器发现渠道！

[1]: https://github.com/michaellatman/mcp-get "GitHub - michaellatman/mcp-get"
