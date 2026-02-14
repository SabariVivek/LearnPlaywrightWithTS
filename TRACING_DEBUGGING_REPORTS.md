# Tracing, Debugging & HTML Reports

## Overview

Tracing and reporting are critical for understanding test failures, debugging issues, and providing visibility in CI/CD pipelines. Playwright provides powerful tools to capture test execution details, generate interactive reports, and record video evidence of test runs.

This guide covers:
- **Tracing**: Capture DOM snapshots, network logs, screenshots at each step
- **HTML Reports**: Generate beautiful, interactive test reports
- **Video Recording**: Record browser interactions for debugging
- **Console/Network Logs**: Capture JavaScript errors and API calls
- **CI/CD Integration**: Configure reports for pipeline visibility

---

## 1. Tracing Basics

### What is Tracing?

Tracing captures a detailed timeline of test execution including:
- DOM snapshots (HTML state at each step)
- Network requests and responses
- Screenshots and video frames
- Console messages and errors
- Timing information

### Benefits

✅ **Debugging**: Understand exactly what happened when test failed
✅ **Documentation**: Visual proof of test execution
✅ **Performance Analysis**: Identify slow operations
✅ **CI/CD Visibility**: See failures without re-running locally
✅ **Compliance**: Audit trail for regulated environments

### Understanding Trace Files

Trace files are `.zip` archives containing:
- `resources/`: Network responses and DOM snapshots
- `trace.network`: Network activity log
- `trace.stacks`: Call stacks
- `trace.events`: Timestamped events

---

## 2. Basic Tracing Implementation

### Enable Tracing in Config

```typescript
// playwright.config.ts
export default defineConfig({
  use: {
    trace: 'on-first-retry',  // Capture trace on first retry/failure
  },
});
```

Options:
- `'off'`: Don't record trace
- `'on'`: Always record trace (uses storage)
- `'retain-on-failure'`: Only record on failure
- `'on-first-retry'`: Record when test retries

### Manual Trace Control

```typescript
import { test } from '@playwright/test';

test('example with manual tracing', async ({ page }) => {
  // Start tracing
  await page.context().tracing.start({
    screenshots: true,
    snapshots: true,
    sources: true,
  });

  // Your test steps
  await page.goto('https://testautomationpractice.blogspot.com/');
  await page.locator('input[id="name"]').fill('John Doe');
  await page.locator('button:has-text("Submit")').click();

  // Stop and save trace
  await page.context().tracing.stop({
    path: 'trace-output.zip',
  });
});
```

### Trace Options Explained

```typescript
await page.context().tracing.start({
  screenshots: true,    // Capture screenshots at each action
  snapshots: true,      // Capture DOM snapshots (HTML state)
  sources: true,        // Include source files for debugging
});
```

---

## 3. Viewing Traces

### Using Playwright Inspector

```bash
# Open trace file in Playwright Inspector
npx playwright show-trace trace-output.zip
```

Inspector features:
- Timeline of actions
- Screenshots at each step
- DOM snapshot viewer
- Network activity
- Network response bodies

### Trace Viewer Web Tool

```bash
# Online trace viewer (no installation needed)
# Visit: https://trace.playwright.dev/
# Upload your trace-output.zip file
```

---

## 4. HTML Reports

### Built-in HTML Reporter

```typescript
// playwright.config.ts
export default defineConfig({
  reporter: [
    ['html'],  // Generate HTML report
    ['list'],  // Console output
  ],
});
```

Reports location: `playwright-report/`

### View HTML Report

```bash
# After test run
npx playwright show-report

# Or open directly
# Open playwright-report/index.html in browser
```

### HTML Report Features

✅ **Summary**: Total tests, passed, failed, skipped
✅ **Test Details**: Each test with status, duration, errors
✅ **Screenshots**: Automatic screenshots on failure
✅ **Video**: Playback of test execution
✅ **Traces**: Link to full trace for debugging
✅ **Filtering**: Filter by status, project, duration
✅ **Sorting**: Sort by any column

---

## 5. Advanced Trace Scenarios

### Conditional Tracing Based on Failure

```typescript
import { test } from '@playwright/test';

test('with conditional tracing', async ({ page }, testInfo) => {
  // Start tracing
  await page.context().tracing.start({
    screenshots: true,
    snapshots: true,
  });

  try {
    // Your test
    await page.goto('https://testautomationpractice.blogspot.com/');
    await page.locator('#nonexistent').click();
  } finally {
    // Only save trace if test failed
    if (testInfo.status !== 'passed') {
      await page.context().tracing.stop({
        path: `trace-${testInfo.testId}.zip`,
      });
    } else {
      await page.context().tracing.stop();
    }
  }
});
```

### Multiple Traces in Single Test

```typescript
import { test } from '@playwright/test';

test('multiple trace segments', async ({ page }) => {
  // Trace Segment 1: Authentication
  await page.context().tracing.start({ screenshots: true });
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  await page.locator('a:has-text("Login")').click();
  
  await page.context().tracing.stop({
    path: 'trace-auth.zip',
  });

  // Segment 2: Main workflow
  await page.context().tracing.start({ screenshots: true });
  
  await page.locator('input[id="name"]').fill('Test User');
  await page.locator('button:has-text("Submit")').click();
  
  await page.context().tracing.stop({
    path: 'trace-workflow.zip',
  });
});
```

---

## 6. Video Recording

### Enable Video in Config

```typescript
// playwright.config.ts
export default defineConfig({
  use: {
    video: 'retain-on-failure',  // Record only on failure
  },
});
```

Options:
- `'off'`: No video recording
- `'on'`: Always record video
- `'retain-on-failure'`: Only keep video if test fails
- `'on-first-retry'`: Record on retry

### Videos Location

Test videos saved to: `test-results/`

```
test-results/
├── test-name-chromium/
│   └── video.webm
├── test-name-firefox/
│   └── video.webm
```

### Specify Video Directory

```typescript
// playwright.config.ts
export default defineConfig({
  webServer: undefined,
  use: {
    video: 'retain-on-failure',
    videoDir: 'videos/',  // Custom directory
  },
});
```

### Programmatic Video Control

```typescript
import { test } from '@playwright/test';

test('manual video control', async ({ page }) => {
  // Start video recording
  const videoPath = page.video()?.path();
  
  // Your test
  await page.goto('https://testautomationpractice.blogspot.com/');
  await page.locator('input[id="email"]').fill('test@example.com');
  
  // Video saves automatically at end
  // Use path for custom processing
  console.log(`Video saved to: ${videoPath}`);
});
```

---

## 7. Console & Network Logs

### Capture Console Messages

```typescript
import { test } from '@playwright/test';

test('capture console logs', async ({ page }) => {
  const logs: string[] = [];
  const errors: string[] = [];

  // Listener for all console messages
  page.on('console', (message) => {
    console.log(`${message.type()}: ${message.text()}`);
    logs.push(message.text());
  });

  // Listener for page crashes
  page.on('crash', () => {
    console.log('🔴 Page crashed!');
  });

  // Listener for uncaught exceptions
  page.on('pageerror', (error) => {
    console.log(`❌ Uncaught error: ${error.message}`);
    errors.push(error.message);
  });

  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Trigger JavaScript error deliberately
  await page.evaluate(() => {
    console.log('Test log from page');
    console.warn('Test warning');
  });

  console.log(`Captured ${logs.length} console messages`);
  console.log(`Captured ${errors.length} errors`);
});
```

### Capture Network Requests

```typescript
import { test } from '@playwright/test';

test('capture network activity', async ({ page }) => {
  const requests: string[] = [];
  const responses: Map<string, number> = new Map();

  // Listener for all requests
  page.on('request', (request) => {
    const url = request.url();
    console.log(`→ ${request.method()} ${url}`);
    requests.push(url);
  });

  // Listener for responses
  page.on('response', (response) => {
    const url = response.url();
    const status = response.status();
    console.log(`← ${status} ${url}`);
    responses.set(url, status);
  });

  // Listener for failed requests
  page.on('requestfailed', (request) => {
    console.log(`❌ FAILED: ${request.url()} - ${request.failure()?.errorText}`);
  });

  await page.goto('https://testautomationpractice.blogspot.com/');

  console.log(`Total requests: ${requests.length}`);
  
  // Check for any 4xx or 5xx errors
  const errorResponses = Array.from(responses.entries())
    .filter(([_, status]) => status >= 400);
  
  console.log(`Error responses: ${errorResponses.length}`);
});
```

### Capture Specific Network Calls

```typescript
import { test, expect } from '@playwright/test';

test('capture and validate specific API call', async ({ page }) => {
  let apiResponse: any = null;

  // Listen for specific API endpoint
  page.on('response', async (response) => {
    if (response.url().includes('/api/users')) {
      apiResponse = await response.json();
      console.log('API Response:', apiResponse);
    }
  });

  await page.goto('https://testautomationpractice.blogspot.com/');
  await page.locator('button:has-text("Load Users")').click();

  // Wait for API call
  await page.waitForTimeout(1000);

  // Verify API response
  expect(apiResponse).not.toBeNull();
  expect(apiResponse.users).toBeDefined();
});
```

---

## 8. CI/CD Integration

### GitHub Actions Configuration

```yaml
# .github/workflows/playwright.yml
name: Playwright Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: 18
      
      - name: Install dependencies
        run: npm ci
        
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Run tests
        run: npm run test
        
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

### GitLab CI Configuration

```yaml
# .gitlab-ci.yml
test:playwright:
  image: mcr.microsoft.com/playwright:v1.40.0-jammy
  
  script:
    - npm ci
    - npm run test
  
  artifacts:
    when: always
    paths:
      - playwright-report/
      - test-results/
    expire_in: 30 days
  
  allow_failure: false
```

### Jenkins Configuration

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
                sh 'npx playwright install --with-deps'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm run test'
            }
        }
    }
    
    post {
        always {
            publishHTML([
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright Report'
            ])
            
            archiveArtifacts(
                artifacts: 'test-results/**',
                allowEmptyArchive: true
            )
        }
    }
}
```

---

## 9. Complete Example: Full Trace & Report Setup

```typescript
import { test, expect } from '@playwright/test';

test.describe('E-commerce checkout with full tracing', () => {
  test('complete purchase with trace and video', async ({ page }, testInfo) => {
    // Start detailed tracing
    await page.context().tracing.start({
      screenshots: true,
      snapshots: true,
      sources: true,
    });

    // Setup listeners
    const consoleLogs: string[] = [];
    const networkErrors: string[] = [];

    page.on('console', (msg) => {
      consoleLogs.push(msg.text());
    });

    page.on('response', (response) => {
      if (response.status() >= 400) {
        networkErrors.push(`${response.status()} ${response.url()}`);
      }
    });

    try {
      // Test steps
      console.log('📍 Step 1: Navigate to store');
      await page.goto('https://testautomationpractice.blogspot.com/');

      console.log('📍 Step 2: Search for product');
      await page.locator('input[id="search"]').fill('Laptop');
      await page.locator('button:has-text("Search")').click();

      console.log('📍 Step 3: Add product to cart');
      await page.locator('button:has-text("Add to Cart")').first().click();

      console.log('📍 Step 4: Navigate to cart');
      await page.locator('a:has-text("View Cart")').click();

      console.log('📍 Step 5: Proceed to checkout');
      await page.locator('button:has-text("Checkout")').click();

      console.log('📍 Step 6: Enter shipping details');
      await page.locator('input[name="address"]').fill('123 Main St');
      await page.locator('input[name="city"]').fill('New York');
      await page.locator('select[name="country"]').selectOption('US');

      console.log('📍 Step 7: Complete purchase');
      await page.locator('button:has-text("Place Order")').click();

      // Verify success
      await expect(page.locator('text=Order Confirmed')).toBeVisible();

      // Log results
      console.log(`✅ Test passed`);
      console.log(`📊 Console logs: ${consoleLogs.length}`);
      console.log(`🌐 Network errors: ${networkErrors.length}`);

    } catch (error) {
      console.error(`❌ Test failed: ${error}`);
      
      // Take screenshot on failure
      await page.screenshot({
        path: `failure-${testInfo.testId}.png`,
      });
      
      throw error;
    } finally {
      // Stop tracing
      await page.context().tracing.stop({
        path: `trace-${testInfo.testId}.zip`,
      });
    }
  });
});
```

---

## 10. Advanced Reports & Dashboards

### Custom Report Generator

```typescript
import { test } from '@playwright/test';
import * as fs from 'fs';
import * as path from 'path';

test.afterAll(async ({ }, testInfo) => {
  // Generate custom summary
  const results = testInfo.config;
  const reportData = {
    timestamp: new Date().toISOString(),
    totalTests: results.testMatch?.length || 0,
    duration: Date.now(),
    environment: process.env.NODE_ENV,
    browser: process.env.BROWSER || 'chromium',
  };

  const reportPath = path.join('custom-reports', 'summary.json');
  fs.writeFileSync(reportPath, JSON.stringify(reportData, null, 2));
});
```

### Multiple Reporter Configuration

```typescript
// playwright.config.ts
export default defineConfig({
  reporter: [
    ['html'],                    // Interactive HTML
    ['json', { outputFile: 'test-results.json' }],  // JSON data
    ['junit', { outputFile: 'junit.xml' }],         // CI/CD format
    ['list'],                    // Console list
    ['github'],                  // GitHub Actions
  ],
});
```

### Report Output Formats

```bash
# HTML Report (default)
playwright-report/index.html

# JSON Report
test-results.json

# JUnit XML (for Jenkins, etc.)
junit.xml

# GitHub Actions Format
(automatically integrated)
```

---

## 11. Best Practices

### ✅ DO

```typescript
// DO: Enable tracing in config
export default defineConfig({
  use: {
    trace: 'retain-on-failure',
  },
});

// DO: Capture console and network errors
page.on('console', (msg) => console.log(msg));
page.on('response', (res) => {
  if (res.status() >= 400) console.log(`Error: ${res.status()}`);
});

// DO: Use descriptive test names
test('should complete checkout with valid payment', async () => {
  // Clear what test does
});

// DO: Commit trace viewer for team analysis
// DO: Archive reports in CI/CD for historical tracking
// DO: Use video for complex UI interactions
// DO: Enable sources in trace for full debugging
```

### ❌ DON'T

```typescript
// DON'T: Always record video (storage overhead)
use: {
  video: 'on',  // Too much storage
}

// DON'T: Ignore network errors
// Missing error handling for API failures

// DON'T: Use screenshots for single elements
// Use traces for better debugging

// DON'T: Disable tracing in production tests
// You'll regret it when debugging failures

// DON'T: Keep all traces without cleanup
// Implement retention policy for storage
```

---

## 12. Key Takeaways

🎯 **Core Concepts:**
- Tracing captures DOM, network, screenshots in real-time
- HTML reports provide interactive test results
- Video recording helps debug complex interactions
- Console/network logs capture JavaScript errors

🛠️ **Implementation:**
- Enable in config: `trace: 'retain-on-failure'`
- Manual control: `tracing.start()` / `tracing.stop()`
- View traces: `npx playwright show-trace`
- View reports: `npx playwright show-report`

📊 **CI/CD Integration:**
- Upload reports as build artifacts
- Link to GitHub Actions, GitLab, Jenkins
- Automatic archival for historical analysis

🔍 **Debugging Strategy:**
1. Run test with tracing enabled
2. Check HTML report for overview
3. Open trace for detailed timeline
4. Review console/network logs for errors
5. Watch video for UI interaction issues

---

## Quick Reference Commands

```bash
# Run tests with tracing
npm run test

# View HTML report
npx playwright show-report

# View specific trace file
npx playwright show-trace trace-output.zip

# List all test results
npm run test -- --reporter=list

# Generate both HTML and JSON reports
npm run test -- --reporter=html --reporter=json

# View test results for specific file
npm run test tests/checkout.spec.ts

# Debug with inspector
npx playwright test --debug
```

---

**Next Steps:**
- Enable tracing in your `playwright.config.ts`
- Run tests and review HTML reports
- Set up CI/CD artifact uploads
- Configure custom reporters for your needs
- Archive traces for long-term reference
