# Playwright Test Fixtures

## What are Test Fixtures?

**Fixtures** are reusable setup and cleanup code for tests.

Think of it like: **Factory that prepares tools before work, cleans up after**

```
Before Test → Setup (Fixture) → Run Test → Cleanup (Fixture) → After Test
```

---

## Why Use Fixtures?

Without Fixtures:
```javascript
test('Test 1', async () => {
  // Repeat setup
  const browser = await chromium.launch();
  const page = await browser.newPage();
  
  // Test code
  await page.goto('...');
  
  // Repeat cleanup
  await page.close();
  await browser.close();
});

test('Test 2', async () => {
  // Same setup again!
  const browser = await chromium.launch();
  const page = await browser.newPage();
  
  // Test code
  await page.goto('...');
  
  // Same cleanup again!
  await page.close();
  await browser.close();
});
```

With Fixtures:
```javascript
test('Test 1', async ({ page }) => {
  // Setup/cleanup automatic!
  await page.goto('...');
});

test('Test 2', async ({ page }) => {
  // Setup/cleanup automatic!
  await page.goto('...');
});
```

Benefits:
- ✅ **DRY** - Don't repeat code
- ✅ **Reusable** - Use in multiple tests
- ✅ **Clean** - Tests focus on logic
- ✅ **Reliable** - Consistent setup

---

## Built-in Fixtures

Playwright provides fixtures automatically:

```javascript
test('Using built-in fixtures', async ({ 
  page,          // Browser page (tab)
  context,       // Browser context (user session)
  browser,       // Browser instance
  browserName    // Browser name (chromium, firefox, webkit)
}) => {
  // Can use all of them
  console.log('Browser:', browserName);
  
  const newPage = await context.newPage();
  await newPage.goto('...');
});
```

---

---

# BUILT-IN FIXTURES

## 1. Page Fixture

**Page** = Browser tab/window.

```javascript
test('Using page fixture', async ({ page }) => {
  // Automatically created and cleaned up
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  await page.fill('input[name="username"]', 'admin');
  
  // After test: page automatically closed
});
```

---

## 2. Context Fixture

**Context** = User session (cookies, storage, profile).

```javascript
test('Using context fixture', async ({ context }) => {
  // Create two pages in same context (same session)
  const page1 = await context.newPage();
  const page2 = await context.newPage();
  
  await page1.goto('https://testautomationpractice.blogspot.com/');
  await page2.goto('https://testautomationpractice.blogspot.com/');
  
  // Both pages share cookies/storage
  console.log('✓ Two pages in same context');
  
  // After test: context and pages automatically closed
});
```

---

## 3. Browser Fixture

**Browser** = Browser application.

```javascript
test('Using browser fixture', async ({ browser }) => {
  // Create multiple contexts in same browser
  const context1 = await browser.newContext();
  const context2 = await browser.newContext();
  
  const page1 = await context1.newPage();
  const page2 = await context2.newPage();
  
  await page1.goto('https://testautomationpractice.blogspot.com/');
  await page2.goto('https://testautomationpractice.blogspot.com/');
  
  console.log('✓ Multiple contexts in same browser');
  
  // After test: browser automatically closed
});
```

---

## 4. BrowserName Fixture

**BrowserName** = Which browser (chromium, firefox, webkit).

```javascript
test('Using browserName fixture', async ({ page, browserName }) => {
  console.log('Running on:', browserName);
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Can do browser-specific stuff
  if (browserName === 'chromium') {
    console.log('Chrome-specific test');
  } else if (browserName === 'firefox') {
    console.log('Firefox-specific test');
  }
});
```

---

---

# CUSTOM FIXTURES

## What are Custom Fixtures?

**Custom Fixtures** = Create your own reusable setup/cleanup.

Think of it like: **Create your own tool from basic tools**

---

## Basic Custom Fixture

### Structure

```javascript
import { test as base } from '@playwright/test';

// Create fixture
const test = base.extend({
  // Fixture name: setup function
  myFixture: async ({ page }, use) => {
    // SETUP (runs before test)
    console.log('Setting up');
    
    // Provide to test
    await use('fixture data here');
    
    // CLEANUP (runs after test)
    console.log('Cleaning up');
  }
});

// Use fixture
test('My test', async ({ myFixture }) => {
  console.log('Using:', myFixture);
});
```

---

## Example 1: Logged-in User Fixture

### What Happens

Fixture logs in → Test uses logged-in page → Cleanup

**Playwright Code:**
```javascript
import { test as base } from '@playwright/test';

// Create custom fixture
const test = base.extend({
  loggedInPage: async ({ page }, use) => {
    // SETUP: Login before test
    console.log('Logging in...');
    
    await page.goto('https://testautomationpractice.blogspot.com/');
    
    await page.fill('input[name="username"]', 'admin');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button:has-text("Login")');
    
    await page.waitForLoadState('networkidle');
    console.log('✓ Logged in');
    
    // Provide to test
    await use(page);
    
    // CLEANUP: Logout after test
    console.log('Logging out...');
    await page.click('button:has-text("Logout")');
    console.log('✓ Logged out');
  }
});

// Use fixture - page is already logged in!
test('User can view dashboard', async ({ loggedInPage }) => {
  const page = loggedInPage;
  
  await page.goto('https://testautomationpractice.blogspot.com/dashboard');
  
  const dashboard = await page.locator('.dashboard').isVisible();
  console.log('✓ Dashboard visible');
});

test('User can view profile', async ({ loggedInPage }) => {
  const page = loggedInPage;
  
  await page.goto('https://testautomationpractice.blogspot.com/profile');
  
  const profile = await page.locator('.profile').isVisible();
  console.log('✓ Profile visible');
});
```

**Explanation:**
- `test.extend()` = Create custom fixtures
- `loggedInPage:` = Fixture name
- Setup before `use()` = Runs before test
- `use(page)` = Give page to test
- Code after `use()` = Cleanup after test
- All tests get logged-in page automatically!

---

## Example 2: Test Data Fixture

### What Happens

Fixture creates test data → Test uses it → Cleanup deletes it

**Playwright Code:**
```javascript
import { test as base } from '@playwright/test';

// Fixture that provides test user data
const test = base.extend({
  testUser: async ({ }, use) => {
    // SETUP: Create test data
    const user = {
      username: 'testuser_' + Date.now(),
      email: 'test_' + Date.now() + '@example.com',
      password: 'SecurePass123!'
    };
    
    console.log('Test user created:', user.username);
    
    // Provide to test
    await use(user);
    
    // CLEANUP: Delete test data
    console.log('Cleaning up test user');
  }
});

// Use fixture
test('Register new user', async ({ page, testUser }) => {
  await page.goto('https://testautomationpractice.blogspot.com/register');
  
  // Fill form with test data
  await page.fill('input[name="username"]', testUser.username);
  await page.fill('input[name="email"]', testUser.email);
  await page.fill('input[name="password"]', testUser.password);
  
  // Submit
  await page.click('button:has-text("Register")');
  
  // Verify
  const success = await page.locator('.success-message').isVisible();
  console.log('✓ User registered');
});

test('User details are saved', async ({ page, testUser }) => {
  // Use same test user in multiple tests
  await page.goto('https://testautomationpractice.blogspot.com/profile');
  
  const username = await page.locator('.username').textContent();
  console.log('Username:', username);
});
```

---

## Example 3: Clean Database Fixture

### What Happens

Fixture resets database → Test runs → Cleanup again

**Playwright Code:**
```javascript
import { test as base } from '@playwright/test';

const test = base.extend({
  cleanDB: async ({ }, use) => {
    // SETUP: Clear database
    console.log('Clearing database...');
    
    // Call API to reset DB
    await fetch('https://api.example.com/reset', { method: 'POST' });
    
    console.log('✓ Database clean');
    
    // Provide nothing, just setup/cleanup
    await use();
    
    // CLEANUP: Clear again
    console.log('Cleaning database after test...');
    await fetch('https://api.example.com/reset', { method: 'POST' });
  }
});

test('Add item to empty database', async ({ page, cleanDB }) => {
  // Database is empty
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Add item
  await page.fill('input[name="itemName"]', 'New Item');
  await page.click('button:has-text("Add")');
  
  // Verify added
  const item = await page.locator('.item-list').textContent();
  console.log('✓ Item added to clean DB');
});
```

---

---

# FIXTURE SCOPE

## What is Scope?

**Scope** = How long fixture lasts.

```
test     → Recreates for each test (default)
worker   → Reuses across tests in same worker
session  → Reuses across all tests
```

---

## Test Scope (Default)

**Recreates for every test.**

```javascript
const test = base.extend({
  fixture: async ({ }, use) => {
    console.log('Setup (test scope)');
    await use('data');
    console.log('Cleanup (test scope)');
  }
});

// Output:
// Setup (test scope)
// Cleanup (test scope)
// Setup (test scope)  <- New setup for test 2
// Cleanup (test scope)
```

---

## Worker Scope

**One setup per worker, reused in multiple tests.**

Good for expensive setup (login, database connection).

```javascript
const test = base.extend({
  expensiveSetup: async ({ }, use) => {
    // Runs once per worker
    console.log('Expensive setup');
    
    const data = await expensiveOperation();
    
    await use(data);
    
    // Runs once after all tests in worker
    console.log('Expensive cleanup');
  }
});
```

---

## Session Scope

**One setup for all tests.**

Good for shared resources (server, database).

```javascript
const test = base.extend({
  sharedServer: async ({ }, use) => {
    // Runs once for all tests
    console.log('Starting server');
    
    const server = await startServer();
    
    await use(server);
    
    // Runs once after all tests
    console.log('Stopping server');
  }
});
```

---

## Example: Worker Scope Fixture

```javascript
import { test as base } from '@playwright/test';

const test = base.extend({
  loggedInPage: async ({ page }, use) => {
    // Runs once per worker (test group)
    console.log('Logging in (worker scope)');
    
    await page.goto('https://testautomationpractice.blogspot.com/');
    
    await page.fill('input[name="username"]', 'admin');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button:has-text("Login")');
    
    await page.waitForLoadState('networkidle');
    
    // Reuse this login for all tests
    await use(page);
    
    console.log('Cleanup (worker scope)');
  }
});

test('Test 1', async ({ loggedInPage }) => {
  console.log('Test 1 running (using same login)');
});

test('Test 2', async ({ loggedInPage }) => {
  console.log('Test 2 running (using same login)');
});

test('Test 3', async ({ loggedInPage }) => {
  console.log('Test 3 running (using same login)');
});

// Output:
// Logging in (worker scope)
// Test 1 running
// Test 2 running
// Test 3 running
// Cleanup (worker scope)
```

---

---

# FIXTURE DEPENDENCIES

## Use One Fixture in Another

```javascript
const test = base.extend({
  fixture1: async ({ }, use) => {
    console.log('Fixture 1 setup');
    await use('data1');
  },
  
  fixture2: async ({ fixture1 }, use) => {
    // Use fixture1 in fixture2
    console.log('Fixture 2 setup (uses fixture1)');
    await use(fixture1 + ' + data2');
  }
});

test('Test', async ({ fixture2 }) => {
  console.log('Got:', fixture2); // 'data1 + data2'
});
```

---

## Example: Testing Different Browsers

```javascript
import { test as base, chromium, firefox, webkit } from '@playwright/test';

const test = base.extend({
  browserSetup: async ({ browserName }, use) => {
    // Launch appropriate browser
    let browser;
    
    if (browserName === 'chromium') {
      browser = await chromium.launch();
    } else if (browserName === 'firefox') {
      browser = await firefox.launch();
    } else {
      browser = await webkit.launch();
    }
    
    console.log('Browser launched:', browserName);
    
    await use(browser);
    
    await browser.close();
    console.log('Browser closed');
  }
});

test('Test on all browsers', async ({ browserSetup, browserName }) => {
  const context = await browserSetup.newContext();
  const page = await context.newPage();
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const title = await page.title();
  console.log(`Page title on ${browserName}:`, title);
});
```

---

---

# COMPLETE EXAMPLES

## Complete Example 1: Professional Test Suite

**fixtures/auth.ts:**
```javascript
import { test as base } from '@playwright/test';

export type TestFixtures = {
  loggedInPage: Page;
};

// Create custom fixture type
export const test = base.extend<TestFixtures>({
  loggedInPage: async ({ page }, use) => {
    // Login before test
    await page.goto('https://testautomationpractice.blogspot.com/');
    
    await page.fill('input[name="username"]', 'admin');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button:has-text("Login")');
    
    await page.waitForLoadState('networkidle');
    
    console.log('✓ Authenticated');
    
    // Provide logged-in page
    await use(page);
    
    // Logout after test
    await page.click('button:has-text("Logout")');
    console.log('✓ Logged out');
  }
});

export { expect } from '@playwright/test';
```

**tests/dashboard.test.ts:**
```javascript
import { test, expect } from '../fixtures/auth';

test('Dashboard is visible', async ({ loggedInPage }) => {
  const page = loggedInPage;
  
  await page.goto('https://testautomationpractice.blogspot.com/dashboard');
  
  const dashboard = await page.locator('.dashboard').isVisible();
  expect(dashboard).toBeTruthy();
});

test('Can view profile', async ({ loggedInPage }) => {
  const page = loggedInPage;
  
  await page.goto('https://testautomationpractice.blogspot.com/profile');
  
  const profile = await page.locator('.profile').isVisible();
  expect(profile).toBeTruthy();
});

test('Can view settings', async ({ loggedInPage }) => {
  const page = loggedInPage;
  
  await page.goto('https://testautomationpractice.blogspot.com/settings');
  
  const settings = await page.locator('.settings').isVisible();
  expect(settings).toBeTruthy();
});
```

---

## Complete Example 2: Multiple Custom Fixtures

**fixtures/index.ts:**
```javascript
import { test as base } from '@playwright/test';

export const test = base.extend({
  // Fixture 1: Test user data
  testUser: async ({ }, use) => {
    const user = {
      username: 'user_' + Date.now(),
      email: 'test_' + Date.now() + '@example.com',
      password: 'Pass123!'
    };
    
    console.log('Test user:', user.username);
    
    await use(user);
    
    // Cleanup if needed
    console.log('Test user cleanup');
  },
  
  // Fixture 2: Logged-in page
  loggedInPage: async ({ page }, use) => {
    await page.goto('https://testautomationpractice.blogspot.com/');
    
    await page.fill('input[name="username"]', 'admin');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button:has-text("Login")');
    
    await page.waitForLoadState('networkidle');
    
    await use(page);
  },
  
  // Fixture 3: API helper
  apiHelper: async ({ }, use) => {
    const BASE_URL = 'https://api.example.com';
    
    const api = {
      async get(endpoint) {
        const response = await fetch(BASE_URL + endpoint);
        return response.json();
      },
      
      async post(endpoint, data) {
        const response = await fetch(BASE_URL + endpoint, {
          method: 'POST',
          body: JSON.stringify(data)
        });
        return response.json();
      }
    };
    
    await use(api);
  }
});

export { expect } from '@playwright/test';
```

**tests/user.test.ts:**
```javascript
import { test, expect } from '../fixtures';

test('User registration', async ({ page, testUser }) => {
  await page.goto('https://testautomationpractice.blogspot.com/register');
  
  await page.fill('input[name="username"]', testUser.username);
  await page.fill('input[name="email"]', testUser.email);
  await page.fill('input[name="password"]', testUser.password);
  
  await page.click('button:has-text("Register")');
  
  await page.waitForSelector('.success-message');
  console.log('✓ User registered');
});

test('Dashboard access', async ({ loggedInPage }) => {
  const page = loggedInPage;
  
  await page.goto('https://testautomationpractice.blogspot.com/dashboard');
  
  const dashboard = await page.locator('.dashboard').isVisible();
  expect(dashboard).toBeTruthy();
});

test('API call', async ({ apiHelper }) => {
  const users = await apiHelper.get('/users');
  console.log('Users:', users);
});
```

---

## Complete Example 3: Advanced Fixture with Options

**fixtures/db.ts:**
```javascript
import { test as base } from '@playwright/test';

type DBOptions = {
  resetDB: boolean;
  seedData: boolean;
};

export const test = base.extend<{}, { db: DBOptions }>({
  db: [
    async ({ }, use) => {
      console.log('Database fixture');
      
      // Setup
      if (process.env.RESET_DB) {
        console.log('Resetting database...');
        await fetch('https://api.example.com/reset', { method: 'POST' });
      }
      
      await use({
        resetDB: true,
        seedData: true
      });
      
      // Cleanup
      console.log('Database cleanup');
    },
    {
      scope: 'test', // Default
      auto: true // Auto-use in all tests
    }
  ]
});

export { expect } from '@playwright/test';
```

---

---

# BEST PRACTICES

### ✅ Do This
```javascript
// Use fixtures for setup/cleanup
const test = base.extend({
  loggedIn: async ({ page }, use) => {
    await page.goto('/login');
    // ... login code ...
    await use(page);
  }
});

// Organize fixtures in separate file
import { test } from './fixtures';

// Use meaningful names
const test = base.extend({
  authenticatedUser: async ({ }, use) => {
    // Clear naming
  }
});

// Document fixture purpose
const test = base.extend({
  // Provides logged-in page after authentication
  loggedIn: async ({ page }, use) => {
    // ...
  }
});
```

### ❌ Don't Do This
```javascript
// Don't put all fixtures in test files
test('My test', async ({ page }) => {
  // Setup code here (should be fixture!)
  const browser = await chromium.launch();
  
  // Test code
});

// Don't make fixtures too complex
const test = base.extend({
  complexFixture: async ({ }, use) => {
    // Doing too many things - split into multiple fixtures
    await setup1();
    await setup2();
    await setup3();
  }
});

// Don't forget cleanup
const test = base.extend({
  fixture: async ({ }, use) => {
    await setup();
    await use();
    // Missing cleanup!
  }
});
```

---

# FIXTURE SCOPES COMPARISON

| Scope | When Runs | Good For | Example |
|-------|-----------|----------|---------|
| **test** | Before/after each test | Isolation, fresh state | Login page |
| **worker** | Before/after worker tests | Expensive setup | Server, DB |
| **session** | Before/after all tests | Shared resources | Global server |

---

# SUMMARY TABLE

| Feature | Built-in | Custom | Use When |
|---------|----------|--------|----------|
| **page** | ✅ | ❌ | Need browser tab |
| **context** | ✅ | ❌ | Need multiple pages |
| **browser** | ✅ | ❌ | Need multiple contexts |
| **loggedIn** | ❌ | ✅ | Tests need auth |
| **testUser** | ❌ | ✅ | Tests need data |
| **API helper** | ❌ | ✅ | Tests need API |

---

# KEY TAKEAWAYS

- ✅ **Fixtures** = Reusable setup/cleanup
- ✅ **Built-in** = page, context, browser, browserName
- ✅ **Custom** = Create your own with `test.extend()`
- ✅ **Setup** = Code before `use()`
- ✅ **Cleanup** = Code after `use()`
- ✅ **DRY** = Don't repeat setup code
- ✅ **Scope** = test, worker, session
- ✅ **Dependencies** = Fixtures can use other fixtures
- ✅ **Reusable** = Import and use in multiple test files

**Test fixtures = Clean, DRY, maintainable tests!** 🎯
