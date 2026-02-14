# Playwright Multiple Tabs/Windows

## What are Multiple Tabs/Windows?

**Multiple Tabs** = Opening multiple browser tabs/pages from same context.

**Multiple Windows** = Opening multiple separate browser windows/contexts.

Think of it like: **One person with multiple notebooks (tabs) vs multiple people in same room (windows)**

```
Single Tab:
  Browser → Page 1

Multiple Tabs (Same Session):
  Browser → Context → Page 1
                   → Page 2
                   → Page 3

Multiple Windows (Different Sessions):
  Browser → Context 1 → Page 1
         → Context 2 → Page 2
```

---

## Why Use Multiple Tabs/Windows?

- ✅ **Multi-user testing** - Admin + customer same time
- ✅ **Cross-tab communication** - Share data between tabs
- ✅ **Parallel workflows** - Test multiple flows simultaneously
- ✅ **Real-world scenarios** - Users open multiple tabs
- ✅ **Concurrent testing** - Load testing multiple users

---

---

# BASIC MULTIPLE TABS

## What is a Tab?

**Tab** = A page in same browser context (shares cookies, storage).

---

## Example 1: Create Multiple Tabs

### What Happens

Open page 1 → Open page 2 → Open page 3 → Use all together

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Multiple tabs', async ({ context }) => {
  console.log('=== Multiple Tabs Test ===\n');
  
  // Create 3 tabs (pages) in same context
  const page1 = await context.newPage();
  const page2 = await context.newPage();
  const page3 = await context.newPage();
  
  console.log('✓ 3 tabs created\n');
  
  // Navigate each tab to different pages
  console.log('Navigating tabs...');
  
  await page1.goto('https://testautomationpractice.blogspot.com/');
  await page2.goto('https://testautomationpractice.blogspot.com/p/about.html');
  await page3.goto('https://testautomationpractice.blogspot.com/p/contact.html');
  
  console.log('✓ All tabs loaded\n');
  
  // Get info from each tab
  const title1 = await page1.title();
  const title2 = await page2.title();
  const title3 = await page3.title();
  
  console.log('Tab 1 title:', title1);
  console.log('Tab 2 title:', title2);
  console.log('Tab 3 title:', title3);
  
  // Close tabs
  await page1.close();
  await page2.close();
  await page3.close();
  
  console.log('\n✓ All tabs closed');
});
```

---

## Example 2: Switch Between Tabs

### What Happens

Do something in tab 1 → Switch to tab 2 → Do something → Switch to tab 1 → Continue

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Switch between tabs', async ({ context }) => {
  // Create 2 tabs
  const tab1 = await context.newPage();
  const tab2 = await context.newPage();
  
  // Tab 1: Login
  console.log('Tab 1: Logging in');
  await tab1.goto('https://testautomationpractice.blogspot.com/');
  await tab1.fill('input[name="username"]', 'admin');
  console.log('✓ Tab 1: Username filled\n');
  
  // Tab 2: Browse products
  console.log('Tab 2: Browsing products');
  await tab2.goto('https://testautomationpractice.blogspot.com/');
  const products = await tab2.locator('.product').count();
  console.log(`✓ Tab 2: Found ${products} products\n`);
  
  // Back to Tab 1: Complete login
  console.log('Back to Tab 1: Completing login');
  await tab1.fill('input[name="password"]', 'password123');
  await tab1.click('button:has-text("Login")');
  console.log('✓ Tab 1: Logged in\n');
  
  // Check Tab 2 again
  console.log('Check Tab 2 again');
  const url2 = tab2.url();
  console.log('Tab 2 URL:', url2);
  
  await tab1.close();
  await tab2.close();
});
```

---

## Example 3: Share Data Between Tabs

### What Happens

Tab 1 logs in → Extract cookie → Tab 2 uses same cookie → Both logged in

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Share cookies between tabs', async ({ context }) => {
  console.log('=== Sharing Cookies Between Tabs ===\n');
  
  // Tab 1: Login
  console.log('Tab 1: Login');
  const tab1 = await context.newPage();
  
  await tab1.goto('https://testautomationpractice.blogspot.com/');
  await tab1.fill('input[name="username"]', 'admin');
  await tab1.fill('input[name="password"]', 'password123');
  await tab1.click('button:has-text("Login")');
  
  await tab1.waitForLoadState('networkidle');
  console.log('✓ Tab 1: Logged in\n');
  
  // Tab 2: Already logged in (shares cookies!)
  console.log('Tab 2: Should be logged in automatically');
  const tab2 = await context.newPage();
  
  await tab2.goto('https://testautomationpractice.blogspot.com/dashboard');
  
  // Tab 2 is logged in because cookies are shared in same context
  const dashboard = await tab2.locator('.dashboard').isVisible();
  console.log('Dashboard visible in Tab 2:', dashboard);
  console.log('✓ Tab 2: Logged in automatically (cookie shared)\n');
  
  // Both tabs are logged in!
  const user1 = await tab1.locator('.user-name').textContent();
  const user2 = await tab2.locator('.user-name').textContent();
  
  console.log('Tab 1 user:', user1);
  console.log('Tab 2 user:', user2);
  
  await tab1.close();
  await tab2.close();
});
```

---

---

# POPUP WINDOWS

## What is a Popup Window?

**Popup** = New window that opens when link clicked (usually with `window.open()`).

---

## Example 1: Handle Popup Window

### What Happens

Click link → New window opens → Interact with popup

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Handle popup window', async ({ page }) => {
  console.log('=== Popup Window Test ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  console.log('Main window loaded');
  
  // Wait for popup to open
  const popupPromise = page.waitForEvent('popup');
  
  // Click link that opens popup
  await page.click('a:has-text("Open Popup")');
  
  // Get popup
  const popup = await popupPromise;
  console.log('✓ Popup opened');
  console.log('Popup URL:', popup.url());
  
  // Do stuff in popup
  const popupTitle = await popup.title();
  console.log('Popup title:', popupTitle);
  
  // Back to main window
  const mainTitle = await page.title();
  console.log('Main window title:', mainTitle);
  
  // Close popup
  await popup.close();
  console.log('✓ Popup closed');
});
```

---

## Example 2: Multiple Popups

### What Happens

Open multiple popups → Manage each one separately

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Handle multiple popups', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const popups = [];
  
  // Listen for all popups
  page.on('popup', popup => {
    console.log('Popup opened:', popup.url());
    popups.push(popup);
  });
  
  // Open 3 popups
  await page.click('a:has-text("Popup 1")');
  await page.click('a:has-text("Popup 2")');
  await page.click('a:has-text("Popup 3")');
  
  // Wait for all popups
  await page.waitForTimeout(1000);
  
  console.log(`Total popups: ${popups.length}`);
  
  // Do something in each popup
  for (let i = 0; i < popups.length; i++) {
    console.log(`Popup ${i + 1}:`, popups[i].url());
    
    // Get title
    const title = await popups[i].title();
    console.log(`  Title: ${title}`);
    
    // Close it
    await popups[i].close();
  }
  
  console.log('✓ All popups handled');
});
```

---

---

# CROSS-TAB COMMUNICATION

## What is Cross-Tab Communication?

**Cross-Tab Comm** = Sharing data/state between different tabs/windows.

Think of it like: **Messages passed between tabs, or shared storage**

---

## Method 1: Shared localStorage

### What Happens

Tab 1 stores data in localStorage → Tab 2 reads same storage

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Communication via localStorage', async ({ context }) => {
  console.log('=== localStorage Communication ===\n');
  
  // Tab 1: Store data
  console.log('Tab 1: Storing data in localStorage');
  const tab1 = await context.newPage();
  
  await tab1.goto('https://testautomationpractice.blogspot.com/');
  
  await tab1.evaluate(() => {
    localStorage.setItem('user_id', '12345');
    localStorage.setItem('session_token', 'abc123xyz');
  });
  
  const stored1 = await tab1.evaluate(() => localStorage.getItem('user_id'));
  console.log('Tab 1 stored user_id:', stored1, '\n');
  
  // Tab 2: Read same data
  console.log('Tab 2: Reading data from localStorage');
  const tab2 = await context.newPage();
  
  await tab2.goto('https://testautomationpractice.blogspot.com/');
  
  // Tab 2 can read same localStorage!
  const stored2 = await tab2.evaluate(() => localStorage.getItem('user_id'));
  const token = await tab2.evaluate(() => localStorage.getItem('session_token'));
  
  console.log('Tab 2 read user_id:', stored2);
  console.log('Tab 2 read token:', token);
  console.log('\n✓ Cross-tab communication via localStorage');
  
  await tab1.close();
  await tab2.close();
});
```

---

## Method 2: Shared sessionStorage

### What Happens

Tab 1 stores data → Tab 2 accesses same sessionStorage

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Communication via sessionStorage', async ({ context }) => {
  console.log('=== sessionStorage Communication ===\n');
  
  const tab1 = await context.newPage();
  const tab2 = await context.newPage();
  
  // Tab 1: Store temporary data
  console.log('Tab 1: Store in sessionStorage');
  await tab1.goto('https://testautomationpractice.blogspot.com/');
  
  await tab1.evaluate(() => {
    sessionStorage.setItem('order_id', 'ORD-2026-001');
    sessionStorage.setItem('cart_items', '5');
  });
  
  console.log('✓ Stored\n');
  
  // Tab 2: Access same data
  console.log('Tab 2: Access sessionStorage');
  await tab2.goto('https://testautomationpractice.blogspot.com/');
  
  const orderId = await tab2.evaluate(() => sessionStorage.getItem('order_id'));
  const cartItems = await tab2.evaluate(() => sessionStorage.getItem('cart_items'));
  
  console.log('Order ID:', orderId);
  console.log('Cart items:', cartItems);
  console.log('\n✓ Data shared between tabs');
  
  await tab1.close();
  await tab2.close();
});
```

---

## Method 3: JavaScript Variables

### What Happens

Tab 1 sets window variable → Tab 2 reads same window variable

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Communication via window variables', async ({ context }) => {
  console.log('=== Window Variables Communication ===\n');
  
  // Tab 1: Set window variable
  console.log('Tab 1: Setting window variable');
  const tab1 = await context.newPage();
  
  await tab1.goto('https://testautomationpractice.blogspot.com/');
  
  await tab1.evaluate(() => {
    window.appConfig = {
      theme: 'dark',
      language: 'en',
      userRole: 'admin'
    };
  });
  
  const config1 = await tab1.evaluate(() => window.appConfig);
  console.log('Tab 1 config:', config1, '\n');
  
  // Tab 2: Access same variable
  console.log('Tab 2: Accessing window variable');
  const tab2 = await context.newPage();
  
  await tab2.goto('https://testautomationpractice.blogspot.com/');
  
  // Note: Window variables don't automatically share across tabs
  // You need to set them separately OR use storage
  
  const config2 = await tab2.evaluate(() => window.appConfig);
  console.log('Tab 2 config:', config2); // Will be undefined
  
  // Better: Copy from localStorage
  await tab1.evaluate(() => {
    localStorage.setItem('appConfig', JSON.stringify(window.appConfig));
  });
  
  const configFromStorage = await tab2.evaluate(() => {
    return JSON.parse(localStorage.getItem('appConfig') || '{}');
  });
  
  console.log('Tab 2 via storage:', configFromStorage);
  console.log('\n✓ Data shared via storage');
  
  await tab1.close();
  await tab2.close();
});
```

---

---

# MULTIPLE WINDOWS (DIFFERENT CONTEXTS)

## What is a Separate Window?

**Separate Window** = Different browser context (separate cookies, storage).

Good for: Different users, different sessions

---

## Example 1: Two Users, Two Windows

### What Happens

Window 1: Admin user → Window 2: Customer user → Test interaction

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Two users in separate windows', async ({ browser }) => {
  console.log('=== Two User Windows Test ===\n');
  
  // Window 1: Admin user
  console.log('Window 1: Admin user');
  const context1 = await browser.newContext();
  const page1 = await context1.newPage();
  
  await page1.goto('https://testautomationpractice.blogspot.com/');
  await page1.fill('input[name="username"]', 'admin');
  await page1.fill('input[name="password"]', 'admin123');
  await page1.click('button:has-text("Login")');
  
  await page1.waitForLoadState('networkidle');
  
  const user1 = await page1.locator('.user-name').textContent();
  console.log('Window 1 logged in as:', user1, '\n');
  
  // Window 2: Customer user
  console.log('Window 2: Customer user');
  const context2 = await browser.newContext();
  const page2 = await context2.newPage();
  
  await page2.goto('https://testautomationpractice.blogspot.com/');
  await page2.fill('input[name="username"]', 'customer1');
  await page2.fill('input[name="password"]', 'customer123');
  await page2.click('button:has-text("Login")');
  
  await page2.waitForLoadState('networkidle');
  
  const user2 = await page2.locator('.user-name').textContent();
  console.log('Window 2 logged in as:', user2, '\n');
  
  // Both are logged in as different users!
  console.log('✓ Two separate user sessions\n');
  
  // Admin performs action
  console.log('Window 1 (Admin): Creating product');
  await page1.click('a:has-text("Create Product")');
  
  // Customer sees the change
  console.log('Window 2 (Customer): Refreshing to see new product');
  await page2.reload();
  
  console.log('✓ Cross-window action verified\n');
  
  // Cleanup
  await context1.close();
  await context2.close();
});
```

---

## Example 2: Admin Panel vs User View

### What Happens

Admin deletes user → User window shows user is gone

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Admin and user actions', async ({ browser }) => {
  console.log('=== Admin and User Interaction ===\n');
  
  // Admin window
  console.log('Admin Window: Logging in');
  const adminContext = await browser.newContext();
  const adminPage = await adminContext.newPage();
  
  await adminPage.goto('https://testautomationpractice.blogspot.com/admin');
  await adminPage.fill('input[name="username"]', 'admin');
  await adminPage.fill('input[name="password"]', 'admin123');
  await adminPage.click('button:has-text("Login")');
  
  await adminPage.waitForLoadState('networkidle');
  console.log('✓ Admin logged in\n');
  
  // User window
  console.log('User Window: Logging in');
  const userContext = await browser.newContext();
  const userPage = await userContext.newPage();
  
  await userPage.goto('https://testautomationpractice.blogspot.com/');
  await userPage.fill('input[name="username"]', 'testuser');
  await userPage.fill('input[name="password"]', 'testpass');
  await userPage.click('button:has-text("Login")');
  
  await userPage.waitForLoadState('networkidle');
  console.log('✓ User logged in\n');
  
  // Admin performs action
  console.log('Admin: Deleting user');
  await adminPage.click('button:has-text("Delete User")');
  console.log('✓ User deleted\n');
  
  // User tries to refresh
  console.log('User: Trying to refresh');
  await userPage.reload();
  
  // User should be logged out
  const isLoggedIn = await userPage.locator('.dashboard').isVisible().catch(() => false);
  console.log('User still logged in:', isLoggedIn);
  
  if (!isLoggedIn) {
    console.log('✓ User was logged out after deletion');
  }
  
  await adminContext.close();
  await userContext.close();
});
```

---

---

# COMPLETE EXAMPLE: E-COMMERCE WORKFLOW

## Scenario

Admin creates product → Customer sees it → Customer buys it → Admin sees order

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Complete ecommerce workflow - Admin and Customer', async ({ browser }) => {
  console.log('=== E-Commerce Workflow ===\n');
  
  // ===== ADMIN WINDOW =====
  console.log('ADMIN WINDOW: Creating product');
  const adminContext = await browser.newContext();
  const adminPage = await adminContext.newPage();
  
  await adminPage.goto('https://testautomationpractice.blogspot.com/admin');
  
  // Login
  await adminPage.fill('input[name="username"]', 'admin');
  await adminPage.fill('input[name="password"]', 'admin123');
  await adminPage.click('button:has-text("Login")');
  await adminPage.waitForLoadState('networkidle');
  
  console.log('✓ Admin logged in');
  
  // Create product
  await adminPage.click('button:has-text("Create Product")');
  
  await adminPage.fill('input[name="product-name"]', 'New Laptop');
  await adminPage.fill('input[name="product-price"]', '999.99');
  await adminPage.fill('input[name="product-description"]', 'High-performance laptop');
  
  await adminPage.click('button:has-text("Save Product")');
  
  console.log('✓ Product created: New Laptop\n');
  
  // ===== CUSTOMER WINDOW =====
  console.log('CUSTOMER WINDOW: Browsing products');
  const customerContext = await browser.newContext();
  const customerPage = await customerContext.newPage();
  
  await customerPage.goto('https://testautomationpractice.blogspot.com/');
  
  // Login
  await customerPage.fill('input[name="username"]', 'customer1');
  await customerPage.fill('input[name="password"]', 'customer123');
  await customerPage.click('button:has-text("Login")');
  await customerPage.waitForLoadState('networkidle');
  
  console.log('✓ Customer logged in');
  
  // Browse products
  await customerPage.goto('https://testautomationpractice.blogspot.com/shop');
  
  // Find the new product
  const newProduct = await customerPage.locator('text=New Laptop').isVisible();
  console.log('✓ New product visible to customer:', newProduct);
  
  // Add to cart
  await customerPage.click('button:has-text("New Laptop")')
    .then(() => customerPage.click('button:has-text("Add to Cart")'));
  
  console.log('✓ Added to cart');
  
  // Checkout
  await customerPage.click('button:has-text("Checkout")');
  
  await customerPage.fill('input[name="address"]', '123 Main St');
  await customerPage.click('button:has-text("Place Order")');
  
  await customerPage.waitForLoadState('networkidle');
  
  const orderConfirm = await customerPage.locator('.order-confirmation').isVisible();
  console.log('✓ Order placed:', orderConfirm, '\n');
  
  // ===== BACK TO ADMIN WINDOW =====
  console.log('ADMIN WINDOW: Checking orders');
  
  await adminPage.click('a:has-text("Orders")');
  await adminPage.waitForLoadState('networkidle');
  
  const orderExists = await adminPage.locator('text=customer1').isVisible();
  console.log('✓ Order from customer visible to admin:', orderExists);
  
  const orderStatus = await adminPage.locator('.order-status').textContent();
  console.log('✓ Order status:', orderStatus, '\n');
  
  console.log('=== Workflow Complete ===');
  
  await adminContext.close();
  await customerContext.close();
});
```

---

---

# TAB/WINDOW MANAGEMENT HELPERS

## Get All Pages/Tabs

```javascript
test('Manage all pages', async ({ context }) => {
  // Create pages
  const page1 = await context.newPage();
  const page2 = await context.newPage();
  const page3 = await context.newPage();
  
  // Get all pages
  const allPages = context.pages();
  console.log('Total pages:', allPages.length); // 3
  
  // Close all
  for (const page of allPages) {
    await page.close();
  }
});
```

---

## Switch to Page by URL

```javascript
async function getPageByURL(context, urlPart) {
  const pages = context.pages();
  
  for (const page of pages) {
    if (page.url().includes(urlPart)) {
      return page;
    }
  }
  
  return null;
}

test('Switch by URL', async ({ context }) => {
  const tab1 = await context.newPage();
  const tab2 = await context.newPage();
  
  await tab1.goto('https://example.com/admin');
  await tab2.goto('https://example.com/user');
  
  // Find admin page
  const adminPage = await getPageByURL(context, '/admin');
  console.log('Found admin page:', adminPage.url());
});
```

---

## Close All Except One

```javascript
async function closeAllExcept(context, keepPage) {
  const pages = context.pages();
  
  for (const page of pages) {
    if (page !== keepPage) {
      await page.close();
    }
  }
}

test('Close all except one', async ({ context }) => {
  const keep = await context.newPage();
  const close1 = await context.newPage();
  const close2 = await context.newPage();
  
  console.log('Before:', context.pages().length); // 3
  
  await closeAllExcept(context, keep);
  
  console.log('After:', context.pages().length); // 1
});
```

---

---

# BEST PRACTICES

### ✅ Do This
```javascript
// Use context for same-session tabs
const tab1 = await context.newPage();
const tab2 = await context.newPage();

// Use browser.newContext() for separate sessions
const context1 = await browser.newContext();
const context2 = await browser.newContext();

// Close pages properly
await page.close();

// Use waitForEvent for popups
const popup = await page.waitForEvent('popup');

// Share data via storage (localStorage, sessionStorage)
await page1.evaluate(() => {
  localStorage.setItem('key', 'value');
});

// Use page references with clear names
const adminPage = await context.newPage();
const customerPage = await context.newPage();
```

### ❌ Don't Do This
```javascript
// Don't create too many tabs unnecessarily
for (let i = 0; i < 100; i++) {
  await context.newPage(); // Too many!
}

// Don't assume popups are same as regular pages
// Handle with waitForEvent('popup')

// Don't forget to close pages
const page = await context.newPage();
// Missing: await page.close();

// Don't expect LocalStorage to auto-sync
// tab1.evaluate(() => localStorage.setItem('x', 'y'));
// tab2 won't see it unless you use same context

// Don't ignore memory leaks
// Test with hundreds of pages without closing
```

---

# SUMMARY TABLE

| Scenario | Use | Shares |
|----------|-----|--------|
| **Same tab** | Single page | N/A |
| **Multiple tabs** | `context.newPage()` | Cookies, localStorage |
| **Popup window** | `waitForEvent('popup')` | Cookies, localStorage |
| **Different user** | `browser.newContext()` | Nothing (isolated) |
| **Async flow** | Multiple pages + async | Page 1, 2, 3 in parallel |
| **Data sharing** | localStorage/sessionStorage | Between tabs in same context |

---

# KEY TAKEAWAYS

- ✅ **Tab** = `context.newPage()` (shares cookies)
- ✅ **Separate window** = `browser.newContext()` (isolated)
- ✅ **Popup** = `waitForEvent('popup')`
- ✅ **Switch tabs** = Keep page references, use them
- ✅ **Share data** = Use localStorage, sessionStorage
- ✅ **Parallel testing** = Multiple contexts for different users
- ✅ **Real-world scenario** = Admin + customer windows
- ✅ **Close properly** = `page.close()`, `context.close()`
- ✅ **Popup handling** = Multiple popups `page.on('popup')`

**Proper multi-tab/window management = Realistic, powerful tests!** 🎯
