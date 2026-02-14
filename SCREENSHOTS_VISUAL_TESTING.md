# Playwright Screenshots & Visual Regression Testing

## What are Screenshots & Visual Regression Tests?

**Screenshot** = Taking a picture of the webpage at a specific moment.

**Visual Regression Test** = Comparing current screenshot with a baseline to detect unwanted changes.

Think of it like: **Taking a photo of a product, then comparing it to yesterday's photo to see if anything changed**

```
First Run: Take screenshot → Save as baseline
Second Run: Take screenshot → Compare with baseline → Same? PASS | Different? FAIL
```

---

## Why Use Screenshots?

- ✅ **Visual bugs** - Things that look wrong (alignment, colors, layout)
- ✅ **Regression** - Make sure redesigns don't break old pages
- ✅ **Responsive** - Verify design on different screen sizes
- ✅ **Evidence** - Attach to reports, bug tickets
- ✅ **CI/CD** - Visual validation in automated pipelines
- ✅ **Documentation** - Visual proof of test execution

```
Problems Screenshots Catch:
- Broken CSS = Element in wrong place
- Missing image = Broken image link
- Wrong colors = CSS not applied
- Font issues = Text doesn't display right
- Layout break = Mobile responsive failed
```

---

## Screenshot vs Visual Regression

| Type | Purpose | Usage |
|------|---------|-------|
| **Screenshot** | Capture current state | Evidence, debugging, manual inspection |
| **Visual Regression** | Detect unwanted changes | Automated comparison, catch visual bugs |

---

---

# BASIC SCREENSHOTS

## Example 1: Full Page Screenshot

### What Happens

Navigate to page → Take screenshot of entire page → Save to file

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Full page screenshot', async ({ page }) => {
  console.log('=== Full Page Screenshot ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  console.log('Taking full page screenshot...');
  
  // Take screenshot of entire page
  await page.screenshot({ 
    path: 'screenshots/full-page.png'
  });
  
  console.log('✓ Screenshot saved: screenshots/full-page.png\n');
});
```

**What gets captured:**
- ✅ Everything visible on page
- ✅ Entire height (scrolls if needed)
- ✅ All content
- ✅ Images, text, buttons

**Screenshot options:**
```javascript
await page.screenshot({
  path: 'filename.png',           // Where to save
  fullPage: true,                 // Capture entire page (default)
  omitBackground: false,          // Include background (default: false)
  maximalViewport: false,         // Don't limit viewport
});
```

---

## Example 2: Element-Only Screenshot

### What Happens

Find specific element → Take screenshot of just that element

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Element screenshot', async ({ page }) => {
  console.log('=== Element Screenshot ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find element
  const button = page.locator('button:has-text("Submit")');
  
  console.log('Taking screenshot of button element...');
  
  // Take screenshot of just the element
  await button.screenshot({ 
    path: 'screenshots/submit-button.png'
  });
  
  console.log('✓ Element screenshot saved\n');
});
```

**What gets captured:**
- ✅ Only the element
- ✅ Element's box (including padding/border)
- ✅ Not surrounding page

**Use cases:**
- Button states (hover, click, disabled)
- Form validation states
- Card components
- Header/footer specific parts

---

## Example 3: Screenshot with Different ViewPorts

### What Happens

Take screenshots at different screen sizes to test responsiveness

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Screenshot at multiple viewports', async ({ browser }) => {
  console.log('=== Responsive Screenshots ===\n');
  
  const viewports = [
    { name: 'desktop', width: 1920, height: 1080 },
    { name: 'tablet', width: 768, height: 1024 },
    { name: 'mobile', width: 375, height: 812 },
  ];
  
  for (const viewport of viewports) {
    console.log(`Testing ${viewport.name} (${viewport.width}x${viewport.height})`);
    
    // Create context with specific viewport
    const context = await browser.newContext({
      viewport: {
        width: viewport.width,
        height: viewport.height,
      }
    });
    
    const page = await context.newPage();
    
    await page.goto('https://testautomationpractice.blogspot.com/');
    
    // Take screenshot at this viewport
    await page.screenshot({
      path: `screenshots/${viewport.name}-homepage.png`,
      fullPage: true
    });
    
    console.log(`✓ ${viewport.name} screenshot saved\n`);
    
    await context.close();
  }
  
  console.log('All responsive screenshots captured');
});
```

**Viewports to test:**
- 1920x1080 (Desktop)
- 1366x768 (Laptop)
- 768x1024 (Tablet)
- 375x812 (iPhone)
- 360x640 (Android)

---

## Example 4: Screenshot on Test Failure

### What Happens

Test fails → Automatically take screenshot → Save for debugging

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Screenshot on failure', async ({ page }) => {
  console.log('=== Screenshot on Failure ===\n');
  
  try {
    await page.goto('https://testautomationpractice.blogspot.com/');
    
    // Try to click element that doesn't exist
    await page.click('button:has-text("NonExistentButton")');
    
  } catch (error) {
    console.log('❌ Test failed');
    console.log('Error:', error.message);
    
    // Take screenshot when error occurs
    const timestamp = new Date().toISOString().replace(/:/g, '-');
    await page.screenshot({
      path: `screenshots/failure-${timestamp}.png`
    });
    
    console.log('✓ Failure screenshot captured');
  }
});
```

**Better approach using playwright hooks:**
```javascript
import { test } from '@playwright/test';

test.afterEach(async ({ page }, testInfo) => {
  // Automatically take screenshot only if test failed
  if (testInfo.status !== 'passed') {
    const name = testInfo.title.replace(/\s+/g, '-').toLowerCase();
    await page.screenshot({
      path: `screenshots/failed-${name}.png`
    });
    
    console.log(`✓ Failure screenshot: failed-${name}.png`);
  }
});

test('This test might fail', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  // Test code...
});
```

---

---

# VISUAL REGRESSION TESTING

## What is Visual Regression?

**Visual Regression Testing** = Comparing current screenshot with baseline screenshot to detect changes.

```
First Test Run (Create Baseline):
  Screenshot → Save as "baseline.png"

Later Test Runs (Compare):
  Screenshot → Compare with "baseline.png"
  Same pixels? → PASS
  Different pixels? → FAIL (potential design change)
```

---

## Example 1: Basic Visual Regression

### What Happens

First time: Take screenshot, save as baseline
Later: Compare new screenshot with baseline

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Visual regression - homepage', async ({ page }) => {
  console.log('=== Visual Regression Test ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Wait for all content to load
  await page.waitForLoadState('networkidle');
  
  console.log('Taking screenshot...');
  
  // Use toHaveScreenshot() - Playwright's visual regression tool
  // First run: Creates baseline
  // Later runs: Compares with baseline
  
  await expect(page).toHaveScreenshot('homepage.png', {
    maxDiffPixels: 100,  // Allow up to 100 pixels difference
    threshold: 0.1       // 10% tolerance
  });
  
  console.log('✓ Visual regression passed\n');
});
```

**How it works:**
1. **First run**: Creates `homepage.png` in `test-results/` folder
2. **Later runs**: Compares new screenshot with saved baseline
3. **If different**: Test fails, shows diff

---

## Example 2: Visual Regression for Specific Element

### What Happens

Compare specific component across tests to catch CSS/design changes

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Visual regression - navigation bar', async ({ page }) => {
  console.log('=== Component Visual Regression ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Get specific element
  const navbar = page.locator('nav.navbar');
  
  console.log('Comparing navbar with baseline...');
  
  // Compare just the navbar
  await expect(navbar).toHaveScreenshot('navbar.png', {
    maxDiffPixels: 50
  });
  
  console.log('✓ Navbar visual regression passed\n');
});
```

---

## Example 3: Visual Regression - Different States

### What Happens

Compare element in different states (normal, hover, active, disabled)

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Visual regression - button states', async ({ page }) => {
  console.log('=== Visual Regression - Button States ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const button = page.locator('button#submitBtn');
  
  // State 1: Normal
  console.log('1. Normal state');
  await expect(button).toHaveScreenshot('button-normal.png');
  
  // State 2: Hover
  console.log('2. Hover state');
  await button.hover();
  await expect(button).toHaveScreenshot('button-hover.png');
  
  // State 3: Focus
  console.log('3. Focus state');
  await button.focus();
  await expect(button).toHaveScreenshot('button-focus.png');
  
  // State 4: Disabled
  console.log('4. Disabled state');
  await page.evaluate(() => {
    document.getElementById('submitBtn').disabled = true;
  });
  await expect(button).toHaveScreenshot('button-disabled.png');
  
  console.log('✓ All button states captured\n');
});
```

---

## Example 4: Visual Regression with Tolerance

### What Happens

Allow minor differences (animations, timings) but catch major changes

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Visual regression with tolerance', async ({ page }) => {
  console.log('=== Visual Regression with Tolerance ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  await page.waitForLoadState('networkidle');
  
  // Option 1: Pixel-level tolerance
  // maxDiffPixels = how many pixels can differ
  await expect(page).toHaveScreenshot('page-with-pixels.png', {
    maxDiffPixels: 200  // Allow 200 pixels of difference
  });
  
  console.log('Option 1: Pixel tolerance used\n');
  
  // Option 2: Percentage tolerance
  // threshold = percentage of diff allowed
  await expect(page).toHaveScreenshot('page-with-percent.png', {
    threshold: 0.15  // Allow 15% difference
  });
  
  console.log('Option 2: Percentage tolerance used\n');
  
  // Option 3: Combine both
  await expect(page).toHaveScreenshot('page-combined.png', {
    maxDiffPixels: 100,
    threshold: 0.1
  });
  
  console.log('✓ All regression tests with tolerance passed\n');
});
```

**When to use tolerance:**
- ✅ Animations (slight timing differences)
- ✅ Anti-aliasing (fonts may render slightly different)
- ✅ Minor layout shifts (content loading)
- ❌ Don't overuse (defeats purpose of regression testing)

---

---

# VISUAL REGRESSION WORKFLOW

## Example 1: Create Baseline Screenshots

### What Happens

First time tests run → Generate baseline screenshots → Save for future comparison

**Playwright Configuration:**
```javascript
// playwright.config.ts
export default {
  use: {
    // Screenshot settings
    screenshot: 'only-on-failure',  // Take screenshots on failure
  },
  
  // Visual regression settings
  snapshotDir: 'test-baselines',   // Where to store baselines
  snapshotPathTemplate: '{snapshotDir}/{testFileDir}/{testFileName}-{platform}{ext}',
};
```

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Create baseline - homepage', async ({ page }) => {
  console.log('=== Creating Baseline Screenshots ===\n');
  
  const pages = [
    { name: 'homepage', url: 'https://testautomationpractice.blogspot.com/' },
    { name: 'about', url: 'https://testautomationpractice.blogspot.com/p/about.html' },
    { name: 'contact', url: 'https://testautomationpractice.blogspot.com/p/contact.html' },
  ];
  
  for (const p of pages) {
    console.log(`Creating baseline for ${p.name}...`);
    
    await page.goto(p.url);
    await page.waitForLoadState('networkidle');
    
    // First run creates, later runs compare
    await expect(page).toHaveScreenshot(`${p.name}.png`);
    
    console.log(`✓ Baseline created: ${p.name}.png\n`);
  }
});
```

**Run to create baselines:**
```bash
# First run - creates baseline files
npx playwright test --update-snapshots

# Now baseline files (*.png) are saved
# Future runs will compare against these
```

---

## Example 2: Update Baselines After Intentional Design Change

### What Happens

Designer approves new design → Update baseline → Future tests use new baseline

**Process:**
```bash
# Test fails because design changed (approved change)
npx playwright test

# Review the diff to confirm it's intentional
# Then update baseline
npx playwright test --update-snapshots

# Now new design is the baseline
# Future tests will compare against it
```

**Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Visual regression - after design update', async ({ page }) => {
  console.log('=== Visual Regression - After Update ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // If design changed:
  // 1. Run: npx playwright test → FAILS (shows diff)
  // 2. Review diff, confirm it's good
  // 3. Run: npx playwright test --update-snapshots → Updates baseline
  // 4. Run: npx playwright test → PASSES (new baseline is now standard)
  
  await expect(page).toHaveScreenshot('homepage.png');
  
  console.log('✓ Baseline up to date\n');
});
```

---

---

# PRACTICAL EXAMPLES

## Example 1: E-Commerce Product Page

### Scenario

Test visual regression of product page across devices

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Product page visual regression', async ({ page }) => {
  console.log('=== Product Page Visual Regression ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/p/product.html?id=123');
  
  await page.waitForLoadState('networkidle');
  
  // Full page comparison
  console.log('1. Full page comparison');
  await expect(page).toHaveScreenshot('product-page-full.png', {
    maxDiffPixels: 150
  });
  
  // Product image
  console.log('2. Product image');
  const image = page.locator('img.product-image');
  await expect(image).toHaveScreenshot('product-image.png');
  
  // Product details
  console.log('3. Product details section');
  const details = page.locator('.product-details');
  await expect(details).toHaveScreenshot('product-details.png');
  
  // Price section
  console.log('4. Price section');
  const price = page.locator('.price-section');
  await expect(price).toHaveScreenshot('price-section.png');
  
  // Reviews section
  console.log('5. Reviews section');
  const reviews = page.locator('.reviews');
  await expect(reviews).toHaveScreenshot('reviews-section.png');
  
  console.log('\n✓ All product sections verified\n');
});
```

---

## Example 2: Form Validation States

### Scenario

Capture form in different validation states

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Form validation visual regression', async ({ page }) => {
  console.log('=== Form Validation Visual States ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/contact');
  
  // State 1: Empty form
  console.log('1. Empty form');
  await expect(page.locator('form')).toHaveScreenshot('form-empty.png');
  
  // State 2: Partially filled
  console.log('2. Partially filled');
  await page.fill('input[name="name"]', 'John Doe');
  await expect(page.locator('form')).toHaveScreenshot('form-partial.png');
  
  // State 3: Validation error
  console.log('3. Validation error');
  await page.fill('input[name="email"]', 'invalid-email');
  await page.click('button[type="submit"]');
  await page.waitForSelector('.error-message');
  await expect(page.locator('form')).toHaveScreenshot('form-error.png');
  
  // State 4: Correct validation
  console.log('4. Correct validation');
  await page.fill('input[name="email"]', 'john@example.com');
  const errorGone = await page.locator('.error-message').isVisible().catch(() => false);
  if (!errorGone) {
    await expect(page.locator('form')).toHaveScreenshot('form-valid.png');
  }
  
  console.log('\n✓ All form states captured\n');
});
```

---

## Example 3: Mobile Responsive Visual Testing

### Scenario

Test same page on different mobile devices

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Mobile responsive visual regression', async ({ browser }) => {
  console.log('=== Mobile Responsive Visual Test ===\n');
  
  const devices = [
    { name: 'iPhone 12', width: 390, height: 844 },
    { name: 'iPhone SE', width: 375, height: 667 },
    { name: 'Android', width: 360, height: 800 },
    { name: 'iPad', width: 1024, height: 1366 },
  ];
  
  for (const device of devices) {
    console.log(`Testing ${device.name} (${device.width}x${device.height})`);
    
    const context = await browser.newContext({
      viewport: { width: device.width, height: device.height }
    });
    
    const page = await context.newPage();
    
    await page.goto('https://testautomationpractice.blogspot.com/');
    await page.waitForLoadState('networkidle');
    
    // Visual regression on mobile
    const deviceName = device.name.toLowerCase().replace(/\s+/g, '-');
    await expect(page).toHaveScreenshot(`homepage-${deviceName}.png`, {
      maxDiffPixels: 100
    });
    
    console.log(`✓ ${device.name} verified\n`);
    
    await context.close();
  }
});
```

---

## Example 4: Dark Mode Comparison

### Scenario

Compare website in light and dark mode

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Light vs dark mode visual regression', async ({ page }) => {
  console.log('=== Light vs Dark Mode ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Light mode
  console.log('1. Light mode');
  // Ensure light mode is set
  await page.evaluate(() => {
    localStorage.setItem('theme', 'light');
    document.documentElement.setAttribute('data-theme', 'light');
  });
  await page.reload();
  await page.waitForLoadState('networkidle');
  
  await expect(page).toHaveScreenshot('theme-light.png');
  
  // Dark mode
  console.log('2. Dark mode');
  // Switch to dark mode
  await page.evaluate(() => {
    localStorage.setItem('theme', 'dark');
    document.documentElement.setAttribute('data-theme', 'dark');
  });
  await page.reload();
  await page.waitForLoadState('networkidle');
  
  await expect(page).toHaveScreenshot('theme-dark.png');
  
  console.log('✓ Both themes verified\n');
});
```

---

---

# HANDLING FLAKY SCREENSHOTS

## Problem: Screenshots Change Slightly Each Run

**Issues:**
- ✅ Anti-aliasing differences
- ✅ Font rendering variations
- ✅ Animation timing
- ✅ Smooth scrolling state

**Solutions:**

### Solution 1: Increase Tolerance

```javascript
await expect(page).toHaveScreenshot('page.png', {
  maxDiffPixels: 500,  // Allow more pixel differences
  threshold: 0.2       // Allow 20% difference
});
```

### Solution 2: Wait for Stability

```javascript
// Wait for all animations to complete
await page.waitForTimeout(1000);

// Wait for all content to load
await page.waitForLoadState('networkidle');

// Wait for specific elements
await page.waitForSelector('.loading', { state: 'hidden' });

// Then take screenshot
await expect(page).toHaveScreenshot('page.png');
```

### Solution 3: Hide Dynamic Content

```javascript
// Hide elements that change (timestamps, counters, etc.)
await page.evaluate(() => {
  // Hide timestamps
  document.querySelectorAll('[data-timestamp]').forEach(el => {
    el.style.visibility = 'hidden';
  });
  
  // Hide random elements
  document.querySelectorAll('.random-ad').forEach(el => {
    el.style.display = 'none';
  });
});

await expect(page).toHaveScreenshot('page.png');
```

### Solution 4: Mask Dynamic Areas

```javascript
import { test, expect } from '@playwright/test';

test('Screenshot with masked areas', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Mask (hide) specific elements from comparison
  await expect(page).toHaveScreenshot('page.png', {
    mask: [
      page.locator('[data-timestamp]'),      // Hide timestamp
      page.locator('.random-banner'),        // Hide random banner
      page.locator('[data-user-id]'),        // Hide user data
    ]
  });
});
```

---

---

# SCREENSHOT OPTIONS REFERENCE

## Common Options

```javascript
// Full page screenshot
await page.screenshot({
  path: 'filename.png',           // Save location
  fullPage: true,                 // Entire page or viewport
  omitBackground: false,          // Include background color
});

// Element screenshot
await element.screenshot({
  path: 'element.png',
  animations: 'disabled',         // Disable animations
  mask: [element],                // Mask (hide) elements
  timeout: 5000,                  // Wait timeout
});

// Visual regression
await expect(element).toHaveScreenshot('baseline.png', {
  maxDiffPixels: 100,             // Pixels allowed to differ
  threshold: 0.1,                 // Percentage tolerance (0-1)
  mask: [element],                // Hide from comparison
  timeout: 5000,
});
```

---

# BEST PRACTICES

### ✅ Do This
```javascript
// Wait for page to fully load
await page.waitForLoadState('networkidle');

// Take screenshot in consistent environment
// Same timezone, same viewport, consistent state

// Use meaningful screenshot names
await page.screenshot({ path: 'checkout-form.png' });

// Store screenshots in version control
// Git track baseline images for regression tests

// Use tolerance for acceptable variations
await expect(page).toHaveScreenshot('page.png', {
  maxDiffPixels: 200,
  threshold: 0.15
});

// Hide dynamic content (timestamps, IDs, counters)
await page.evaluate(() => {
  document.querySelectorAll('[data-dynamic]').forEach(el => {
    el.textContent = 'DYNAMIC';
  });
});

// Take multiple screenshots for different components
await expect(header).toHaveScreenshot('header.png');
await expect(footer).toHaveScreenshot('footer.png');

// Update baselines after approved design changes
// npx playwright test --update-snapshots
```

### ❌ Don't Do This
```javascript
// Don't take screenshots without waiting
await page.goto(url);
await page.screenshot();  // Page might not be ready!

// Don't use overly strict tolerance
// Makes tests brittle to minor changes

// Don't screenshot with random data
// Each run will look different - flaky tests

// Don't mix baseline updates with test runs
// Always review diffs before updating

// Don't ignore flaky screenshots
// Find root cause, don't just increase tolerance

// Don't take huge screenshots at excessive resolution
// Slows down tests, large files

// Don't screenshot elements that are off-viewport
// Need to scroll/wait for visibility first
```

---

---

# VISUAL REGRESSION WORKFLOW

## Complete Workflow Example

```javascript
import { test, expect } from '@playwright/test';

test.describe('Visual Regression Suite', () => {
  
  test.beforeEach(async ({ page }) => {
    // Consistent starting state
    await page.goto('https://testautomationpractice.blogspot.com/');
    await page.waitForLoadState('networkidle');
  });

  test('Homepage layout', async ({ page }) => {
    // First run: Creates baseline
    // Later runs: Compares with baseline
    await expect(page).toHaveScreenshot('homepage-layout.png', {
      maxDiffPixels: 100,
      threshold: 0.1
    });
  });

  test('Navigation bar', async ({ page }) => {
    const navbar = page.locator('nav');
    await expect(navbar).toHaveScreenshot('navbar.png');
  });

  test('Hero section', async ({ page }) => {
    const hero = page.locator('.hero-section');
    await expect(hero).toHaveScreenshot('hero.png');
  });

  test('Product cards', async ({ page }) => {
    const cards = page.locator('.product-card');
    
    // Each card
    for (let i = 0; i < await cards.count(); i++) {
      await expect(cards.nth(i)).toHaveScreenshot(`product-card-${i}.png`);
    }
  });

  test('Footer', async ({ page }) => {
    const footer = page.locator('footer');
    await expect(footer).toHaveScreenshot('footer.png');
  });
});
```

---

# KEY TAKEAWAYS

- ✅ **Screenshot** = Capture current state of page/element
- ✅ **Visual Regression** = Compare current with baseline to catch changes
- ✅ **Use `toHaveScreenshot()`** for visual regression testing
- ✅ **First run** = Creates baseline screenshot
- ✅ **Later runs** = Compare with baseline
- ✅ **Different fails** = Test fails, shows diff
- ✅ **Tolerance** = Allow minor variations (animations, anti-aliasing)
- ✅ **Update baselines** = After approved design changes
- ✅ **Test at multiple sizes** = Desktop, tablet, mobile
- ✅ **Hide dynamic content** = Timestamps, counters, random data
- ✅ **Wait before screenshot** = Ensures page is ready
- ✅ **Evidence of execution** = For debugging and reports

**Proper screenshot management = Catch visual bugs before production!** 🎯
