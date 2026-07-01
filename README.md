# Collect Repo Plugin 🗂️

**Codex 插件** — 将 GitHub 项目收录到 Notion 知识库，全自动截图、打标签、抓取更新时间。

## 功能

- **元数据提取** — 访问 GitHub 页面，提取项目名、描述、Star 数、技术栈
- **智能标签** — 根据项目描述和技术栈自动生成 5-8 个具体描述性标签（禁止水标签）
- **自动截图** — Playwright 无头浏览器截图（1280×800），失败时回退到本地浏览器
- **图床上传** — 截图自动上传到 [likunqi.top](https://likunqi.top)
- **更新时间** — 从 GitHub 页面提取最后推送时间，自动判定更新状态（近一年/两年/三年/暂停更新）
- **Notion 入库** — 使用 Notion API v2025-09-03 创建完整记录
- **同类推荐** — 通过 `gh CLI` 搜索类似项目，自动去重检查是否已收录
- **AGENTS.md 更新** — 自动追加收录日志到项目 AGENTS.md

## 安装

### 前提条件

1. **Codex CLI** — 需要安装 Codex 桌面版
2. **Node.js** — 使用 Codex 内置运行时（含 Playwright）
3. **Notion API Token** — 环境变量 `NOTION_API_TOKEN`
4. **Notion Data Source ID** — 环境变量 `NOTION_DATA_SOURCE_ID`（你的 Notion 数据库 UUID）
4. **gh CLI** — 已认证的 GitHub CLI（用于 Step 7 搜索类似项目）

### 安装插件

```json
// 将以下内容添加到 ~/.agents/plugins/marketplace.json
{
  "name": "personal",
  "interface": { "displayName": "Personal" },
  "plugins": [
    {
      "name": "collect-repo",
      "source": {
        "source": "github",
        "owner": "likunqi",
        "repo": "collect-repo-plugin",
        "path": "."
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Productivity"
    }
  ]
}
```

安装后重启 Codex 即可使用。

## 使用方式

给 Codex 发送一条消息即可，例如：

> 收录这个项目 https://github.com/xxx/yyy
> 收藏这个仓库
> 帮我保存 [GitHub 链接] 到数据库

插件会自动完成全部 8 个步骤。

## 数据库结构

Notion 数据库名：**github知识库**

| 字段 | 类型 | 说明 |
|------|------|------|
| Name | title | 项目名称 |
| 网址 | url | GitHub 链接 |
| 来源 | rich_text | 来源标记（如"自定义收录 - Codex推荐"） |
| 标签 | multi_select | 5-8 个描述性标签 |
| 详情 | rich_text | 项目中文描述 |
| 图片 | url | 首页截图（上传到 likunqi.top） |
| Date | date | GitHub 最后推送时间 |
| 最后更新时间 | date | 同 Date |
| 扫描时间 | date | 数据扫描时间 |
| 更新状态 | status | 近一年更新 / 近两年更新 / 近三年更新 / 暂停更新 |
| 更新状态（计算） | formula | 自动计算（引用的 Date 字段） |
| 喜欢 | checkbox | 收藏标记 |

## 技术栈

- **Notion API** — v2025-09-03
- **Playwright** — 浏览器截图（playwright-core@1.61.0）
- **Node.js** — 数据处理、Notion API 调用
- **PowerShell / gh CLI** — 搜索同类项目

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| 0.2.0 | 2026-06-30 | 截图改为 Playwright 优先 + 本地浏览器 fallback，标签规则化，Notion API v2 适配 |
| 0.1.0 | 2026-06-30 | 初始版本 |

## 许可证

MIT
