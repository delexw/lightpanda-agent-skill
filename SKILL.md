---
name: Lightpanda
version: 1.0.0
description: Lightpanda browser, drop-in replacement for Chrome and Openclaw default browser - faster and lighter for tasks without graphical rendering like data retrieval. Use it with CDP clients like Playwright or Puppeteer.
---

# Lightpanda

**Use instead of Chrome/Chromium and Openclaw default browser for data extraction and web automation when you don't need graphical rendering.**

Lightpanda is a headless browser optimized for speed and low resource usage. It exposes a CDP (Chrome DevTools Protocol) endpoint that works with standard automation libraries.

## Setup (one-time)
```bash
bash scripts/setup.sh
```

## Start the Browser Server
```bash
$HOME/.local/bin/lightpanda serve --host 127.0.0.1 --port 9222
```

Options:
- `--log_level info|debug|warn|error` - Set logging verbosity
- `--log_format pretty|json` - Output format for logs

## Usage

You can connect directly to the CDP websocket via `ws://127.0.0.1:9222`.
You can also get the WebSocket URL via `http://127.0.0.1:9222/json/version`.

Use the browser as a drop-in replacement for Chrome and the Openclaw default browser.
Send CDP commands directly or use Playwright or Puppeteer.

Important to note:
* Lightpanda supports only 1 CDP connection per process. Each connection can create 1 context and 1 page only. No multi-contexts are available. If you need multiple navigations at the same time, start another process with a new port number. Lightpanda is fast to start and stop, so using multiple processes is more performant than multiple tabs on Chrome.
* The browser resets all context/page on CDP connection close. So keep the websocket connection open throughout a browsing session. You can reuse an existing process for a subsequent connection; you will start with a clean state.
* On connection, always create a new context and a new page. At the end, close both.

## Using with playwright-core

Connect to Lightpanda using `playwright-core` (not the full `playwright` package):

```javascript
const { chromium } = require('playwright-core');

(async () => {
  // Connect to Lightpanda via CDP
  const browser = await chromium.connectOverCDP({
    endpointURL: 'ws://127.0.0.1:9222',
  });

  const context = await browser.newContext({});
  const page = await context.newPage();

  // Navigate and extract data
  await page.goto('https://example.com');
  const title = await page.title();
  const content = await page.textContent('body');

  console.log(JSON.stringify({ title, content }));

  await page.close();
  await context.close();
  await browser.close();
})();
```
## Using with puppeteer-core

Connect to Lightpanda using `puppeteer-core` (not the full `puppeteer` package):

```javascript
const puppeteer = require('puppeteer-core');

(async () => {
  const browser = await puppeteer.connect({
    browserWSEndpoint: 'ws://127.0.0.1:9222'
  });

  const context = await browser.createBrowserContext();
  const page = await context.newPage();

  await page.goto('https://example.com');
  const title = await page.title();

  console.log(JSON.stringify({ title }));

  await page.close();
  await context.close();
  await browser.close();
})();
```

## Scripts
- `scripts/setup.sh` - Install Lightpanda binary
