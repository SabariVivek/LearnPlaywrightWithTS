# Playwright Architecture

## What is Playwright Architecture?

Playwright architecture is like a **hierarchy of objects** that control how the browser works.

Think of it like: **A company structure - with boss, departments, and employees**

```
President (Browser)
    │
    ├── Department 1 (BrowserContext)
    │   ├── Employee 1 (Page)
    │   ├── Employee 2 (Page)
    │   └── Employee 3 (Page)
    │
    └── Department 2 (BrowserContext)
        ├── Employee 1 (Page)
        └── Employee 2 (Page)
```

---

## The Main Components

### 1. Browser - The Application

**Browser** is the actual browser application (Chrome, Firefox, Safari).

Think of it like: **The computer application itself**

```javascript
Browser
    └── The entire Chrome/Firefox application running
```

---

## 2. BrowserContext - The User Session

**BrowserContext** is an isolated session within the browser. Like a separate user profile.

Think of it like: **Different user accounts on the same computer**

```javascript
Browser
    ├── BrowserContext (User 1)
    └── BrowserContext (User 2)
```

---

## 3. Page - The Tab

**Page** is a single tab or window. It's where you interact with the website.

Think of it like: **One browser tab**

```javascript
Browser
    └── BrowserContext
        ├── Page (Tab 1)
        ├── Page (Tab 2)
        └── Page (Tab 3)
```

---

# Complete Programs with Examples

## Program 1: Launch Browser

### What Does It Do?
Opens the browser application (Chrome/Firefox/Safari).

### Simple Program

```javascript
import { chromium } from '@playwright/test';

async function launchBrowser() {
  // Open the browser
  const browser = await chromium.launch();
  
  console.log('✓ Browser launched successfully');
  
  // Close it
  await browser.close();
  console.log('✓ Browser closed');
}

launchBrowser();
```

### Output
```
✓ Browser launched successfully
✓ Browser closed
```

### How It Works
1. `chromium.launch()` - Opens Chrome browser
2. `browser.close()` - Closes the browser

---

## Program 2: Launch Browser in Headed Mode

### What Does It Do?
Opens browser so you can see it (by default it's hidden/headless).

### Simple Program

```javascript
import { chromium } from '@playwright/test';

async function launchHeadedBrowser() {
  // Open browser with window visible
  const browser = await chromium.launch({ 
    headless: false  // Show the browser
  });
  
  console.log('✓ Browser opened (you can see it)');
  
  // Wait 3 seconds
  await new Promise(resolve => setTimeout(resolve, 3000));
  
  await browser.close();
  console.log('✓ Browser closed');
}

launchHeadedBrowser();
```

### Output
Browser window opens for 3 seconds, then closes automatically.

---

## Program 3: Create BrowserContext

### What Does It Do?
Creates an isolated user session in the browser.

### Simple Program

```javascript
import { chromium } from '@playwright/test';

async function createContext() {
  const browser = await chromium.launch();
  
  // Create a context (like a new user account)
  const context = await browser.newContext();
  
  console.log('✓ Context created (isolated session)');
  
  // Close context
  await context.close();
  
  await browser.close();
  console.log('✓ Everything closed');
}

createContext();
```

### Output
```
✓ Context created (isolated session)
✓ Everything closed
```

---

## Program 4: Create Multiple Contexts

### What Does It Do?
Creates 2 separate user sessions in same browser.

### Simple Program

```javascript
import { chromium } from '@playwright/test';

async function multipleContexts() {
  const browser = await chromium.launch({ headless: false });
  
  // Context 1 - User 1
  const context1 = await browser.newContext();
  console.log('✓ Context 1 created (User 1)');
  
  // Context 2 - User 2 (completely separate)
  const context2 = await browser.newContext();
  console.log('✓ Context 2 created (User 2)');
  
  // Both contexts are independent
  console.log('✓ Context 1 and Context 2 do not share cookies/data');
  
  // Close both
  await context1.close();
  await context2.close();
  
  await browser.close();
}

multipleContexts();
```

### Output
```
✓ Context 1 created (User 1)
✓ Context 2 created (User 2)
✓ Context 1 and Context 2 do not share cookies/data
```

---

## Program 5: Incognito Context (Private Mode)

### What Does It Do?
Creates a private/incognito context (like incognito mode in browser).

### Simple Program

```javascript
import { chromium } from '@playwright/test';

async function incognitoContext() {
  const browser = await chromium.launch({ headless: false });
  
  // Normal context (stores cookies, history)
  const normalContext = await browser.newContext();
  console.log('✓ Normal context created');
  
  // Incognito context (no cookies saved, fresh session)
  const incognitoContext = await browser.newContext();
  console.log('✓ Incognito context created (private/fresh)');
  
  // In incognito, cookies and data are not saved
  console.log('✓ Incognito mode = no saved data');
  
  await normalContext.close();
  await incognitoContext.close();
  await browser.close();
}

incognitoContext();
```

### Output
```
✓ Normal context created
✓ Incognito context created (private/fresh)
✓ Incognito mode = no saved data
```

---

## Program 6: Create Page (Tab)

### What Does It Do?
Creates a new tab/page within a context.

### Simple Program

```javascript
import { chromium } from '@playwright/test';

async function createPage() {
  const browser = await chromium.launch();
  const context = await browser.newContext();
  
  // Create a page (like opening a new tab)
  const page = await context.newPage();
  console.log('✓ Page created (new tab)');
  
  // Navigate to website
  await page.goto('https://testautomationpractice.blogspot.com/');
  console.log('✓ Page loaded');
  
  // Close page
  await page.close();
  
  await context.close();
  await browser.close();
}

createPage();
```

### Output
```
✓ Page created (new tab)
✓ Page loaded
```

---

## Program 7: Multiple Pages in One Context

### What Does It Do?
Creates 3 different tabs all in the same context (same user session).

### Simple Program

```javascript
import { chromium } from '@playwright/test';

async function multiplePages() {
  const browser = await chromium.launch({ headless: false });
  const context = await browser.newContext();
  
  // Create 3 pages (3 tabs)
  const page1 = await context.newPage();
  const page2 = await context.newPage();
  const page3 = await context.newPage();
  
  console.log('✓ 3 pages created (3 tabs)');
  
  // Navigate all to same website
  await page1.goto('https://testautomationpractice.blogspot.com/');
  await page2.goto('https://testautomationpractice.blogspot.com/');
  await page3.goto('https://testautomationpractice.blogspot.com/');
  
  console.log('✓ All 3 pages loaded');
  
  // Fill form in each page
  await page1.fill('input[name="firstname"]', 'John');
  await page2.fill('input[name="firstname"]', 'Jane');
  await page3.fill('input[name="firstname"]', 'Jack');
  
  console.log('✓ All 3 pages filled with different data');
  
  // All pages share same session (logged in as same user)
  console.log('✓ All pages use SAME login cookies');
  
  // Close all pages
  await page1.close();
  await page2.close();
  await page3.close();
  
  await context.close();
  await browser.close();
}

multiplePages();
```

### Output
```
✓ 3 pages created (3 tabs)
✓ All 3 pages loaded
✓ All 3 pages filled with different data
✓ All pages use SAME login cookies
```

---

## Program 8: Close Page

### What Does It Do?
Closes one page/tab while keeping others open.

### Simple Program

```javascript
import { chromium } from '@playwright/test';

async function closePage() {
  const browser = await chromium.launch({ headless: false });
  const context = await browser.newContext();
  
  // Create 2 pages
  const page1 = await context.newPage();
  const page2 = await context.newPage();
  
  await page1.goto('https://testautomationpractice.blogspot.com/');
  await page2.goto('https://testautomationpractice.blogspot.com/');
  
  console.log('✓ 2 pages open');
  
  // Close page 1
  await page1.close();
  console.log('✓ Page 1 closed');
  
  // Page 2 still open
  console.log('✓ Page 2 still working');
  
  // Close page 2
  await page2.close();
  console.log('✓ Page 2 closed');
  
  await context.close();
  await browser.close();
}

closePage();
```

### Output
```
✓ 2 pages open
✓ Page 1 closed
✓ Page 2 still working
✓ Page 2 closed
```

---

## Program 9: Close Browser

### What Does It Do?
Closes the entire browser and everything in it (all contexts, all pages).

### Simple Program

```javascript
import { chromium } from '@playwright/test';

async function closeBrowser() {
  const browser = await chromium.launch();
  const context = await browser.newContext();
  const page = await context.newPage();
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  console.log('✓ Browser open');
  
  // This closes EVERYTHING
  await browser.close();
  console.log('✓ Browser closed (all contexts and pages closed too)');
}

closeBrowser();
```

### Output
```
✓ Browser open
✓ Browser closed (all contexts and pages closed too)
```

---

## Program 10: Two Different Users Testing

### What Does It Do?
Simulates 2 different users using the website at the same time.

### Real Example - Admin vs Customer

```javascript
import { chromium } from '@playwright/test';

async function twoUsersTest() {
  const browser = await chromium.launch({ headless: false });
  
  // USER 1 - ADMIN
  const adminContext = await browser.newContext();
  const adminPage = await adminContext.newPage();
  
  // USER 2 - CUSTOMER
  const customerContext = await browser.newContext();
  const customerPage = await customerContext.newPage();
  
  console.log('✓ 2 users created (2 separate contexts)');
  
  // Admin goes to website
  await adminPage.goto('https://testautomationpractice.blogspot.com/');
  
  // Customer goes to same website
  await customerPage.goto('https://testautomationpractice.blogspot.com/');
  
  console.log('✓ Both users on website');
  
  // Admin fills form as admin
  await adminPage.fill('input[name="firstname"]', 'Admin');
  console.log('✓ Admin filled form');
  
  // Customer fills form as customer
  await customerPage.fill('input[name="firstname"]', 'Customer');
  console.log('✓ Customer filled form');
  
  // They have completely separate sessions
  console.log('✓ Admin cookies ≠ Customer cookies');
  console.log('✓ Admin data ≠ Customer data');
  
  // Admin closes browser
  await adminPage.close();
  await adminContext.close();
  console.log('✓ Admin logged out');
  
  // Customer still browsing
  console.log('✓ Customer still browsing');
  
  // Close customer
  await customerPage.close();
  await customerContext.close();
  
  await browser.close();
}

twoUsersTest();
```

### Output
```
✓ 2 users created (2 separate contexts)
✓ Both users on website
✓ Admin filled form
✓ Customer filled form
✓ Admin cookies ≠ Customer cookies
✓ Admin data ≠ Customer data
✓ Admin logged out
✓ Customer still browsing
```

---

## Program 11: Complete Architecture Example

### What Does It Do?
Shows everything together - browser, contexts, pages, and lifecycle.

### Real Example

```javascript
import { chromium } from '@playwright/test';

async function completeArchitecture() {
  console.log('\n========== BROWSER LAUNCH ==========');
  
  // Step 1: Launch Browser
  const browser = await chromium.launch({ headless: false });
  console.log('1. Browser launched (Chrome opened)');
  
  console.log('\n========== CREATE CONTEXTS ==========');
  
  // Step 2: Create Context 1 (Admin)
  const adminContext = await browser.newContext();
  console.log('2. Admin Context created (User Session 1)');
  
  // Step 3: Create Context 2 (Customer)
  const customerContext = await browser.newContext();
  console.log('3. Customer Context created (User Session 2)');
  
  console.log('\n========== CREATE PAGES ==========');
  
  // Step 4: Create Pages in Admin Context
  const adminPage1 = await adminContext.newPage();
  const adminPage2 = await adminContext.newPage();
  console.log('4a. Admin created 2 pages (2 tabs for admin)');
  
  // Step 5: Create Page in Customer Context
  const customerPage = await customerContext.newPage();
  console.log('4b. Customer created 1 page (1 tab for customer)');
  
  console.log('\n========== NAVIGATE PAGES ==========');
  
  // Admin uses his pages
  await adminPage1.goto('https://testautomationpractice.blogspot.com/');
  await adminPage2.goto('https://testautomationpractice.blogspot.com/');
  console.log('5a. Admin opened 2 tabs on website');
  
  // Customer uses his page
  await customerPage.goto('https://testautomationpractice.blogspot.com/');
  console.log('5b. Customer opened 1 tab on website');
  
  console.log('\n========== INTERACT WITH PAGES ==========');
  
  // Admin Page 1: Fill form
  await adminPage1.fill('input[name="firstname"]', 'Admin_John');
  console.log('6a. Admin Page 1: Filled first name');
  
  // Admin Page 2: Fill form
  await adminPage2.fill('input[name="firstname"]', 'Admin_Second_Tab');
  console.log('6b. Admin Page 2: Filled first name');
  
  // Customer: Fill form
  await customerPage.fill('input[name="firstname"]', 'Customer_Jane');
  console.log('6c. Customer: Filled first name');
  
  console.log('\n========== CLOSE PAGES ==========');
  
  // Close Admin pages
  await adminPage1.close();
  console.log('7a. Admin Page 1 closed');
  
  await adminPage2.close();
  console.log('7b. Admin Page 2 closed');
  
  // Close Customer page
  await customerPage.close();
  console.log('7c. Customer Page closed');
  
  console.log('\n========== CLOSE CONTEXTS ==========');
  
  // Close Admin context
  await adminContext.close();
  console.log('8a. Admin Context closed');
  
  // Close Customer context
  await customerContext.close();
  console.log('8b. Customer Context closed');
  
  console.log('\n========== CLOSE BROWSER ==========');
  
  // Close Browser
  await browser.close();
  console.log('9. Browser closed (all closed)');
  
  console.log('\n✓ Complete lifecycle finished!');
}

completeArchitecture();
```

### Output
```
========== BROWSER LAUNCH ==========
1. Browser launched (Chrome opened)

========== CREATE CONTEXTS ==========
2. Admin Context created (User Session 1)
3. Customer Context created (User Session 2)

========== CREATE PAGES ==========
4a. Admin created 2 pages (2 tabs for admin)
4b. Customer created 1 page (1 tab for customer)

========== NAVIGATE PAGES ==========
5a. Admin opened 2 tabs on website
5b. Customer opened 1 tab on website

========== INTERACT WITH PAGES ==========
6a. Admin Page 1: Filled first name
6b. Admin Page 2: Filled first name
6c. Customer: Filled first name

========== CLOSE PAGES ==========
7a. Admin Page 1 closed
7b. Admin Page 2 closed
7c. Customer Page closed

========== CLOSE CONTEXTS ==========
8a. Admin Context closed
8b. Customer Context closed

========== CLOSE BROWSER ==========
9. Browser closed (all closed)

✓ Complete lifecycle finished!
```

---

## Architecture Hierarchy Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                       BROWSER (Chrome)                      │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  CONTEXT 1 (Admin User - Logged In)                │  │
│  │  Cookies: admin_session=xyz                        │  │
│  │                                                    │  │
│  │  ┌─────────────────┐    ┌─────────────────┐      │  │
│  │  │ PAGE 1 (Tab 1)  │    │ PAGE 2 (Tab 2)  │      │  │
│  │  │                 │    │                 │      │  │
│  │  │ URL: website/   │    │ URL: website/   │      │  │
│  │  │ Form filled    │    │ Form filled    │      │  │
│  │  └─────────────────┘    └─────────────────┘      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  CONTEXT 2 (Customer User - Logged In)              │  │
│  │  Cookies: customer_session=abc                      │  │
│  │                                                    │  │
│  │  ┌─────────────────┐                              │  │
│  │  │ PAGE 1 (Tab 1)  │                              │  │
│  │  │                 │                              │  │
│  │  │ URL: website/   │                              │  │
│  │  │ Form filled    │                              │  │
│  │  └─────────────────┘                              │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Key Points to Remember

| Component | What It Is | How Many | Can Have |
|-----------|-----------|---------|----------|
| **Browser** | The application | 1 per test | Contexts |
| **Context** | User session | Multiple | Pages, Cookies |
| **Page** | Tab/Window | Multiple | Elements, URLs |

---

## Important Rules

### 1. Browser Contains Context
```javascript
Browser
  └── Can have many Contexts
```

### 2. Context Contains Pages
```javascript
Context
  └── Can have many Pages
```

### 3. All Pages in Context Share Session
```javascript
Context
  ├── Page 1 (same cookies)
  ├── Page 2 (same cookies)
  └── Page 3 (same cookies)
```

### 4. Each Context is Independent
```javascript
Context 1 (Admin) ≠ Context 2 (Customer)
- Different cookies
- Different sessions
- Different data
- Can run simultaneously
```

---

## Common Mistakes to Avoid

### ❌ Wrong - Forgetting to Close
```javascript
const browser = await chromium.launch();
const context = await browser.newContext();
const page = await context.newPage();
// Never closed - browser stays open!
```

### ✓ Right - Close in Correct Order
```javascript
const browser = await chromium.launch();
const context = await browser.newContext();
const page = await context.newPage();

await page.close();      // Close page first
await context.close();   // Close context second
await browser.close();   // Close browser last
```

---

## Quick Start Template

```javascript
import { chromium } from '@playwright/test';

async function template() {
  // 1. Launch browser
  const browser = await chromium.launch();
  
  // 2. Create context
  const context = await browser.newContext();
  
  // 3. Create page
  const page = await context.newPage();
  
  // 4. Use page
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // 5. Close in reverse order
  await page.close();
  await context.close();
  await browser.close();
}

template();
```

---

## Summary

**Playwright Architecture** is simple:
1. **Browser** = The application
2. **Context** = User session (like different user accounts)
3. **Page** = Tab within that session

All organized in a hierarchy, and you close them in reverse order!

Happy testing! 🎯
