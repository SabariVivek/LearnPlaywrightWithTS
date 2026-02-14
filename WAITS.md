# Playwright Waits and Synchronization

## What are Waits and Synchronization?

**Waits** are mechanisms to make your tests wait for things that take time.

**Synchronization** is making your test script work together with the webpage - waiting when things are loading, and proceeding when things are ready.

Think of it like: **Waiting for a traffic light to turn green before crossing the street**

```
Without waits = Test runs too fast, clicks before page loads = FAILS
With waits = Test waits for page, then clicks = PASSES
```

---

## Why Do We Need Waits?

Webpages are **dynamic** - things take time to load:
- Page load
- Element appears/disappears
- Data fetches from server
- Animations complete
- JavaScript executes

Without waits, your test clicks before the element is ready → **Test fails**

---

## Three Types of Waits

1. **AutoWait** - Playwright does it automatically
2. **Explicit Wait** - You tell Playwright to wait
3. **Load States** - Wait for page to finish loading

---

# AUTOWAIT - Automatic Waiting

## What is AutoWait?

**AutoWait** means Playwright automatically waits for elements before interacting with them.

Think of it like: **Smart assistant that waits before handing you the pen**

### How AutoWait Works

When you try to interact with an element, Playwright:
1. Waits for element to appear (up to 30 seconds)
2. Waits for element to be enabled/ready
3. Waits for element to stop moving
4. Then performs the action

```
User writes: await button.click()
              ↓
Playwright: "Is button visible?" → NO → Wait
            "Is button visible?" → NO → Wait
            "Is button visible?" → YES → Click!
```

### Default AutoWait Timeout

```javascript
// Default: Playwright waits 30 seconds before failing
// Can be changed globally in playwright.config.ts

export default {
  use: {
    actionTimeout: 30000, // 30 seconds
  },
};
```

---

## Example 1: AutoWait for Click

### What Happens

Element takes 2 seconds to appear. Without AutoWait = FAIL. With AutoWait = PASS.

**HTML:**
```html
<!-- Button appears after 2 seconds (via JavaScript) -->
<div id="buttonContainer"></div>
```

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('AutoWait - click button that appears', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Button doesn't exist yet, but will appear in 2 seconds
  const button = page.locator('button:has-text("Click Me")');
  
  // Playwright automatically waits up to 30 seconds
  await button.click(); // Waits 2 seconds, then clicks
  
  console.log('✓ Button clicked (AutoWait handled the wait)');
});
```

**Explanation:**
- Button takes 2 seconds to appear
- Without AutoWait, test would fail immediately
- With AutoWait, Playwright waits and clicks
- Automatic - no extra code needed!

---

## Example 2: AutoWait for Fill

### What Happens

Input field is disabled, then becomes enabled after data loads.

**HTML:**
```html
<input type="text" id="nameField" disabled placeholder="Enter name">
<!-- enabled after 3 seconds -->
```

**Playwright Code:**
```javascript
test('AutoWait - fill disabled input', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const nameField = page.locator('#nameField');
  
  // Field is disabled now, but AutoWait waits for it to be enabled
  await nameField.fill('John Doe'); // Waits until enabled, then fills
  
  console.log('✓ Disabled field filled (AutoWait waited for enable)');
});
```

**Explanation:**
- Input is disabled initially
- Takes 3 seconds to become enabled
- AutoWait detects disabled state
- Waits until enabled
- Then fills it

---

## Example 3: AutoWait for Visibility

### What Happens

Element is hidden (display: none), then becomes visible.

**HTML:**
```html
<div id="successMessage" style="display: none">
  Form submitted successfully!
</div>
<!-- becomes visible after form submit -->
```

**Playwright Code:**
```javascript
test('AutoWait - wait for hidden element', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Fill form
  await page.locator('button[type="submit"]').click();
  
  // Message is hidden, AutoWait waits for it to appear
  const message = page.locator('#successMessage');
  
  // This automatically waits until visible (up to 30 seconds)
  const text = await message.textContent();
  
  console.log('Message:', text);
});
```

**Explanation:**
- Message hidden initially
- AutoWait detects hidden state
- After submit, message appears
- AutoWait waits, then gets text
- No explicit wait needed!

---

## Example 4: AutoWait Timeout

### What Happens

Element never appears, AutoWait times out after 30 seconds.

**Playwright Code:**
```javascript
test('AutoWait timeout', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const nonExistentButton = page.locator('button:has-text("DoesNotExist")');
  
  try {
    // Waits 30 seconds, element never appears
    await nonExistentButton.click(); // FAILS after 30 seconds
  } catch (error) {
    console.log('✓ Timeout after 30 seconds - Button never appeared');
  }
});
```

---

## AutoWait Actions (AutoWait Enabled)

These actions have **AutoWait enabled by default**:

```javascript
// These automatically wait for element to be ready
await element.click();
await element.fill('text');
await element.check();
await element.selectOption('value');
await element.type('text');
await element.press('Enter');
await element.hover();
await element.focus();
await element.screenshot();
```

---

## When AutoWait Is NOT Enough

Sometimes you need to wait for things AutoWait can't detect:

```javascript
// AutoWait not enough - need explicit wait for:

// 1. API response to complete
await page.waitForResponse(response => response.url().includes('/api'));

// 2. Network to be idle
await page.waitForLoadState('networkidle');

// 3. Custom condition
await page.waitForFunction(() => window.dataLoaded === true);

// 4. Specific element state
await page.waitForSelector('.product-loaded');
```

---

---

# EXPLICIT WAIT - Manual Wait Commands

## What is Explicit Wait?

**Explicit Wait** is when YOU tell Playwright to wait for something specific.

Think of it like: **You need to wait, so you look at your watch and count seconds**

### When to Use

- API calls complete
- Network settles down
- Custom conditions
- Page finishes loading
- Specific element state

---

## 1. waitFor() - Wait for Element State

### What Does It Do?

Waits for an element to reach a specific state (visible, hidden, enabled, disabled, attached, detached).

### Syntax
```javascript
await element.waitFor({ state: 'visible' | 'hidden' | 'attached' | 'detached' });
```

### Example 1: Wait for Element to Become Visible

**HTML:**
```html
<!-- Hidden initially -->
<div id="loader" style="display: none">Loading...</div>
<!-- Shows after 2 seconds, then hides after 5 seconds -->
```

**Playwright Code:**
```javascript
test('Wait for element to be visible', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const loader = page.locator('#loader');
  
  // Explicitly wait for loader to appear
  await loader.waitFor({ state: 'visible' });
  
  console.log('✓ Loader appeared (waited explicitly)');
  
  // Now wait for it to disappear
  await loader.waitFor({ state: 'hidden' });
  
  console.log('✓ Loader disappeared');
});
```

**Explanation:**
- `state: 'visible'` = Wait until element appears
- Default timeout = 30 seconds
- Throws error if timeout reached
- `state: 'hidden'` = Wait until element disappears

### Example 2: Wait for Button to be Enabled

**HTML:**
```html
<!-- Disabled initially -->
<button id="submitBtn" disabled>Submit</button>
<!-- enabled after validation -->
```

**Playwright Code:**
```javascript
test('Wait for button to be enabled', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Fill form (triggers validation)
  await page.locator('input[name="email"]').fill('test@example.com');
  
  // Wait for button to become enabled
  const submitBtn = page.locator('#submitBtn');
  await submitBtn.waitFor({ state: 'attached' }); // Element exists
  
  // Then click it
  await submitBtn.click();
  
  console.log('✓ Button enabled and clicked');
});
```

---

## 2. waitForNavigation() - Wait for Page Change

### What Does It Do?

Waits for browser to navigate to a new page/URL.

### Syntax
```javascript
await Promise.all([
  page.waitForNavigation(),
  page.click('a[href="/products"]') // Action that causes navigation
]);
```

### Example 1: Wait for Navigation After Click

**Playwright Code:**
```javascript
test('Wait for navigation', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Wait for page change and click together
  const [response] = await Promise.all([
    page.waitForNavigation(),
    page.click('a:has-text("Next Page")') // Click causes navigation
  ]);
  
  console.log('✓ Navigated to:', response.url());
});
```

**Explanation:**
- `waitForNavigation()` = Wait for page to change
- Must do together with action that causes navigation
- Gets the response from new page
- Useful for link clicks

### Example 2: Wait and Get Response Status

**Playwright Code:**
```javascript
test('Navigation with response check', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const [response] = await Promise.all([
    page.waitForNavigation(),
    page.click('button:has-text("Go to Products")')
  ]);
  
  // Check response status
  if (response.ok()) {
    console.log('✓ Page loaded successfully (status 200)');
  } else {
    console.log('✗ Page failed to load');
  }
});
```

---

## 3. waitForLoadState() - Wait for Page to Load

### What Does It Do?

Waits for page to finish loading (network calls complete, JavaScript runs, etc).

### Syntax
```javascript
await page.waitForLoadState('load' | 'domcontentloaded' | 'networkidle');
```

### States Explained

```javascript
// 'domcontentloaded' - HTML parsed (fastest)
await page.waitForLoadState('domcontentloaded');

// 'load' - Page fully loaded with images, styles (normal)
await page.waitForLoadState('load');

// 'networkidle' - No network activity for 500ms (slowest, most reliable)
await page.waitForLoadState('networkidle');
```

### Example 1: Wait for Network to Settle

**HTML:**
```html
<!-- Page makes AJAX calls after loading -->
```

**Playwright Code:**
```javascript
test('Wait for network idle', async ({ page }) => {
  // Navigate and wait for network to settle
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Page loaded, and all AJAX calls complete
  await page.waitForLoadState('networkidle');
  
  console.log('✓ Page fully loaded and all data fetched');
  
  // Now safe to interact
  const products = page.locator('.product');
  const count = await products.count();
  
  console.log('Products found:', count);
});
```

**Explanation:**
- `'networkidle'` = No network calls for 500ms
- Most reliable for data-heavy pages
- Ensures all AJAX complete

### Example 2: Wait After Click

**Playwright Code:**
```javascript
test('Wait for page state after action', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Click button that loads more data
  await page.click('button:has-text("Load More")');
  
  // Wait for new content to fully load
  await page.waitForLoadState('networkidle');
  
  // New content ready
  const newProducts = page.locator('.new-product');
  const count = await newProducts.count();
  
  console.log('✓ New products loaded:', count);
});
```

---

## 4. waitForFunction() - Wait for Custom Condition

### What Does It Do?

Waits until a JavaScript condition becomes true.

### Syntax
```javascript
await page.waitForFunction(() => {
  return condition; // JavaScript expression
});
```

### Example 1: Wait for JavaScript Variable

**Playwright Code:**
```javascript
test('Wait for JavaScript condition', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Wait for specific variable to be set
  await page.waitForFunction(() => {
    return window.dataLoadComplete === true;
  });
  
  console.log('✓ Data loading complete');
});
```

**Explanation:**
- Runs JavaScript on page
- Waits until it returns true
- Very flexible for custom conditions

### Example 2: Wait for Element Count

**Playwright Code:**
```javascript
test('Wait for specific number of elements', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Click "Load More"
  await page.click('button:has-text("Load More")');
  
  // Wait for at least 10 items to be loaded
  await page.waitForFunction(() => {
    const items = document.querySelectorAll('.product-item');
    return items.length >= 10;
  });
  
  console.log('✓ At least 10 products loaded');
});
```

---

## 5. waitForSelector() - Wait for CSS Selector

### What Does It Do?

Waits for element matching selector to appear.

### Syntax
```javascript
await page.waitForSelector('selector', { timeout: 5000 });
```

### Example 1: Wait for Specific Element

**Playwright Code:**
```javascript
test('Wait for selector', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Submit form
  await page.click('button[type="submit"]');
  
  // Wait for success message to appear
  await page.waitForSelector('.success-message');
  
  console.log('✓ Success message appeared');
});
```

**Explanation:**
- Waits for element to appear
- Similar to `element.waitFor()`
- But uses CSS selector directly

### Example 2: Wait with Custom Timeout

**Playwright Code:**
```javascript
test('Wait with timeout', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  try {
    // Wait only 5 seconds (not default 30)
    await page.waitForSelector('.optional-element', { timeout: 5000 });
    console.log('✓ Element appeared within 5 seconds');
  } catch {
    console.log('✗ Element did not appear in 5 seconds');
  }
});
```

---

## 6. waitForResponse() - Wait for Network Response

### What Does It Do?

Waits for specific API response to come back.

### Syntax
```javascript
const response = await page.waitForResponse(
  response => response.url().includes('/api/data')
);
```

### Example 1: Wait for API Call

**Playwright Code:**
```javascript
test('Wait for API response', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Click button that makes API call
  const [response] = await Promise.all([
    page.waitForResponse(response => 
      response.url().includes('/api/products')
    ),
    page.click('button:has-text("Fetch Products")')
  ]);
  
  const data = await response.json();
  console.log('✓ API Response:', data);
});
```

**Explanation:**
- Waits for network request with specific URL
- Catches the response
- Can read response body
- `response.url()` = Get URL
- `response.ok()` = Check if 200 status
- `await response.json()` = Get JSON data

### Example 2: Check Response Status

**Playwright Code:**
```javascript
test('Check API response status', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const [response] = await Promise.all([
    page.waitForResponse(response => 
      response.url().includes('/api/search')
    ),
    page.click('button:has-text("Search")')
  ]);
  
  if (response.status() === 200) {
    console.log('✓ Search successful');
  } else if (response.status() === 404) {
    console.log('✗ Not found');
  } else if (response.status() === 500) {
    console.log('✗ Server error');
  }
});
```

---

---

# LOAD STATES - Page Loading Status

## What are Load States?

**Load States** are different stages of page loading.

Think of it like: **Traffic light colors - Red (loading), Yellow (DOM ready), Green (fully loaded)**

## Three Load States

### 1. 'domcontentloaded'

- HTML parsed
- DOM ready
- CSS ready
- JavaScript might still be running

**Fastest, but JavaScript might not be done**

```javascript
await page.goto(url);
await page.waitForLoadState('domcontentloaded');
// Page structure ready, but might be loading data
```

### 2. 'load'

- All resources loaded
- Images, styles, fonts loaded
- All visible page loaded
- Some background AJAX might continue

**Normal speed, most common**

```javascript
await page.goto(url);
await page.waitForLoadState('load');
// Page visually ready, background tasks might continue
```

### 3. 'networkidle'

- No network activity for 500ms
- All AJAX calls complete
- All background requests done
- Page and all data fully ready

**Slowest but most reliable**

```javascript
await page.goto(url);
await page.waitForLoadState('networkidle');
// Everything loaded, no pending requests
```

---

## Example 1: Compare Load States

**Playwright Code:**
```javascript
test('Different load states', async ({ page }) => {
  // Start timer
  const startTime = Date.now();
  
  // 1. Go to page - default waits for 'load' state
  await page.goto('https://testautomationpractice.blogspot.com/');
  console.log('Page navigated');
  
  // 2. Wait for DOM ready (fastest)
  await page.waitForLoadState('domcontentloaded');
  let elapsed = Date.now() - startTime;
  console.log(`domcontentloaded: ${elapsed}ms`);
  
  // 3. Wait for full load
  await page.waitForLoadState('load');
  elapsed = Date.now() - startTime;
  console.log(`load: ${elapsed}ms`);
  
  // 4. Wait for network idle (slowest)
  await page.waitForLoadState('networkidle');
  elapsed = Date.now() - startTime;
  console.log(`networkidle: ${elapsed}ms`);
  
  console.log('✓ All load states complete');
});
```

**Output Example:**
```
Page navigated
domcontentloaded: 245ms
load: 1230ms
networkidle: 2105ms
✓ All load states complete
```

---

## Example 2: Choose Right Load State

**Playwright Code:**
```javascript
test('Choose load state based on need', async ({ page }) => {
  // For simple static pages - domcontentloaded is fine
  await page.goto('https://testautomationpractice.blogspot.com/about');
  await page.waitForLoadState('domcontentloaded');
  
  const heading = await page.locator('h1').textContent();
  console.log('About page:', heading);
  
  // For pages with images - wait for 'load'
  await page.goto('https://testautomationpractice.blogspot.com/products');
  await page.waitForLoadState('load');
  
  const imageCount = await page.locator('img').count();
  console.log('Images loaded:', imageCount);
  
  // For data-heavy pages - wait for 'networkidle'
  await page.goto('https://testautomationpractice.blogspot.com/dashboard');
  await page.waitForLoadState('networkidle');
  
  const dataPoints = await page.locator('[data-loaded="true"]').count();
  console.log('Data items loaded:', dataPoints);
});
```

---

## Example 3: Wait After Action

**Playwright Code:**
```javascript
test('Wait for load state after action', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Click button that opens new page
  await page.click('a:has-text("Open New")');
  
  // Wait for new page to load
  await page.waitForLoadState('networkidle');
  
  // Check new page loaded
  const newPageContent = await page.locator('h1').textContent();
  console.log('✓ New page loaded:', newPageContent);
});
```

---

---

# COMPLETE EXAMPLES

## Complete Example 1: Form Submission with Waits

```javascript
import { test } from '@playwright/test';

test('Form submission with various waits', async ({ page }) => {
  console.log('--- Starting test ---');
  
  // Navigate with default load state (load)
  await page.goto('https://testautomationpractice.blogspot.com/');
  console.log('✓ Page loaded');
  
  // AutoWait: Input appears dynamically
  const emailInput = page.getByLabel('Email:');
  await emailInput.fill('user@example.com');
  console.log('✓ Email filled (AutoWait handled this)');
  
  // AutoWait: Button becomes enabled after filling
  const submitBtn = page.locator('button[type="submit"]');
  await submitBtn.click(); // AutoWait waits for enabled
  console.log('✓ Form submitted');
  
  // Explicit Wait: Wait for success message
  const successMsg = page.locator('.success-message');
  await successMsg.waitFor({ state: 'visible' });
  console.log('✓ Success message visible (explicit wait)');
  
  // Load State Wait: Wait for page to load
  await page.waitForLoadState('networkidle');
  console.log('✓ All data loaded (network idle)');
  
  console.log('--- Test complete ---');
});
```

---

## Complete Example 2: Multi-Step Workflow

```javascript
import { test } from '@playwright/test';

test('Complete workflow with all wait types', async ({ page }) => {
  // 1. Navigate
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // 2. AutoWait: Click dynamic button
  const loadMoreBtn = page.locator('button:has-text("Load More")');
  await loadMoreBtn.click(); // Waits for button to be clickable
  
  // 3. Explicit Wait: Wait for content to appear
  const newContent = page.locator('.new-items');
  await newContent.waitFor({ state: 'visible', timeout: 10000 });
  
  // 4. Load State: Wait for data fetch
  await page.waitForLoadState('networkidle');
  
  // 5. Explicit Wait: Wait for specific count
  await page.waitForFunction(() => {
    const items = document.querySelectorAll('.item');
    return items.length >= 20;
  });
  
  // 6. AutoWait: Fill search (no explicit wait needed)
  const searchBox = page.getByPlaceholder('Search items');
  await searchBox.fill('laptop');
  
  // 7. Explicit Wait: Wait for search results
  const results = page.locator('.search-results');
  await results.waitFor({ state: 'visible' });
  
  console.log('✓ All steps completed');
});
```

---

# WHEN TO USE WHAT

## Quick Decision Tree

```
WAITING FOR...
    ↓
Button click to work?
    → Use AutoWait (no code needed)
    
Element to appear?
    → Use waitFor({ state: 'visible' })
    
Page to fully load?
    → Use waitForLoadState('networkidle')
    
Navigation to happen?
    → Use waitForNavigation()
    
API call to complete?
    → Use waitForResponse()
    
Custom JavaScript condition?
    → Use waitForFunction()
    
Input to become enabled?
    → Use AutoWait (no code needed)
```

---

# SUMMARY TABLE

| Scenario | AutoWait | Explicit Wait | Load State |
|----------|----------|----------------|-----------|
| **Click button** | ✅ Use | ❌ Not needed | ❌ No |
| **Fill input** | ✅ Use | ❌ Not needed | ❌ No |
| **Wait element visible** | ❌ No | ✅ Use `waitFor()` | ❌ No |
| **Page fully loaded** | ❌ No | ❌ No | ✅ Use |
| **API response** | ❌ No | ✅ Use `waitForResponse()` | ❌ No |
| **Navigation** | ❌ No | ✅ Use `waitForNavigation()` | ❌ No |
| **Custom condition** | ❌ No | ✅ Use `waitForFunction()` | ❌ No |

---

## Best Practices

### ✅ Do This
```javascript
// Let AutoWait work
await button.click(); // No explicit wait

// Wait for state change
await element.waitFor({ state: 'visible' });

// Wait for page to settle
await page.waitForLoadState('networkidle');
```

### ❌ Don't Do This
```javascript
// Hard-coded sleep (BAD!)
await page.waitForTimeout(5000); // Never do this!

// Over-waiting
await page.waitForLoadState('networkidle'); // For simple pages

// Not letting AutoWait work
await element.waitFor({ state: 'visible' });
await element.click(); // AutoWait would handle this
```

---

## Timeout Configuration

### Set Default Timeout Globally

```javascript
// In playwright.config.ts
export default {
  use: {
    actionTimeout: 30000, // 30 seconds for actions
    navigationTimeout: 30000, // 30 seconds for navigation
  },
};
```

### Override Timeout Per Test

```javascript
test('With custom timeout', async ({ page }) => {
  // This action waits max 60 seconds
  await page.locator('button').click({ timeout: 60000 });
  
  // This wait times out after 5 seconds
  await page.waitForSelector('.element', { timeout: 5000 });
});
```

---

## Summary

**Three wait mechanisms work together:**

1. **AutoWait** (Automatic)
   - No code needed
   - Waits before any interaction
   - Default 30 seconds

2. **Explicit Wait** (Manual)
   - You control what to wait for
   - waitFor(), waitForFunction(), waitForResponse(), etc.
   - More specific

3. **Load States** (Page Loading)
   - Wait for page to reach state
   - domcontentloaded, load, networkidle
   - Ensures entire page ready

**Use all three together for reliable, robust tests!** 🎯

---

## Key Takeaways

- ✅ AutoWait handles most cases automatically
- ✅ Use explicit waits when AutoWait isn't enough
- ✅ Use load states when entire page needs to be ready
- ✅ Never use hard-coded sleep (waitForTimeout)
- ✅ Configure timeouts based on your needs
- ✅ Test will wait if needed, then proceed when ready

**Proper synchronization = Stable, flaky-free tests!** 🎯
