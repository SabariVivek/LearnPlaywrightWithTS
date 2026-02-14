# Playwright Error Handling & Retry Logic

## What are Errors & Retry Logic?

**Error** = Something goes wrong during test execution (element not found, timeout, navigation fails).

**Retry Logic** = Automatically retry the failed action/test to handle flaky situations.

Think of it like: **Calling a friend - if they don't answer (error), you call again (retry)**

```
First Try: Click button → Failed
Retry 1: Click button → Failed
Retry 2: Click button → Success ✓

Without retry = One failure = Test fails
With retry = Multiple attempts = Test passes if any succeeds
```

---

## Why Use Error Handling & Retry Logic?

In real-world testing:
- ✅ **Network issues** - Temporary connection problems
- ✅ **Slow elements** - Page sometimes loads slower
- ✅ **Flaky tests** - Test passes 90% of the time
- ✅ **Race conditions** - Timing-dependent failures
- ✅ **Browser inconsistency** - Same test behaves differently
- ✅ **Third-party services** - External service delays

**Without retry:**
```
Run 100 tests → 95 pass, 5 fail (flaky)
Rerun → 98 pass, 2 fail (different failures)
Conclusion: Broken tests! (Actually just flaky)
```

**With retry:**
```
Run 100 tests with retry → All pass (retries handled flakiness)
Conclusion: Stable tests! ✓
```

---

## Types of Errors

| Error Type | Example | Solution |
|-----------|---------|----------|
| **Timeout** | Element not found in 30s | Retry, increase timeout |
| **Network** | Page load fails | Retry, check connection |
| **Assertion** | Expected ≠ Actual | Usually don't retry |
| **Navigation** | URL change fails | Retry, check server |
| **Element** | Selector doesn't exist | Retry, verify selector |

---

---

# PLAYWRIGHT BUILT-IN RETRY MECHANISMS

## What Does Playwright Retry Automatically?

Playwright has **automatic retry** for:
- ✅ **Auto-waiting** - Waits for element before action (30s default)
- ✅ **Auto-retrying assertions** - Retries until true or timeout
- ✅ **Test retry** - Retry entire test if it fails

---

## Example 1: Auto-Wait (Built-in Retry)

### What Happens

You try to click → Element not visible → Playwright waits and retries → Element appears → Click succeeds

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Auto-wait example', async ({ page }) => {
  console.log('=== Auto-Wait (Built-in Retry) ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  console.log('Attempting to click button...');
  
  // Element might not be visible immediately
  // Playwright automatically waits and retries (up to 30 seconds)
  
  try {
    await page.click('button:has-text("Submit")');
    console.log('✓ Button clicked after auto-wait');
  } catch (error) {
    console.log('❌ Button not found even after retry');
  }
});
```

**How auto-wait works:**
1. Try to click → Element not visible
2. Wait 100ms → Try again
3. Wait 100ms → Try again
4. ... (repeat) ...
5. After 30 seconds → Give up, throw error

**Custom timeout:**
```javascript
// Change timeout globally in config
export default {
  use: {
    actionTimeout: 60000,  // 60 seconds instead of 30
  },
};

// Or per action
await page.click('button', { timeout: 10000 });  // 10 seconds
```

---

## Example 2: Auto-Retrying Assertions

### What Happens

Check assertion → Not true yet → Retry → Eventually true → Pass

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Auto-retrying assertion', async ({ page }) => {
  console.log('=== Auto-Retrying Assertions ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Simulate element appearing after delay
  await page.evaluate(() => {
    setTimeout(() => {
      const newElement = document.createElement('div');
      newElement.className = 'success-message';
      newElement.textContent = 'Success!';
      document.body.appendChild(newElement);
    }, 2000);  // Appears after 2 seconds
  });
  
  console.log('Checking for success message...');
  
  // This assertion automatically retries for 30 seconds
  // Playwright checks every 100ms
  // If it finds it within 30s → PASS
  // If not found after 30s → FAIL
  
  await expect(page.locator('.success-message')).toBeVisible();
  
  console.log('✓ Assertion passed (element appeared)\n');
});
```

**Key difference:**
```javascript
// Non-retrying assertion
expect(5).toBe(5);  // Check once, immediately

// Auto-retrying assertion
await expect(page.locator('text=Success')).toBeVisible();  // Check repeatedly for 30s
```

---

## Example 3: Test-Level Retry

### What Happens

Test fails → Run entire test again → If passes on second run, test passes

**Playwright Configuration:**
```javascript
// playwright.config.ts
export default {
  retries: 2,  // Retry failed tests up to 2 times
};
```

**How it works in CI:**
```
Test 1: FAIL → Retry
Retry 1: PASS → Mark as passed
Result: Test passed (after 1 retry)

---

Test 2: FAIL → Retry
Retry 1: FAIL → Retry
Retry 2: PASS → Mark as passed
Result: Test passed (after 2 retries)

---

Test 3: FAIL → Retry
Retry 1: FAIL → Retry
Retry 2: FAIL → Mark as failed
Result: Test failed (exceeded retries)
```

**Code Example:**
```javascript
import { test, expect } from '@playwright/test';

test('Flaky test with retry', async ({ page }) => {
  console.log('=== Flaky Test Example ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Simulate flaky condition (50% chance of failure)
  const random = Math.random();
  
  console.log(`Random value: ${random.toFixed(2)}`);
  
  if (random < 0.5) {
    throw new Error('Random failure (expected in flaky test)');
  }
  
  console.log('✓ Test passed\n');
  
  // With retry: 2 in playwright.config.ts
  // First run fails → Retry 1 → 50% chance
  // If Retry 1 fails → Retry 2 → 50% chance
  // Probability all 3 fail = 0.5^3 = 12.5% (much better)
});
```

**Configure per test:**
```javascript
test('Important test', async ({ page }) => {
  // ...
});

test.describe('Flaky suite', () => {
  test.describe.configure({ retries: 3 });  // Retry 3 times for this suite
  
  test('Test 1', async ({ page }) => { });
  test('Test 2', async ({ page }) => { });
});
```

---

---

# CUSTOM ERROR HANDLING

## Example 1: Try-Catch Error Handling

### What Happens

Attempt action → If error → Catch and handle → Continue or retry

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Try-catch error handling', async ({ page }) => {
  console.log('=== Try-Catch Error Handling ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Try to find and click element
  try {
    console.log('Attempting to click element...');
    await page.click('button#submit', { timeout: 5000 });
    console.log('✓ Click succeeded');
    
  } catch (error) {
    console.log('❌ Click failed with error:');
    console.log('  Error type:', error.name);
    console.log('  Error message:', error.message);
    
    // Handle specific error
    if (error.message.includes('Timeout')) {
      console.log('  → Element not found, trying alternative...');
      
      // Try alternative selector
      try {
        await page.click('button:has-text("Submit")');
        console.log('  ✓ Alternative selector worked');
      } catch (e) {
        console.log('  ❌ Alternative also failed');
      }
      
    } else if (error.message.includes('not found')) {
      console.log('  → Selector invalid, skipping');
      
    } else {
      console.log('  → Unexpected error, re-throwing');
      throw error;
    }
  }
});
```

---

## Example 2: Custom Retry Function

### What Happens

Create reusable function that retries action multiple times with delays

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

// Custom retry function
async function retryWithBackoff(
  action,
  maxRetries = 3,
  delayMs = 1000
) {
  console.log(`\nRetrying ${action.name} (max ${maxRetries} retries)`);
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      console.log(`  Attempt ${attempt}...`);
      const result = await action();
      console.log(`  ✓ Success on attempt ${attempt}\n`);
      return result;
      
    } catch (error) {
      console.log(`  ❌ Failed: ${error.message}`);
      
      if (attempt === maxRetries) {
        console.log(`  Exhausted all retries\n`);
        throw error;
      }
      
      // Exponential backoff: 1s, 2s, 4s, 8s...
      const delayWithBackoff = delayMs * Math.pow(2, attempt - 1);
      console.log(`  Waiting ${delayWithBackoff}ms before retry...`);
      
      await new Promise(resolve => setTimeout(resolve, delayWithBackoff));
    }
  }
}

test('Custom retry function', async ({ page }) => {
  console.log('=== Custom Retry Function ===');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Use retry function
  await retryWithBackoff(
    async () => {
      // This can fail randomly
      const random = Math.random();
      if (random < 0.7) {  // 70% failure rate
        throw new Error('Random network error');
      }
      
      await page.click('button#submit');
      console.log('  Button clicked');
    },
    3,      // Max 3 retries
    1000    // Start with 1 second delay
  );
  
  console.log('✓ Action succeeded after retries\n');
});
```

---

## Example 3: Retry Specific Assertion

### What Happens

Check condition repeatedly until true or max attempts

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

// Retry assertion function
async function retryAssertion(
  condition,
  message = 'Assertion',
  maxRetries = 5,
  delayMs = 1000
) {
  let lastError;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const result = await condition();
      
      if (result) {
        console.log(`✓ ${message} passed on attempt ${attempt}`);
        return;
      }
      
    } catch (error) {
      lastError = error;
    }
    
    if (attempt < maxRetries) {
      await new Promise(resolve => setTimeout(resolve, delayMs));
    }
  }
  
  throw new Error(`${message} failed after ${maxRetries} attempts`);
}

test('Retry specific assertion', async ({ page }) => {
  console.log('=== Retry Assertion ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Wait for counter to reach specific value
  await retryAssertion(
    async () => {
      const count = await page.locator('.counter').textContent();
      return parseInt(count) >= 10;
    },
    'Counter >= 10',
    10,      // Max 10 attempts
    500      // 500ms between retries
  );
  
  console.log('✓ Counter reached 10\n');
});
```

---

---

# RETRY STRATEGIES

## Strategy 1: Fixed Delay Retry

### What Happens

Fail → Wait 1 second → Retry → Fail → Wait 1 second → Retry → Success

```javascript
async function retryWithFixedDelay(action, maxRetries = 3, delayMs = 1000) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await action();
    } catch (error) {
      if (attempt === maxRetries) throw error;
      await new Promise(r => setTimeout(r, delayMs));  // Fixed 1 second
    }
  }
}

// Usage
await retryWithFixedDelay(
  async () => await page.click('button'),
  3,      // 3 retries
  1000    // 1 second each
);
```

**Best for:** Temporary network hiccups

---

## Strategy 2: Exponential Backoff

### What Happens

Fail → Wait 1s → Retry → Fail → Wait 2s → Retry → Fail → Wait 4s → Retry

```javascript
async function retryWithExponentialBackoff(action, maxRetries = 3, baseDelayMs = 1000) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await action();
    } catch (error) {
      if (attempt === maxRetries) throw error;
      
      // Exponential: 1s, 2s, 4s, 8s, 16s...
      const delay = baseDelayMs * Math.pow(2, attempt - 1);
      console.log(`Waiting ${delay}ms before retry ${attempt}...`);
      await new Promise(r => setTimeout(r, delay));
    }
  }
}

// Usage
await retryWithExponentialBackoff(async () => await page.goto(url), 3, 1000);
```

**Best for:** Server overload, rate limiting

---

## Strategy 3: Immediate Retry

### What Happens

Fail → Immediately retry (no delay) → Fail → Retry

```javascript
async function retryImmediate(action, maxRetries = 3) {
  let lastError;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await action();
    } catch (error) {
      lastError = error;
      // No delay, immediate retry
    }
  }
  
  throw lastError;
}

// Usage
await retryImmediate(async () => await page.click(selector), 5);
```

**Best for:** Race conditions, timing issues

---

## Strategy 4: Conditional Retry (Only on Specific Errors)

### What Happens

Error occurs → Check error type → Retry only if temporary error → Otherwise fail immediately

```javascript
async function retryOnTemporaryError(action, maxRetries = 3) {
  const temporaryErrors = [
    'Timeout',
    'ECONNREFUSED',
    'ENOTFOUND',
    'ERR_NETWORK',
    'net::ERR_CONNECTION_RESET'
  ];
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await action();
    } catch (error) {
      const isTemporary = temporaryErrors.some(e => error.message.includes(e));
      
      if (!isTemporary) {
        console.log('Permanent error, not retrying');
        throw error;
      }
      
      if (attempt === maxRetries) {
        console.log('Max retries exhausted');
        throw error;
      }
      
      console.log(`Temporary error, retrying (${attempt}/${maxRetries})`);
      await new Promise(r => setTimeout(r, 1000 * attempt));
    }
  }
}

// Usage
await retryOnTemporaryError(async () => {
  await page.goto('https://...');
}, 3);
```

**Best for:** Distinguishing between temporary and permanent failures

---

---

# ERROR HANDLING IN TESTS

## Example 1: Navigation Errors

### What Happens

Page navigation might fail → Retry or use alternative approach

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Handle navigation errors', async ({ page }) => {
  console.log('=== Navigation Error Handling ===\n');
  
  const urls = [
    'https://testautomationpractice.blogspot.com/',
    'https://testautomationpractice.blogspot.com/p/about.html',
    'https://testautomationpractice.blogspot.com/p/contact.html',
  ];
  
  for (const url of urls) {
    try {
      console.log(`Navigating to ${url}`);
      
      await page.goto(url, {
        waitUntil: 'networkidle',  // Wait for network to be idle
        timeout: 30000              // 30 second timeout
      });
      
      console.log('✓ Navigation succeeded');
      
    } catch (error) {
      console.log(`❌ Navigation failed: ${error.message}`);
      
      // Try again with different settings
      try {
        console.log('Retrying with different strategy...');
        await page.goto(url, {
          waitUntil: 'domcontentloaded',  // Less strict wait
          timeout: 60000                   // Longer timeout
        });
        console.log('✓ Retry succeeded\n');
        
      } catch (retryError) {
        console.log('❌ Retry also failed, continuing\n');
      }
    }
  }
});
```

---

## Example 2: Element Interaction Errors

### What Happens

Click might fail (element disabled, hidden, etc.) → Handle appropriately

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Handle element interaction errors', async ({ page }) => {
  console.log('=== Element Interaction Error Handling ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const button = page.locator('button#submit');
  
  // Check element state before interacting
  try {
    console.log('Checking button state...');
    
    const isVisible = await button.isVisible();
    console.log('Visible:', isVisible);
    
    const isEnabled = await button.isEnabled();
    console.log('Enabled:', isEnabled);
    
    if (!isVisible || !isEnabled) {
      console.log('❌ Button not ready');
      
      // Wait for button to be ready
      console.log('Waiting for button...');
      await button.waitFor({ state: 'visible', timeout: 5000 });
      console.log('✓ Button now visible');
    }
    
    // Now click
    await button.click();
    console.log('✓ Button clicked\n');
    
  } catch (error) {
    console.log(`❌ Error: ${error.message}`);
    
    // Alternative: Try JS click
    try {
      console.log('Trying JS click...');
      await page.evaluate(() => {
        document.getElementById('submit').click();
      });
      console.log('✓ JS click worked\n');
      
    } catch (jsError) {
      console.log('❌ JS click also failed\n');
      throw error;
    }
  }
});
```

---

## Example 3: Assertion Errors with Context

### What Happens

Assertion fails → Capture context (screenshot, state) → Report useful info

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Assertion error with context', async ({ page }) => {
  console.log('=== Assertion Error with Context ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  try {
    const price = await page.locator('.product-price').textContent();
    console.log('Product price:', price);
    
    // This assertion might fail
    const numericPrice = parseFloat(price.replace('$', ''));
    
    expect(numericPrice).toBeGreaterThan(100);
    
  } catch (error) {
    console.log('❌ Assertion failed');
    console.log('Error:', error.message);
    
    // Capture context
    const url = page.url();
    const title = await page.title();
    const price = await page.locator('.product-price').textContent().catch(() => 'N/A');
    
    console.log('\nContext:');
    console.log('  URL:', url);
    console.log('  Title:', title);
    console.log('  Price:', price);
    
    // Take screenshot
    await page.screenshot({
      path: `screenshots/assertion-error-${Date.now()}.png`
    });
    
    console.log('  Screenshot: saved');
    
    // Re-throw with context
    throw new Error(`Assertion failed:\n  ${error.message}\n  Context: ${url}`);
  }
});
```

---

---

# ERROR HANDLING PATTERNS

## Pattern 1: Graceful Degradation

### What Happens

Feature fails → Use fallback → Test continues

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Graceful degradation', async ({ page }) => {
  console.log('=== Graceful Degradation ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Try modern selector
  let element = await page.locator('[data-testid="product"]').first().catch(() => null);
  
  if (!element) {
    console.log('Modern selector failed, trying backup...');
    // Fallback to CSS class
    element = await page.locator('.product').first().catch(() => null);
  }
  
  if (!element) {
    console.log('CSS fallback failed, trying XPath...');
    // Final fallback to XPath
    element = await page.locator('//div[@class="product"]').first().catch(() => null);
  }
  
  if (element) {
    console.log('✓ Element found with fallback selector');
    const text = await element.textContent();
    console.log('Content:', text);
  } else {
    console.log('❌ All selectors failed');
  }
});
```

---

## Pattern 2: Retry with Different Selectors

### What Happens

Primary selector fails → Try alternative selectors → One works or all fail

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

async function findElementWithFallbacks(page, selectors) {
  for (const selector of selectors) {
    try {
      console.log(`Trying selector: ${selector}`);
      const element = page.locator(selector);
      await element.waitFor({ state: 'visible', timeout: 2000 });
      console.log('✓ Found');
      return element;
    } catch (e) {
      console.log('✗ Not found');
    }
  }
  throw new Error('Element not found with any selector');
}

test('Retry with different selectors', async ({ page }) => {
  console.log('=== Retry Different Selectors ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const selectors = [
    'button#submit',                    // Primary
    'button[type="submit"]',            // Secondary
    'button:has-text("Submit")',        // Tertiary
    '.submit-button',                   // Fallback 1
    '[role="button"][name*="submit"]',  // Fallback 2
  ];
  
  const element = await findElementWithFallbacks(page, selectors);
  await element.click();
  
  console.log('✓ Button clicked\n');
});
```

---

## Pattern 3: Timeout Escalation

### What Happens

Try with short timeout → If timeout, try with longer timeout

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

async function waitWithEscalation(selector, page) {
  const timeouts = [5000, 10000, 20000, 30000];
  
  for (const timeout of timeouts) {
    try {
      console.log(`Waiting with timeout ${timeout}ms...`);
      await page.locator(selector).waitFor({ state: 'visible', timeout });
      console.log('✓ Found');
      return;
    } catch (e) {
      console.log(`✗ Timeout after ${timeout}ms`);
    }
  }
  
  throw new Error('Element not found even with max timeout');
}

test('Timeout escalation', async ({ page }) => {
  console.log('=== Timeout Escalation ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Slowly loads element
  await page.evaluate(() => {
    setTimeout(() => {
      const el = document.createElement('div');
      el.className = 'slow-element';
      el.textContent = 'Finally here!';
      document.body.appendChild(el);
    }, 15000);  // 15 second delay
  });
  
  await waitWithEscalation('.slow-element', page);
  
  console.log('✓ Element found after escalated timeout\n');
});
```

---

---

# BEST PRACTICES

### ✅ Do This
```javascript
// Use built-in retry first (simplest)
await expect(page.locator('.element')).toBeVisible();

// Configure test retry for flaky tests
test.describe('Flaky suite', () => {
  test.describe.configure({ retries: 2 });
  test('Test', async ({ page }) => { });
});

// Use appropriate timeout for action
await page.goto(url, { timeout: 30000 });

// Try-catch for expected failures
try {
  await page.click(selector);
} catch (error) {
  // Handle gracefully
}

// Log context when assertion fails
try {
  expect(value).toBe(expected);
} catch (e) {
  console.log('Current URL:', page.url());
  console.log('Screenshot taken');
  throw e;
}

// Retry on temporary errors only
if (error.message.includes('ECONNREFUSED')) {
  // Retry - temporary error
} else if (error.message.includes('Element not found')) {
  // Don't retry - permanent error
  throw error;
}

// Use exponential backoff for server issues
await retryWithExponentialBackoff(action, 3, 1000);

// Check element state before interaction
if (await button.isEnabled()) {
  await button.click();
}
```

### ❌ Don't Do This
```javascript
// Don't use fixed delays everywhere
await new Promise(r => setTimeout(r, 5000));  // Brittle!

// Don't retry everything
// Some failures shouldn't be retried

// Don't ignore all errors
try {
  await page.click(selector);
} catch (e) {
  // Silently continue - BAD
}

// Don't use excessive retries
test.describe.configure({ retries: 10 });  // Overkill

// Don't retry only with longer timeouts
// Use exponential backoff or strategy

// Don't mix data-dependent and timing-dependent failures
// Handle separately

// Don't log nothing on failure
console.log('Test failed');  // Useless
// Better: Log context, error, state

// Don't retry assertions
// Use auto-retrying assertions instead
expect(true).toBe(false);  // Won't help to retry this
```

---

---

# COMPLETE ERROR HANDLING EXAMPLE

```javascript
import { test, expect } from '@playwright/test';

// Utility functions
async function retryWithBackoff(action, maxRetries = 3, delayMs = 1000) {
  for (let i = 1; i <= maxRetries; i++) {
    try {
      return await action();
    } catch (error) {
      if (i === maxRetries) throw error;
      const delay = delayMs * Math.pow(2, i - 1);
      await new Promise(r => setTimeout(r, delay));
    }
  }
}

async function findElement(page, selectors) {
  for (const selector of selectors) {
    try {
      await page.locator(selector).waitFor({ timeout: 2000 });
      return page.locator(selector);
    } catch (e) {
      // Continue to next selector
    }
  }
  throw new Error('Element not found');
}

test.describe('Robust Error Handling Suite', () => {
  
  test.beforeEach(async ({ page }) => {
    try {
      await page.goto('https://testautomationpractice.blogspot.com/');
      await page.waitForLoadState('networkidle');
    } catch (error) {
      console.log('Navigation failed, attempting retry');
      await page.goto('https://testautomationpractice.blogspot.com/', {
        waitUntil: 'domcontentloaded'
      });
    }
  });

  test('Robust element interaction', async ({ page }) => {
    const button = await findElement(page, [
      'button[id="submit"]',
      'button:has-text("Submit")',
      '.submit-btn'
    ]);
    
    await retryWithBackoff(
      async () => await button.click(),
      3,
      500
    );
    
    console.log('✓ Button clicked');
  });

  test('Assertion with error context', async ({ page }) => {
    try {
      const price = await page.locator('.price').textContent();
      expect(price).toContain('$');
    } catch (error) {
      const context = {
        url: page.url(),
        title: await page.title(),
        error: error.message
      };
      
      await page.screenshot({
        path: `screenshots/error-${Date.now()}.png`
      });
      
      console.error('Assertion failed with context:', context);
      throw error;
    }
  });

  test('Timeout escalation', async ({ page }) => {
    const timeouts = [5000, 10000, 20000];
    
    for (const timeout of timeouts) {
      try {
        await page.locator('.dynamic-element')
          .waitFor({ state: 'visible', timeout });
        break;
      } catch (e) {
        if (timeout === timeouts[timeouts.length - 1]) {
          throw e;
        }
      }
    }
    
    console.log('✓ Element found');
  });
});
```

---

# KEY TAKEAWAYS

- ✅ **Error** = Something fails during test
- ✅ **Retry** = Try again to overcome flakiness
- ✅ **Built-in auto-wait** = Default 30s timeout with retries
- ✅ **Auto-retrying assertions** = Use `await expect()`
- ✅ **Test retry** = Configure in playwright.config.ts
- ✅ **Custom retry** = For specific needs
- ✅ **Exponential backoff** = Best for server issues
- ✅ **Try-catch** = Handle expected errors
- ✅ **Temporary vs permanent errors** = Know which to retry
- ✅ **Log context** = Show state when errors occur
- ✅ **Graceful degradation** = Use fallbacks
- ✅ **Don't over-retry** = Use judiciously

**Proper error handling & retry logic = Resilient, stable tests!** 🎯
