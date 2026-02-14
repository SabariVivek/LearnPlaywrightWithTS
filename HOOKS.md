# Playwright Hooks - Test Lifecycle Management

## What are Hooks?

**Hooks** are functions that run at specific times in test execution.

Think of it like: **Traffic controller directing when traffic flows**

```
beforeAll → [beforeEach → Test 1 → afterEach] → [beforeEach → Test 2 → afterEach] → afterAll
```

---

## Why Use Hooks?

- ✅ **Setup** - Prepare before tests
- ✅ **Cleanup** - Clean after tests
- ✅ **State Management** - Control when things happen
- ✅ **Error Handling** - Catch and respond to failures
- ✅ **Screenshots** - Capture on failure

---

## Four Types of Hooks

| Hook | Runs | Use For |
|------|------|---------|
| **beforeAll** | Once before all tests | Start server, create DB |
| **beforeEach** | Before each test | Login, reset state |
| **afterEach** | After each test | Cleanup, take screenshot |
| **afterAll** | Once after all tests | Stop server, close DB |

---

---

# BEFOREALL - Run Once Before All Tests

## What Does It Do?

**beforeAll** = Runs one time, before first test starts.

Think of it like: **Opening ceremony before games start**

---

## When and Why Use It

Use for expensive operations that should happen only once:
- Start server
- Connect to database
- Load large files
- Create shared test data

---

## Example 1: Start Server

**What Happens:**
Start server before any tests → All tests run → Server still running

```javascript
import { test } from '@playwright/test';

test.beforeAll(async () => {
  console.log('=== Starting beforeAll hook ===');
  
  console.log('Starting test server...');
  
  // Simulating server startup
  await new Promise(resolve => setTimeout(resolve, 1000));
  
  console.log('✓ Server started on http://localhost:3000');
  console.log('=== beforeAll complete ===\n');
});

test('Test 1', async ({ page }) => {
  console.log('Running Test 1');
  await page.goto('http://localhost:3000');
});

test('Test 2', async ({ page }) => {
  console.log('Running Test 2');
  await page.goto('http://localhost:3000');
});

test('Test 3', async ({ page }) => {
  console.log('Running Test 3');
  await page.goto('http://localhost:3000');
});

// Output:
// === Starting beforeAll hook ===
// Starting test server...
// ✓ Server started on http://localhost:3000
// === beforeAll complete ===
//
// Running Test 1
// Running Test 2
// Running Test 3
```

---

## Example 2: Connect to Database

```javascript
import { test } from '@playwright/test';

let database;

test.beforeAll(async () => {
  console.log('Connecting to database...');
  
  // Simulate DB connection
  database = {
    connection: 'connected',
    query: async (sql) => {
      console.log('Executing:', sql);
      return [];
    }
  };
  
  console.log('✓ Database connected');
});

test('Test 1 - Query users', async () => {
  const users = await database.query('SELECT * FROM users');
  console.log('Users:', users);
});

test('Test 2 - Query products', async () => {
  const products = await database.query('SELECT * FROM products');
  console.log('Products:', products);
});
```

---

## Example 3: Load Test Data Once

```javascript
import { test } from '@playwright/test';

let testData;

test.beforeAll(async () => {
  console.log('Loading test data (large file)...');
  
  // Simulate loading large dataset
  await new Promise(resolve => setTimeout(resolve, 2000));
  
  testData = {
    users: ['user1', 'user2', 'user3'],
    products: ['product1', 'product2'],
    orders: ['order1', 'order2', 'order3']
  };
  
  console.log('✓ Test data loaded');
});

test('Test 1 - Check users', async () => {
  console.log('Users available:', testData.users.length);
});

test('Test 2 - Check products', async () => {
  console.log('Products available:', testData.products.length);
});

test('Test 3 - Check orders', async () => {
  console.log('Orders available:', testData.orders.length);
});
```

---

---

# BEFOREEACH - Run Before Each Test

## What Does It Do?

**beforeEach** = Runs before every single test.

Think of it like: **Athlete stretching before each match**

---

## When and Why Use It

Use for per-test setup:
- Login for each test
- Clear state
- Reset page
- Prepare fresh data

---

## Example 1: Login Before Each Test

**What Happens:**
Test 1 runs → Login → Test 1 does stuff → End
Test 2 runs → Login (again!) → Test 2 does stuff → End

```javascript
import { test } from '@playwright/test';

test.beforeEach(async ({ page }) => {
  console.log('\n--- beforeEach: Logging in ---');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  await page.fill('input[name="username"]', 'admin');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button:has-text("Login")');
  
  await page.waitForLoadState('networkidle');
  
  console.log('✓ Logged in\n');
});

test('Test 1 - View dashboard', async ({ page }) => {
  console.log('Test 1: Dashboard');
  
  await page.goto('https://testautomationpractice.blogspot.com/dashboard');
  
  const dashboard = await page.locator('.dashboard').isVisible();
  console.log('Dashboard visible:', dashboard);
});

test('Test 2 - View profile', async ({ page }) => {
  console.log('Test 2: Profile');
  
  await page.goto('https://testautomationpractice.blogspot.com/profile');
  
  const profile = await page.locator('.profile').isVisible();
  console.log('Profile visible:', profile);
});

// Output:
// --- beforeEach: Logging in ---
// ✓ Logged in
//
// Test 1: Dashboard
// Dashboard visible: true
//
// --- beforeEach: Logging in ---
// ✓ Logged in
//
// Test 2: Profile
// Profile visible: true
```

---

## Example 2: Reset Page State

```javascript
import { test } from '@playwright/test';

test.beforeEach(async ({ page }) => {
  console.log('beforeEach: Clearing page state');
  
  // Clear all data
  await page.evaluate(() => {
    localStorage.clear();
    sessionStorage.clear();
  });
  
  console.log('✓ Page state cleared');
});

test('Test 1 - Fresh state', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const stored = await page.evaluate(() => localStorage.length);
  console.log('LocalStorage items:', stored); // Should be 0
});

test('Test 2 - Also fresh state', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const stored = await page.evaluate(() => localStorage.length);
  console.log('LocalStorage items:', stored); // Should be 0
});
```

---

## Example 3: Navigate to Home Page

```javascript
import { test } from '@playwright/test';

test.beforeEach(async ({ page }) => {
  console.log('beforeEach: Navigating to home');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  console.log('✓ Home page loaded');
});

test('Test 1 - From home page', async ({ page }) => {
  console.log('Test 1 starts at home');
  
  const title = await page.title();
  console.log('Page title:', title);
});

test('Test 2 - Also from home page', async ({ page }) => {
  console.log('Test 2 starts at home');
  
  const url = page.url();
  console.log('Current URL:', url);
});
```

---

---

# AFTEREACH - Run After Each Test

## What Does It Do?

**afterEach** = Runs after every single test.

Think of it like: **Cleaning equipment after each use**

---

## When and Why Use It

Use for per-test cleanup:
- Take screenshots on failure
- Logout
- Clear data
- Capture logs
- Close resources

---

## Example 1: Screenshot on Failure

**What Happens:**
Test fails → afterEach runs → Takes screenshot

```javascript
import { test } from '@playwright/test';
import * as fs from 'fs';

test.afterEach(async ({ page }, testInfo) => {
  if (testInfo.status !== 'passed') {
    console.log(`\nafterEach: Taking screenshot (status: ${testInfo.status})`);
    
    const screenshot = await page.screenshot({
      path: `screenshots/${testInfo.title}-${Date.now()}.png`
    });
    
    console.log('✓ Screenshot saved');
  }
});

test('Test 1 - Will pass', async ({ page }) => {
  console.log('Test 1 running');
  console.log('✓ Test 1 passed');
});

test('Test 2 - Will fail', async ({ page }) => {
  console.log('Test 2 running');
  
  const element = await page.locator('.nonexistent').isVisible();
  
  if (!element) {
    console.log('✗ Test 2 failed');
    throw new Error('Element not found');
  }
});

// Output:
// Test 1 running
// ✓ Test 1 passed
//
// Test 2 running
// ✗ Test 2 failed
//
// afterEach: Taking screenshot (status: failed)
// ✓ Screenshot saved
```

---

## Example 2: Logout After Each Test

```javascript
import { test } from '@playwright/test';

test.beforeEach(async ({ page }) => {
  console.log('beforeEach: Logging in');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  await page.fill('input[name="username"]', 'admin');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button:has-text("Login")');
});

test.afterEach(async ({ page }) => {
  console.log('afterEach: Logging out');
  
  try {
    await page.click('button:has-text("Logout")');
    console.log('✓ Logged out');
  } catch (error) {
    console.log('Logout button not found');
  }
});

test('Test 1', async ({ page }) => {
  console.log('Test 1: User is logged in');
});

test('Test 2', async ({ page }) => {
  console.log('Test 2: User is logged in');
});

// Output:
// beforeEach: Logging in
// Test 1: User is logged in
// afterEach: Logging out
// ✓ Logged out
//
// beforeEach: Logging in
// Test 2: User is logged in
// afterEach: Logging out
// ✓ Logged out
```

---

## Example 3: Capture Console Logs

```javascript
import { test } from '@playwright/test';

let consoleLogs = [];

test.beforeEach(async ({ page }) => {
  console.log('beforeEach: Setting up console capture');
  
  consoleLogs = [];
  
  // Capture all console messages
  page.on('console', msg => {
    consoleLogs.push(`[${msg.type()}] ${msg.text()}`);
  });
});

test.afterEach(async ({ page }, testInfo) => {
  console.log(`\nafterEach: Console logs for "${testInfo.title}":`);
  
  consoleLogs.forEach(log => {
    console.log('  ' + log);
  });
});

test('Test 1', async ({ page }) => {
  await page.evaluate(() => {
    console.log('App started');
    console.log('Loading data...');
    console.log('Data loaded');
  });
});

test('Test 2', async ({ page }) => {
  await page.evaluate(() => {
    console.warn('Warning: Performance slow');
    console.error('Error: Failed to load');
  });
});
```

---

---

# AFTERALL - Run Once After All Tests

## What Does It Do?

**afterAll** = Runs one time, after last test finishes.

Think of it like: **Closing ceremony after all games end**

---

## When and Why Use It

Use for global cleanup:
- Stop server
- Close database
- Clean up files
- Generate report

---

## Example 1: Stop Server

```javascript
import { test } from '@playwright/test';

let server;

test.beforeAll(async () => {
  console.log('beforeAll: Starting server');
  
  // Simulate server start
  server = { running: true };
  
  console.log('✓ Server started');
});

test.afterAll(async () => {
  console.log('\nafterAll: Stopping server');
  
  server.running = false;
  
  console.log('✓ Server stopped');
});

test('Test 1', async ({ page }) => {
  console.log('Test 1: Server is running');
});

test('Test 2', async ({ page }) => {
  console.log('Test 2: Server is running');
});

// Output:
// beforeAll: Starting server
// ✓ Server started
//
// Test 1: Server is running
// Test 2: Server is running
//
// afterAll: Stopping server
// ✓ Server stopped
```

---

## Example 2: Close Database Connection

```javascript
import { test } from '@playwright/test';

let database;

test.beforeAll(async () => {
  console.log('beforeAll: Connecting to database');
  
  database = {
    connected: true,
    query: async (sql) => []
  };
  
  console.log('✓ Database connected');
});

test.afterAll(async () => {
  console.log('\nafterAll: Closing database connection');
  
  database.connected = false;
  
  console.log('✓ Database closed');
});

test('Test 1 - Query data', async () => {
  console.log('Test 1: Using database');
  
  await database.query('SELECT * FROM users');
});

test('Test 2 - Query more data', async () => {
  console.log('Test 2: Using database');
  
  await database.query('SELECT * FROM products');
});
```

---

## Example 3: Generate Report

```javascript
import { test } from '@playwright/test';

let testResults = [];

test.beforeEach(async ({ }, testInfo) => {
  testResults.push({
    title: testInfo.title,
    startTime: Date.now()
  });
});

test.afterEach(async ({ }, testInfo) => {
  const result = testResults.find(r => r.title === testInfo.title);
  result.duration = Date.now() - result.startTime;
  result.status = testInfo.status;
});

test.afterAll(() => {
  console.log('\nafterAll: Generating report');
  console.log('\n=== Test Results ===');
  
  testResults.forEach(result => {
    console.log(
      `${result.title}: ${result.status} (${result.duration}ms)`
    );
  });
  
  const passed = testResults.filter(r => r.status === 'passed').length;
  const failed = testResults.filter(r => r.status === 'failed').length;
  
  console.log(`\nSummary: ${passed} passed, ${failed} failed`);
});

test('Test 1', async () => {
  console.log('Test 1 running');
});

test('Test 2', async () => {
  console.log('Test 2 running');
});
```

---

---

# HOOK SCOPE

## Scope of Hooks

Hooks can be applied in different scopes:

```javascript
// For single test file
test.beforeEach(...);

// For specific test
test('My test', async () => {});

// For test describe group
test.describe('My group', () => {
  test.beforeEach(...);
  
  test('Test in group', async () => {});
});
```

---

## Example: Hooks in Describe Group

```javascript
import { test, expect } from '@playwright/test';

test.describe('Admin Tests', () => {
  test.beforeEach(async ({ page }) => {
    console.log('Admin beforeEach: Logging in as admin');
    
    await page.goto('https://testautomationpractice.blogspot.com/');
    await page.fill('input[name="username"]', 'admin');
    await page.fill('input[name="password"]', 'admin123');
    await page.click('button:has-text("Login")');
  });

  test('Admin can see admin panel', async ({ page }) => {
    await page.goto('/admin');
    const panel = await page.locator('.admin-panel').isVisible();
    expect(panel).toBeTruthy();
  });

  test('Admin can manage users', async ({ page }) => {
    await page.goto('/admin/users');
    const userList = await page.locator('.user-list').isVisible();
    expect(userList).toBeTruthy();
  });
});

test.describe('User Tests', () => {
  test.beforeEach(async ({ page }) => {
    console.log('User beforeEach: Logging in as regular user');
    
    await page.goto('https://testautomationpractice.blogspot.com/');
    await page.fill('input[name="username"]', 'user1');
    await page.fill('input[name="password"]', 'user123');
    await page.click('button:has-text("Login")');
  });

  test('User can see dashboard', async ({ page }) => {
    await page.goto('/dashboard');
    const dashboard = await page.locator('.dashboard').isVisible();
    expect(dashboard).toBeTruthy();
  });

  test('User cannot see admin panel', async ({ page }) => {
    try {
      await page.goto('/admin');
      const blocked = await page.locator('.access-denied').isVisible();
      expect(blocked).toBeTruthy();
    } catch {
      console.log('Access denied as expected');
    }
  });
});
```

---

---

# COMPLETE EXAMPLES

## Complete Example 1: Full Test Lifecycle

```javascript
import { test, expect } from '@playwright/test';

let server;
let database;

test.beforeAll(async () => {
  console.log('\n========== BEFOREALL ==========');
  console.log('Starting server...');
  
  server = { running: true };
  await new Promise(resolve => setTimeout(resolve, 500));
  
  console.log('Connecting to database...');
  database = { connected: true };
  
  console.log('✓ Setup complete\n');
});

test.beforeEach(async ({ page }) => {
  console.log('--- beforeEach: Login ---');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  await page.fill('input[name="username"]', 'admin');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button:has-text("Login")');
});

test('Test 1 - Dashboard', async ({ page }) => {
  console.log('Test 1 running');
  
  await page.goto('https://testautomationpractice.blogspot.com/dashboard');
  
  const dashboard = await page.locator('.dashboard');
  expect(dashboard).toBeDefined();
  
  console.log('✓ Test 1 passed');
});

test('Test 2 - Profile', async ({ page }) => {
  console.log('Test 2 running');
  
  await page.goto('https://testautomationpractice.blogspot.com/profile');
  
  const profile = await page.locator('.profile');
  expect(profile).toBeDefined();
  
  console.log('✓ Test 2 passed');
});

test.afterEach(async ({ page }, testInfo) => {
  console.log(`--- afterEach: Cleanup ---`);
  
  try {
    await page.click('button:has-text("Logout")');
    console.log('✓ Logged out\n');
  } catch {
    console.log('Logout button not found\n');
  }
});

test.afterAll(async () => {
  console.log('========== AFTERALL ==========');
  console.log('Stopping server...');
  
  server.running = false;
  
  console.log('Closing database...');
  database.connected = false;
  
  console.log('✓ Teardown complete');
});
```

---

## Complete Example 2: Error Handling in Hooks

```javascript
import { test, expect } from '@playwright/test';

test.beforeEach(async ({ page }, testInfo) => {
  console.log(`\nbeforeEach: Setting up "${testInfo.title}"`);
  
  try {
    await page.goto('https://testautomationpractice.blogspot.com/');
    console.log('✓ Page loaded');
  } catch (error) {
    console.log('✗ Failed to load page:', error.message);
    throw error;
  }
});

test.afterEach(async ({ page }, testInfo) => {
  console.log(`afterEach: Cleaning up "${testInfo.title}"`);
  
  try {
    // Capture screenshot if failed
    if (testInfo.status !== 'passed') {
      console.log('Taking screenshot due to failure');
      await page.screenshot({ path: `failure-${Date.now()}.png` });
    }
    
    // Clear cookies
    await page.context().clearCookies();
    console.log('✓ Cookies cleared');
    
  } catch (error) {
    console.log('Error during cleanup:', error.message);
  }
});

test('Test 1', async ({ page }) => {
  console.log('Executing Test 1');
  expect(true).toBeTruthy();
});

test('Test 2', async ({ page }) => {
  console.log('Executing Test 2');
  expect(true).toBeTruthy();
});
```

---

## Complete Example 3: Data Management with Hooks

```javascript
import { test } from '@playwright/test';

let testContext = {
  users: [],
  createdItems: []
};

test.beforeAll(async () => {
  console.log('beforeAll: Loading seed data');
  
  testContext.users = [
    { id: 1, name: 'Admin', role: 'admin' },
    { id: 2, name: 'User1', role: 'user' }
  ];
  
  console.log('✓ Seed data loaded');
});

test.beforeEach(async ({ page }, testInfo) => {
  console.log(`beforeEach: Reset for "${testInfo.title}"`);
  
  testContext.createdItems = [];
  
  console.log('✓ Test context reset');
});

test('Test 1 - Create item', async ({ page }) => {
  console.log('Creating item...');
  
  const item = { id: 1, name: 'Item 1' };
  testContext.createdItems.push(item);
  
  console.log('Items created:', testContext.createdItems.length);
});

test('Test 2 - Create multiple items', async ({ page }) => {
  console.log('Creating multiple items...');
  
  testContext.createdItems.push({ id: 1, name: 'Item A' });
  testContext.createdItems.push({ id: 2, name: 'Item B' });
  
  console.log('Items created:', testContext.createdItems.length);
});

test.afterEach(async ({ }, testInfo) => {
  console.log(`afterEach: Cleanup for "${testInfo.title}"`);
  console.log('Items to cleanup:', testContext.createdItems.length);
  
  testContext.createdItems = [];
  
  console.log('✓ Cleanup complete\n');
});

test.afterAll(async () => {
  console.log('afterAll: Final cleanup');
  console.log('Test users:', testContext.users.length);
});
```

---

---

# HOOKS VS FIXTURES

## Key Differences

| Feature | Hooks | Fixtures |
|---------|-------|----------|
| **Purpose** | Setup/cleanup timing | Provide reusable data/state |
| **Returns data?** | No | Yes |
| **Reusable** | In same file | Across files |
| **Scope control** | Limited | Better |
| **Use for** | Login, cleanup | Page, user, API |

---

## When to Use What

```javascript
// Use HOOKS for:
// - Login/logout (timing matters)
// - Database/server start/stop
// - Global setup/cleanup
// - Screenshots on failure

test.beforeEach(async ({ page }) => {
  await login(page);
});

// Use FIXTURES for:
// - Reusable data (test user, page)
// - Complex setup logic
// - Multiple test files
// - Providing objects to tests

const test = base.extend({
  loggedInPage: async ({ page }, use) => {
    await login(page);
    await use(page);
  }
});
```

---

---

# BEST PRACTICES

### ✅ Do This
```javascript
// Use beforeEach for per-test setup
test.beforeEach(async ({ page }) => {
  await page.goto('/');
});

// Use afterEach for cleanup
test.afterEach(async ({ page }) => {
  await page.context().clearCookies();
});

// Use error handling in hooks
test.afterEach(async ({ page }) => {
  try {
    await cleanup();
  } catch (error) {
    console.log('Cleanup error:', error);
  }
});

// Use describe groups for scoped hooks
test.describe('Admin', () => {
  test.beforeEach(async ({ page }) => {
    await loginAsAdmin(page);
  });
});
```

### ❌ Don't Do This
```javascript
// Don't put test logic in hooks
test.beforeEach(async ({ page }) => {
  // DON'T: This is test logic
  await page.click('button');
  const result = await page.locator('.result').text();
  expect(result).toBe('...'); // Wrong!
});

// Don't use same hook multiple times
test.beforeEach(async () => {}); // First one
test.beforeEach(async () => {}); // Overwrites!

// Don't put expensive operations in beforeEach
test.beforeEach(async () => {
  // Slow! Runs before EVERY test
  await slowDatabaseQuery();
});

// Don't forget error handling
test.afterEach(async ({ page }) => {
  // If this throws, cleanup might not finish
  await page.click('button');
});
```

---

# HOOK EXECUTION ORDER

## Single Test

```
beforeAll (once)
  ↓
beforeEach (test 1)
  ↓
Test 1
  ↓
afterEach (test 1)
  ↓
beforeEach (test 2)
  ↓
Test 2
  ↓
afterEach (test 2)
  ↓
afterAll (once)
```

---

# SUMMARY TABLE

| Hook | Frequency | Use For | Example |
|------|-----------|---------|---------|
| **beforeAll** | Once | Start server, connect DB | Launch server |
| **beforeEach** | Before each test | Login, reset state | Clear localStorage |
| **afterEach** | After each test | Screenshot, logout | Take failure screenshot |
| **afterAll** | Once | Stop server, close DB | Stop server |

---

# KEY TAKEAWAYS

- ✅ **beforeAll** = Once before all tests
- ✅ **beforeEach** = Before each test
- ✅ **afterEach** = After each test
- ✅ **afterAll** = Once after all tests
- ✅ **Timing** = Important for lifecycle
- ✅ **Error Handling** = Always wrap in try-catch
- ✅ **Describe Groups** = Scope hooks to groups
- ✅ **Cleanup** = Always cleanup in afterEach/afterAll
- ✅ **Performance** = Use beforeAll for expensive operations

**Proper hooks = Clean test lifecycle and robust tests!** 🎯
