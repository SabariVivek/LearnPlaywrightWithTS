# Playwright Frames & iframes

## What are Frames and iframes?

**Frame** = A separate document embedded inside a webpage.

**iframe** = HTML tag that embeds another webpage inside main page (`<iframe>` tag).

Think of it like: **Picture frame inside a poster - separate content, same display**

```
Main Page
├── Content
├── iframe (separate document)
│   ├── Different HTML
│   ├── Different CSS
│   └── Different JavaScript
└── More content
```

---

## Why Test Frames and iframes?

Real websites use iframes for:
- ✅ **Third-party plugins** - Payment processors, ads, analytics
- ✅ **Embedded content** - Maps, videos, calendars
- ✅ **Isolated components** - Chat widgets, feedback forms
- ✅ **Legacy integration** - Older systems embedded in new apps
- ✅ **Security isolation** - Untrusted content in sandbox

**Problem**: You can't directly interact with iframe content from main page.

**Solution**: Playwright provides `.frameLocator()` and `.frame()` to access iframe content.

---

## Key Difference: Main Page vs iframe

```javascript
// Looking at HTML structure:
<html>
  <body>
    <input id="mainInput">                    <!-- Accessible directly -->
    
    <iframe name="paymentFrame">
      <html>
        <body>
          <input id="cardNumber">              <!-- NOT accessible directly! -->
        </body>
      </html>
    </iframe>
  </body>
</html>

// Playwright Code:
await page.fill('#mainInput', 'test');        // Works - in main page

await page.fill('#cardNumber', '1234');       // FAILS! - in iframe
// Error: Element not found! It's inside iframe, invisible to main page

// Correct way:
const frame = page.frame({ name: 'paymentFrame' });
await frame.fill('#cardNumber', '1234');      // Works! - accessed through frame
```

---

---

# BASIC FRAME OPERATIONS

## What is a Frame Reference?

**Frame Reference** = An object pointing to an iframe's document. Like getting a reference to the iframe, then accessing its contents.

---

## Example 1: Access iframe by Name

### What Happens

Find iframe by name → Get reference → Fill input inside iframe

**HTML:**
```html
<body>
  <h1>Payment Page</h1>
  
  <iframe name="paymentFrame" src="payment.html"></iframe>
</body>
```

**Inside iframe (payment.html):**
```html
<body>
  <input id="cardNumber" placeholder="Card Number">
  <input id="expiry" placeholder="MM/YY">
  <button id="submit">Submit</button>
</body>
```

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Access iframe by name', async ({ page }) => {
  console.log('=== Access iframe by Name ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Get reference to iframe by name
  const frame = page.frame({ name: 'paymentFrame' });
  
  if (frame) {
    console.log('✓ Found iframe by name: paymentFrame\n');
    
    // Now use frame to interact with iframe content
    await frame.fill('#cardNumber', '4532-1488-0343-6467');
    console.log('✓ Filled card number in iframe');
    
    await frame.fill('#expiry', '12/25');
    console.log('✓ Filled expiry in iframe');
    
    await frame.click('#submit');
    console.log('✓ Clicked submit in iframe\n');
  } else {
    console.log('❌ iframe not found');
  }
});
```

**Explanation:**
- `page.frame({ name: 'paymentFrame' })` = Find iframe with name="paymentFrame"
- `frame.fill()` = Fill input inside that iframe
- `frame.click()` = Click button inside that iframe

---

## Example 2: Access iframe by Index

### What Happens

Get all iframes → Pick one by index/position → Access its content

**HTML:**
```html
<body>
  <h1>Multi-iframe Page</h1>
  
  <p>First section</p>
  <iframe id="frame1"></iframe>
  
  <p>Second section</p>
  <iframe id="frame2"></iframe>
  
  <p>Third section</p>
  <iframe id="frame3"></iframe>
</body>
```

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Access multiple iframes by index', async ({ page }) => {
  console.log('=== Access Multiple iframes ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Method 1: Get all frames
  const allFrames = page.frames();
  console.log(`Total frames on page: ${allFrames.length}`);
  console.log('(Includes main page + all iframes)\n');
  
  // Method 2: Access specific iframe by index
  // frames[0] = main page
  // frames[1] = first iframe
  // frames[2] = second iframe
  
  console.log('Frame 0 (Main):', allFrames[0].name() || 'main page');
  console.log('Frame 1:', allFrames[1].name() || 'unnamed');
  console.log('Frame 2:', allFrames[2].name() || 'unnamed');
  
  // Interact with first iframe (index 1)
  const firstIframe = allFrames[1];
  const input = await firstIframe.locator('input').first();
  
  if (input) {
    await input.fill('Test data');
    console.log('\n✓ Filled input in first iframe');
  }
});
```

---

## Example 3: Access iframe by Locator Strategy

### What Happens

Use frameLocator() to find and access iframe in one step

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Access iframe using frameLocator', async ({ page }) => {
  console.log('=== Using frameLocator ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Method 1: Using frameLocator (modern approach)
  // frameLocator finds the iframe, then you access elements inside
  
  const iframeLocator = page.frameLocator('iframe[name="payment"]');
  
  // Fill input inside iframe
  await iframeLocator.locator('#cardNumber').fill('1234-5678-9012-3456');
  console.log('✓ Filled card number using frameLocator');
  
  // Click button inside iframe
  await iframeLocator.locator('button[type="submit"]').click();
  console.log('✓ Clicked submit button\n');
  
  // Get text from iframe
  const successMsg = await iframeLocator.locator('.success-message').textContent();
  console.log('Success message:', successMsg);
});
```

**Why use frameLocator?**
- ✅ Combines finding iframe + accessing element
- ✅ Cleaner syntax
- ✅ Works with async operations

---

---

# NESTED FRAMES (FRAMES INSIDE FRAMES)

## What are Nested Frames?

**Nested Frames** = iframe inside an iframe (multiple levels).

Think of it like: **Picture frame inside another picture frame**

```
Main Page
└── iframe (Level 1)
    └── iframe (Level 2) ← Nested
        └── Content
```

---

## Example 1: Two-Level Nested Frames

### What Happens

Access main page → Go to level 1 iframe → Go to level 2 iframe → Interact

**HTML Structure:**
```html
<!-- Main page -->
<body>
  <h1>Main Page</h1>
  
  <iframe name="outerFrame" src="outer.html">
    <!-- Outer iframe content -->
    <iframe name="innerFrame" src="inner.html">
      <!-- Inner iframe content -->
      <input id="deepInput" value="">
    </iframe>
  </iframe>
</body>
```

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Nested frames - two levels', async ({ page }) => {
  console.log('=== Nested Frames (2 Levels) ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Step 1: Access outer iframe
  console.log('Step 1: Getting outer iframe');
  const outerFrame = page.frame({ name: 'outerFrame' });
  
  if (!outerFrame) {
    console.log('❌ Outer iframe not found');
    return;
  }
  console.log('✓ Found outer iframe\n');
  
  // Step 2: Access inner iframe (from within outer iframe)
  console.log('Step 2: Getting inner iframe (from outer)');
  const innerFrame = outerFrame.frame({ name: 'innerFrame' });
  
  if (!innerFrame) {
    console.log('❌ Inner iframe not found');
    return;
  }
  console.log('✓ Found inner iframe\n');
  
  // Step 3: Interact with content inside inner iframe
  console.log('Step 3: Interacting with content in inner iframe');
  await innerFrame.fill('#deepInput', 'Nested content');
  console.log('✓ Filled input in nested iframe\n');
  
  // Step 4: Verify
  const value = await innerFrame.locator('#deepInput').inputValue();
  console.log('Value in nested input:', value);
});
```

**Explanation:**
- `page.frame()` = Get outer iframe from main page
- `outerFrame.frame()` = Get inner iframe FROM outer iframe (chained)
- `innerFrame.fill()` = Fill input in inner iframe
- Access goes: Main Page → Outer iframe → Inner iframe → Content

---

## Example 2: Using frameLocator for Nested Frames

### What Happens

Chain frameLocator calls to reach deeply nested content

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Nested frames using frameLocator chain', async ({ page }) => {
  console.log('=== Nested Frames with frameLocator ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Can't chain frameLocator directly in one line
  // But can use frame() after frameLocator()
  
  // Step 1: Use frameLocator to find outer iframe
  const outerFrameLocator = page.frameLocator('iframe[name="outerFrame"]');
  console.log('✓ Located outer iframe');
  
  // Step 2: Get frame object from frameLocator
  // Then access inner iframe
  const outerFrame = page.frame({ name: 'outerFrame' });
  const innerFrame = outerFrame.frame({ name: 'innerFrame' });
  console.log('✓ Located inner iframe\n');
  
  // Step 3: Access deeply nested element
  // Using the inner frame reference
  const input = innerFrame.locator('#deepInput');
  await input.fill('Deeply nested value');
  
  console.log('✓ Filled deeply nested input');
});
```

---

## Example 3: Three-Level Nested Frames

### What Happens

Main page → Outer iframe → Middle iframe → Inner iframe → Content

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Three-level nested frames', async ({ page }) => {
  console.log('=== Three-Level Nested Frames ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Step 1: Outer iframe (from main page)
  const level1 = page.frame({ name: 'level1' });
  console.log('✓ Level 1: Outer iframe');
  
  if (!level1) return;
  
  // Step 2: Middle iframe (from level 1)
  const level2 = level1.frame({ name: 'level2' });
  console.log('✓ Level 2: Middle iframe');
  
  if (!level2) return;
  
  // Step 3: Inner iframe (from level 2)
  const level3 = level2.frame({ name: 'level3' });
  console.log('✓ Level 3: Inner iframe\n');
  
  if (!level3) return;
  
  // Step 4: Interact with deeply nested content
  await level3.fill('#input', 'Three levels deep');
  
  const result = await level3.locator('#input').inputValue();
  console.log('Input value:', result);
  console.log('✓ Successfully accessed 3-level nested content');
});
```

---

---

# WORKING WITH IFRAME CONTENT

## Example 1: Fill Form Inside iframe

### What Happens

Navigate to page with iframe → Fill form fields inside iframe → Submit

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Fill form in iframe', async ({ page }) => {
  console.log('=== Form Inside iframe ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Get iframe reference
  const frame = page.frame({ name: 'contactForm' });
  
  if (!frame) {
    console.log('❌ iframe not found');
    return;
  }
  
  console.log('Filling form in iframe...\n');
  
  // Fill text inputs
  await frame.fill('input[name="firstName"]', 'John');
  console.log('✓ First name: John');
  
  await frame.fill('input[name="lastName"]', 'Doe');
  console.log('✓ Last name: Doe');
  
  await frame.fill('input[name="email"]', 'john@example.com');
  console.log('✓ Email: john@example.com');
  
  // Fill textarea
  await frame.fill('textarea[name="message"]', 'This is a test message');
  console.log('✓ Message filled');
  
  // Select dropdown
  await frame.selectOption('select[name="country"]', 'USA');
  console.log('✓ Country selected: USA');
  
  // Check checkbox
  await frame.check('input[type="checkbox"]');
  console.log('✓ Checkbox checked');
  
  // Submit form
  await frame.click('button[type="submit"]');
  console.log('\n✓ Form submitted');
  
  // Verify success
  const successMsg = await frame.locator('.success-message').textContent();
  console.log('Success message:', successMsg);
});
```

---

## Example 2: Read Content from iframe

### What Happens

Get text, attributes, values from elements inside iframe

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Read content from iframe', async ({ page }) => {
  console.log('=== Reading Content from iframe ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const frame = page.frame({ name: 'dataFrame' });
  
  if (!frame) {
    console.log('❌ iframe not found');
    return;
  }
  
  console.log('Reading iframe content...\n');
  
  // Get text content
  const heading = await frame.locator('h1').textContent();
  console.log('Heading:', heading);
  
  // Get input value
  const inputValue = await frame.locator('input#username').inputValue();
  console.log('Username value:', inputValue);
  
  // Get text from multiple elements
  const rows = frame.locator('table tr');
  const rowCount = await rows.count();
  console.log(`Total rows in table: ${rowCount}`);
  
  // Get specific cell value
  const cell = await frame.locator('table tr:nth-child(2) td:nth-child(1)').textContent();
  console.log('First cell of row 2:', cell);
  
  // Get all text from paragraph
  const paragraph = await frame.locator('p.info').allTextContents();
  console.log('Paragraph contents:', paragraph);
  
  // Check if element exists in iframe
  const headerExists = await frame.locator('h1').isVisible();
  console.log('Header visible:', headerExists);
  
  // Get attribute
  const linkHref = await frame.locator('a.external').getAttribute('href');
  console.log('Link href:', linkHref);
});
```

---

## Example 3: Wait for Content in iframe

### What Happens

Wait for element inside iframe to appear before interacting

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Wait for content in iframe', async ({ page }) => {
  console.log('=== Wait for iframe Content ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const frame = page.frame({ name: 'dynamicFrame' });
  
  if (!frame) {
    console.log('❌ iframe not found');
    return;
  }
  
  console.log('Waiting for data to load in iframe...\n');
  
  // Wait for element to appear
  await frame.waitForSelector('#data-loaded', { timeout: 5000 });
  console.log('✓ Data loaded indicator found');
  
  // Wait for specific content to be visible
  await frame.locator('text=Success').waitFor({ state: 'visible' });
  console.log('✓ Success message appeared');
  
  // Wait with custom timeout
  try {
    await frame.waitForSelector('.expensive-content', { timeout: 10000 });
    console.log('✓ Expensive content loaded');
  } catch (e) {
    console.log('⚠ Content did not load within timeout');
  }
  
  // Get content after waiting
  const content = await frame.locator('.data-content').textContent();
  console.log('Content:', content);
});
```

---

---

# IFRAME DETECTION & NAVIGATION

## Example 1: Find All iframes on Page

### What Happens

List all iframes → Check their names/ids → Choose which to use

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Detect all iframes', async ({ page }) => {
  console.log('=== Detecting All iframes ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Method 1: Get all frames
  const allFrames = page.frames();
  
  console.log(`Total frames: ${allFrames.length}`);
  console.log('(Includes main page)\n');
  
  console.log('Frame Details:');
  console.log('-'.repeat(50));
  
  for (let i = 0; i < allFrames.length; i++) {
    const frame = allFrames[i];
    const name = frame.name() || '(unnamed)';
    const parentFrame = frame.parentFrame();
    const childFrames = frame.childFrames();
    
    console.log(`\nFrame ${i}:`);
    console.log(`  Name: ${name}`);
    console.log(`  Has parent: ${parentFrame ? 'Yes' : 'No'}`);
    console.log(`  Child frames: ${childFrames.length}`);
    
    // Get iframe element URL (if available)
    try {
      const url = frame.url();
      console.log(`  URL: ${url}`);
    } catch (e) {
      console.log(`  URL: Not accessible`);
    }
  }
  
  console.log('\n' + '-'.repeat(50));
});
```

---

## Example 2: Navigate Within iframe

### What Happens

Click link inside iframe that navigates to new page within iframe

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Navigate within iframe', async ({ page }) => {
  console.log('=== Navigation Inside iframe ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const frame = page.frame({ name: 'contentFrame' });
  
  if (!frame) {
    console.log('❌ iframe not found');
    return;
  }
  
  console.log('Initial page in iframe loaded');
  
  // Get initial content
  const initialTitle = await frame.locator('h1').textContent();
  console.log('Initial title:', initialTitle, '\n');
  
  // Click link to navigate within iframe
  console.log('Clicking link to navigate...');
  await frame.click('a:has-text("Go to Products")');
  
  // Wait for navigation
  await frame.waitForLoadState('networkidle');
  console.log('✓ Navigation complete\n');
  
  // Get new content
  const newTitle = await frame.locator('h1').textContent();
  console.log('New title:', newTitle);
  
  // Can go back (iframe has history)
  console.log('\nGoing back...');
  await frame.goBack();
  await frame.waitForLoadState('networkidle');
  
  const backTitle = await frame.locator('h1').textContent();
  console.log('Back to title:', backTitle);
});
```

---

---

# PRACTICAL EXAMPLES

## Example 1: Payment Processor (Stripe-like iframe)

### Scenario

Checkout page with payment form in iframe → Fill card details → Submit

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Payment form in iframe', async ({ page }) => {
  console.log('=== Payment Processor iframe ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/checkout');
  
  // Get payment iframe
  const paymentFrame = page.frame({ name: 'stripeFrame' });
  
  if (!paymentFrame) {
    console.log('❌ Payment iframe not found');
    return;
  }
  
  console.log('Payment frame found\n');
  
  // Fill shipping info (main page)
  console.log('1. Filling shipping address...');
  await page.fill('input[name="address"]', '123 Main St');
  await page.fill('input[name="city"]', 'Boston');
  await page.fill('input[name="state"]', 'MA');
  console.log('✓ Shipping filled\n');
  
  // Fill payment info (inside iframe)
  console.log('2. Filling payment details in iframe...');
  await paymentFrame.fill('input[id="card-number"]', '4242424242424242');
  console.log('✓ Card number filled');
  
  await paymentFrame.fill('input[id="expiry"]', '12/25');
  console.log('✓ Expiry filled');
  
  await paymentFrame.fill('input[id="cvc"]', '123');
  console.log('✓ CVC filled\n');
  
  // Submit (button might be in main page)
  console.log('3. Submitting payment...');
  await page.click('button:has-text("Complete Purchase")');
  
  // Wait for confirmation
  await page.waitForSelector('.order-confirmation', { timeout: 5000 });
  console.log('✓ Order confirmed');
  
  // Get confirmation details
  const orderNumber = await page.locator('.order-id').textContent();
  console.log('Order number:', orderNumber);
});
```

---

## Example 2: Embedded Map (Google Maps-like iframe)

### Scenario

Page with embedded map iframe → Get map info or verify it's visible

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Embedded map in iframe', async ({ page }) => {
  console.log('=== Embedded Map iframe ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/locations');
  
  console.log('Checking for map iframe...\n');
  
  // Find map iframe
  const mapFrame = page.frameLocator('iframe[title="Location Map"]');
  
  // Verify map is visible
  const mapLoaded = await mapFrame.locator('div[role="main"]').isVisible();
  console.log('Map loaded and visible:', mapLoaded);
  
  // Get map info (if available)
  const mapTitle = await mapFrame.locator('.map-title').textContent().catch(() => null);
  console.log('Map title:', mapTitle || 'Not found');
  
  // Main page has address
  const mainAddress = await page.locator('.store-address').textContent();
  console.log('Store address:', mainAddress);
  
  console.log('\n✓ Map iframe verified');
});
```

---

## Example 3: Chat Widget (Embedded iframe)

### Scenario

Page with chat widget iframe → Send message → Read response

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Chat widget in iframe', async ({ page }) => {
  console.log('=== Chat Widget iframe ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find chat iframe
  const chatFrame = page.frame({ name: 'chatWidget' });
  
  if (!chatFrame) {
    console.log('Chat widget not found');
    return;
  }
  
  console.log('✓ Chat widget found\n');
  
  // Wait for chat to load
  await chatFrame.waitForSelector('.chat-input', { timeout: 5000 });
  console.log('✓ Chat loaded');
  
  // Send message
  console.log('Sending message...');
  await chatFrame.fill('input.chat-input', 'Hello, do you have stock?');
  await chatFrame.click('button.send-btn');
  
  console.log('✓ Message sent\n');
  
  // Wait for response
  await chatFrame.waitForSelector('.bot-message', { timeout: 5000 });
  console.log('✓ Bot responded');
  
  // Get response
  const response = await chatFrame.locator('.bot-message').last().textContent();
  console.log('Bot response:', response);
});
```

---

---

# COMMON ISSUES & SOLUTIONS

## Issue 1: "Element not found in iframe"

**Problem:**
```javascript
// This fails - looking in main page
await page.fill('#cardNumber', '1234');
// Error: Element not found
```

**Solution:**
```javascript
// Get frame first, then fill
const frame = page.frame({ name: 'payment' });
await frame.fill('#cardNumber', '1234');  // Works!
```

---

## Issue 2: "Waiting for element that doesn't exist yet"

**Problem:**
```javascript
// Iframe loads dynamically
const frame = page.frame({ name: 'dynamic' }); // Returns null initially
await frame.fill('input', 'data');  // Crashes - frame is null
```

**Solution:**
```javascript
// Wait for frame to appear first
await page.waitForSelector('iframe[name="dynamic"]', { timeout: 5000 });

// Then get reference
const frame = page.frame({ name: 'dynamic' });
await frame.fill('input', 'data');  // Now it works
```

---

## Issue 3: "Nested iframes - can't go 3 levels deep"

**Problem:**
```javascript
// Trying to access in one step (doesn't work)
const deepFrame = page.frame('level1').frame('level2').frame('level3');
```

**Solution:**
```javascript
// Access step by step
const level1 = page.frame({ name: 'level1' });
const level2 = level1.frame({ name: 'level2' });
const level3 = level2.frame({ name: 'level3' });

// Now use level3
```

---

## Issue 4: "iframe is loading - timing out"

**Problem:**
```javascript
const frame = page.frame({ name: 'data' });
// Frame not ready yet, returns null
```

**Solution:**
```javascript
// Wait for iframe to be ready
await page.waitForFunction(() => {
  return page.frame({ name: 'data' }) !== null;
}, { timeout: 5000 });

const frame = page.frame({ name: 'data' });
// Now safe to use
```

---

---

# BEST PRACTICES

### ✅ Do This
```javascript
// Use descriptive names for iframes
<iframe name="paymentForm" id="payment-iframe"></iframe>

// Get frame reference, store it
const frame = page.frame({ name: 'payment' });
if (frame) {
  await frame.fill('input', 'data');
}

// Use frameLocator for element access
await page.frameLocator('iframe[name="payment"]')
  .locator('input')
  .fill('data');

// Wait for iframe before accessing
await page.waitForSelector('iframe[name="data"]');

// Navigate within iframe (iframe can have history)
await frame.click('a');
await frame.waitForLoadState('networkidle');

// Handle nested iframes step by step
const outer = page.frame({ name: 'outer' });
const inner = outer.frame({ name: 'inner' });

// Check frame exists before using
if (frame) {
  await frame.fill('input', 'value');
}
```

### ❌ Don't Do This
```javascript
// Don't access iframe content without getting frame first
await page.fill('input-inside-iframe', 'data');  // FAILS

// Don't assume iframe is loaded immediately
const frame = page.frame({ name: 'payment' });
await frame.fill('input', 'data');  // Might crash if not ready

// Don't forget to chain steps for nested iframes
// This doesn't work:
const deepFrame = page.frame('a').frame('b').frame('c');  // Wrong!

// Don't use invalid selectors
page.frameLocator('iframe');  // Missing proper selector
page.frameLocator('iframe[name]');  // Too vague

// Don't ignore null checks
const frame = page.frame({ name: 'missing' });  // Returns null
await frame.fill('input', 'data');  // Crashes
```

---

---

# FRAME METHODS REFERENCE

## Getting Frame References

```javascript
// By name
page.frame({ name: 'frameName' })

// By index from all frames
page.frames()[2]

// Child frames of a frame
frame.childFrames()

// Parent frame
frame.parentFrame()

// All frames
page.frames()
```

---

## Accessing Elements in Frame

```javascript
// Method 1: frame.locator()
const frame = page.frame({ name: 'x' });
await frame.locator('input').fill('data');

// Method 2: frameLocator()
await page.frameLocator('iframe[name="x"]')
  .locator('input')
  .fill('data');
```

---

## Frame Navigation

```javascript
// Navigate within iframe
await frame.goto('https://...');

// Go back in iframe history
await frame.goBack();

// Go forward
await frame.goForward();

// Reload iframe
await frame.reload();
```

---

## Waiting in Frame

```javascript
// Wait for selector
await frame.waitForSelector('input', { timeout: 5000 });

// Wait for URL
await frame.waitForURL(/products/);

// Wait for function
await frame.waitForFunction(() => {
  return document.querySelectorAll('tr').length > 10;
});

// Wait for load state
await frame.waitForLoadState('networkidle');
```

---

# KEY TAKEAWAYS

- ✅ **iframe** = Separate document embedded in page
- ✅ **Can't access directly** from main page
- ✅ **Use `page.frame()`** to get reference
- ✅ **Use `frameLocator()`** for modern approach
- ✅ **Nested iframes** require step-by-step access
- ✅ **Always check if frame exists** before using
- ✅ **Wait for iframe** if loading dynamically
- ✅ **Each frame** has its own DOM/context
- ✅ **Common use cases** - payment forms, maps, chat widgets
- ✅ **Navigate within iframe** using frame methods

**Proper iframe handling = Success with embedded content!** 🎯
