# lightpanda

## Overview

Lightpanda integration skill for Claude Code and OpenClaw. Lightpanda is an AI-native headless browser built with Zig, offering 11x faster performance and 9x less memory usage compared to Chrome.

## When to Use

Use this skill when:
- You need **high-performance browser automation** for AI agents
- Working with **Claude MCP** or **OpenClaw** browser automation
- Running **large-scale web scraping** or data extraction
- Building **LLM-powered web interaction** tools
- You want a **drop-in Chrome replacement** for better performance

## Key Features

- **11x faster** than Chrome
- **9x less memory** usage
- **CDP/Playwright/Puppeteer compatible**
- **Docker-ready** for quick deployment
- **AI-native** design (not adapted from human browser)

## Installation

### Quick Start with Docker

```bash
# Pull and run Lightpanda
docker run -d -p 9222:9222 --name lightpanda lightpanda/lightpanda:latest

# Verify connection
curl http://localhost:9222/json/version
```

### From Source

```bash
git clone https://github.com/lightpanda-io/lightpanda.git
cd lightpanda
zig build
./zig-out/bin/lightpanda
```

## Integration Patterns

### Claude Code MCP Integration

Add to `.claude/config.json`:

```json
{
  "mcpServers": {
    "lightpanda": {
      "command": "node",
      "args": ["path/to/agent-skill/dist/index.js"],
      "env": {
        "LIGHTPANDA_URL": "http://localhost:9222"
      }
    }
  }
}
```

### Playwright Integration

```javascript
const { chromium } = require('playwright');

const browser = await chromium.connect('ws://localhost:9222');
const page = await browser.newPage();
await page.goto('https://example.com');
```

### Python/OpenClaw Integration

```python
from playwright.async_api import async_playwright

async with async_playwright() as p:
    browser = await p.chromium.connect('ws://localhost:9222')
    page = await browser.new_page()
    await page.goto('https://example.com')
```

## Common Commands

| Task | Command |
|------|---------|
| Start Lightpanda | `docker run -d -p 9222:9222 lightpanda/lightpanda:latest` |
| Check status | `curl http://localhost:9222/json/version` |
| Stop container | `docker stop lightpanda` |
| View logs | `docker logs lightpanda` |

## Performance Comparison

| Metric | Chrome | Lightpanda |
|--------|--------|------------|
| Speed | 1x | **11x** |
| Memory | 1x | **-9x** |
| Startup | Slow | Fast |
| Design | Human-first | **AI-first** |

## Architecture

```
┌─────────────────┐
│  Your Agent     │
│  (Claude/OpenClaw)│
└────────┬────────┘
         │ CDP/Playwright
         ▼
┌─────────────────┐
│  Lightpanda     │
│  (Zig-based)    │
└─────────────────┘
```

## Troubleshooting

**Connection refused**: Ensure Lightpanda is running on port 9222
```bash
docker ps | grep lightpanda
```

**WebSocket errors**: Check firewall settings
```bash
curl -i -N -H "Connection: Upgrade" -H "Upgrade: websocket" http://localhost:9222/
```

**Memory issues**: Use Docker resource limits
```bash
docker run -d -p 9222:9222 --memory="512m" lightpanda/lightpanda:latest
```

## Resources

- Official GitHub: https://github.com/lightpanda-io/lightpanda
- Agent Skill: https://github.com/lightpanda-io/agent-skill
- CDP Protocol: https://chromedevtools.github.io/devtools-protocol/
- Documentation: https://docs.lightpanda.io

## Notes

- Lightpanda is a **drop-in replacement** for Chrome in CDP-based tools
- Perfect for **containerized environments** due to small footprint
- Supports all major automation frameworks (Playwright, Puppeteer)
- Actively maintained with **regular performance updates**
