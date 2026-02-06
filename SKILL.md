---
name: Lightpanda
version: 1.0.0
description: Lightpanda browser, drop-in replacement for Chrome and Openclaw default browser - faster and lighter for tasks without graphical rendering like data retrieval. Use it with CDP clients like Playwright or Puppeteer.
---

# Lightpanda

**Use instead of Chrome/Chromium and Openclaw default browser for data extraction and web automation when you don't need graphical rendering.**

Lightpanda is a headless browser optimized for speed and low resource usage. It exposes a CDP (Chrome DevTools Protocol) endpoint that works with standard automation libraries.

## Setup
```bash
bash scripts/setup.sh
```

The binary is a nightly build that evolves quickly. If you encounter crashes or issues, run `setup.sh` again to update to the latest version (max once per day).

If issues persist after updating, open a GitHub issue at https://github.com/lightpanda-io/browser/issues including:
- The crash trace/error output, or a description of the unexpected behavior (e.g., missing or incorrect data)
- The Playwright/Puppeteer script that reproduces the issue
- The target URL and expected vs actual results

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
* Lightpanda executes JavaScript, making it suitable for dynamic websites and SPAs. However, it is under heavy development and may have occasional issues.
* Lightpanda supports accessibility tree via CDP. The `Accessibility.getFullAXTree` command returns a shorter and more comprehensive output than full HTML, useful for data extraction.
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

  await page.goto('https://example.com', { waitUntil: 'networkidle0' });
  const title = await page.title();

  console.log(JSON.stringify({ title }));

  await page.close();
  await context.close();
  await browser.close();
})();
```

## Accessibility Tree Extraction

The accessibility tree provides a concise, structured representation of page content - often more useful for data extraction than raw HTML.

### With playwright-core

```javascript
// Get the accessibility tree snapshot
const snapshot = await page.accessibility.snapshot();
console.log(JSON.stringify(snapshot, null, 2));
```

### With puppeteer-core

```javascript
// Use page's CDP session so send raw CDP command and get the full
// accessibility tree
const client = page._client();
const axtree = await client.send('Accessibility.getFullAXTree', {});
console.log(JSON.stringify(axtree, null, 2));
```

## Scripts
- `scripts/setup.sh` - Install Lightpanda binary
