---
name: collect-repo
description: >
  Save GitHub repos to the user's Notion github知识库 database with full metadata.
  Use when the user provides a URL and asks to add it to their collection.
  Handles: visiting the page via in-app browser, generating rich tags (NO generic tags),
  in-app browser screenshot + upload, Notion page creation.

## Database Info
- Data Source ID: 23a6a5fa-4984-802a-a53f-000bb3d8182c
- Token: env:NOTION_API_TOKEN
- Image upload: https://likunqi.top/upload

## User Interest Profile
Based on 1269+ collected repos. The user is a novelty tool hunter:
- Loves AI/LLM tools, self-hostable (Docker/CF/Vercel) projects
- Collects MCP, Cloudflare, proxy/VPN, netdisk automation, media tools
- Banned water tags: 开源, Git, Github, Java, Python, 开发, 学习, 工具
- Each project needs at least 3 specific tags (5-8 is best)

## Workflow

### Step 1: Visit URL and Extract Metadata
Use the control-in-app-browser skill to navigate to the GitHub repo page.
Read the page title, description, and other visible metadata.
Write a Chinese description of the project (1 sentence).

### Step 2: Generate Tags (CRITICAL - NO banned tags)
- 5-8 specific, descriptive tags
- NEVER use: 开源, Git, Github, Java, Python, 开发, 学习, 工具
- GOOD examples: AI聊天机器人, Docker部署, Cloudflare Workers, SSL证书
- Generate from project description and tech stack keywords

### Step 3: Screenshot (Playwright primary, in-app browser fallback)

**Strategy**: Try Playwright first (max 2 attempts). On 2nd failure, fall back to control-in-app-browser.

**Primary: Playwright** (use bundled Node.js, playwright-core@1.61.0):
1. Launch headless Chromium (viewport: 1280x800)
2. Navigate with waitUntil: "load", timeout: 30s
3. Wait 2-3s for full render
4. Take screenshot (fullPage: false)
5. Close browser
6. If timeout/error, retry once. If still fails, go to fallback.

**Fallback: control-in-app-browser** (after 2 Playwright failures):
1. Use control-in-app-browser skill to navigate to the page
2. Call tab.playwright.screenshot to capture 1280x800
3. If sandboxed, transfer via asUint8Array()

### Step 4: Upload to likunqi.top
Use Node.js https with proper Buffer-based multipart:
- Build multipart with Buffer.concat (NOT toString binary - corrupts PNG)
- POST to https://likunqi.top/upload
- Parse response JSON to get image URL

### Step 5: Fetch Update Time
Extract relative-time element datetime attribute from the GitHub page via browser.

### Step 6: Create Notion Page (API v2025-09-03)
Use parent with type data_source_id, NOT database_id (deprecated).
Fields: Name, 网址, 来源, 标签, 详情, 图片, Date, 最后更新时间, 扫描时间, 更新状态

Status: <365d = 近一年更新, <730d = 近两年更新, <1095d = 近三年更新, >= 1095d = 暂停更新

### Step 7: Find Similar Projects
Search for related GitHub projects on the same topic and recommend 2-3 alternatives.

1. Use agent-reach (gh CLI) to search: `gh search repos "<keywords>" --sort stars --limit 5 --json fullName,stargazersCount,description,url`
2. Run 2-3 different queries targeting different angles (tech stack, use case, category).
3. Always exclude the collected repo itself from results.
4. Check each candidate against Notion: query data source with filter `{ "property": "网址", "url": { "equals": "https://github.com/..." } }`.
5. From remaining unfound candidates, pick 2-3 best matches sorted by stars (highest first).
6. Present recommendations clearly with star counts, descriptions, and GitHub URLs.

**Search strategy**: Tailor queries to project keywords/topics. Examples:
- `topic:mcp topic:cli stars:>2000`
- `topic:ai-agent topic:search stars:>1000`
- `"multi platform" scraper CLI stars:>1000`

### Step 8: Update AGENTS.md
Add entry to the update log section.

## Technical Notes
- Use Node.js require(https), NOT PowerShell (corrupts Chinese in Notion API)
- Formula field 更新状态（计算） auto-computes from Date
- Screenshots: primary = Playwright (up to 2 retries), fallback = control-in-app-browser
- Upload: must use Buffer.concat - no toString(binary) (corrupts PNG)
- ECONNRESET: retry up to 3 times with 1-2s interval
- Banned tags: 开源, Git, Github, Java, Python, 开发, 学习, 工具
- Step 7: use agent-reach (gh CLI) for GitHub search; Node.js https for Notion dedup check
