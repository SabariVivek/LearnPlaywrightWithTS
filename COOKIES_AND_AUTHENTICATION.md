# Playwright Cookies & Authentication

## What are Cookies & Authentication?

**Cookies** are small files the browser stores that remember information about you.

**Authentication** is the process of logging in and verifying who you are.

Think of it like: **ID card + memory - Cookie remembers you, Authentication proves it's you**

```
First visit → Login (Authentication) → Browser stores cookie
Next visit → Browser sends cookie → Recognized without login (Persistent Auth)
```

---

## Why Use Cookies & Authentication?

In real tests:
- ❌ **Without**: Every test logs in manually = SLOW
- ✅ **With**: Save login session, reuse it = FAST

Also:
- Save browser state between test runs
- Test different user roles efficiently
- Skip repetitive login steps
- Simulate real user behavior

---

## Three Main Approaches

1. **Manual Login** - Log in during test (simplest)
2. **Store Cookies** - Save cookies, reuse them (faster)
3. **Persistent Login** - Keep session across tests (fastest)

---

---

# APPROACH 1: MANUAL LOGIN

## What is Manual Login?

**Manual Login** = Your test performs login steps each time.

Think of it like: **Going to bank, showing ID, logging in every time**

---

## Example 1: Simple Login

### What Happens

Open page → Fill username → Fill password → Click login → Verify

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Manual login - simple', async ({ page }) => {
  // Navigate to login page
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Fill username
  await page.fill('input[name="username"]', 'admin');
  
  // Fill password
  await page.fill('input[name="password"]', 'password123');
  
  // Click login button
  await page.click('button:has-text("Login")');
  
  // Wait for navigation
  await page.waitForLoadState('networkidle');
  
  // Verify logged in
  const greeting = await page.locator('.user-greeting').textContent();
  console.log('✓ Logged in as:', greeting);
});
```

**Explanation:**
- Simple and direct
- Works for every test
- But repeats login steps

---

## Example 2: Login with Verification

### What Happens

Login → Verify dashboard loads → Verify user name shows

**Playwright Code:**
```javascript
test('Login with verification', async ({ page }) => {
  console.log('--- Starting login test ---');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Fill credentials
  const username = page.locator('input[name="username"]');
  const password = page.locator('input[name="password"]');
  
  await username.fill('testuser');
  await password.fill('SecurePass123!');
  
  console.log('✓ Credentials filled');
  
  // Click login
  const loginBtn = page.locator('button[type="submit"]');
  await loginBtn.click();
  
  // Wait for dashboard
  await page.waitForLoadState('networkidle');
  
  // Verify URL changed
  console.log('✓ URL:', page.url());
  
  // Verify user name displayed
  const userDisplay = await page.locator('.user-name').textContent();
  console.log('✓ User:', userDisplay);
  
  // Verify logout button exists
  const logoutBtn = await page.locator('button:has-text("Logout")').isVisible();
  console.log('✓ Logged in successfully:', logoutBtn);
});
```

---

## Example 3: Reusable Login Function

### What Happens

Create function → Use in multiple tests → Same login logic everywhere

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

// Reusable login function
async function login(page, username, password) {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  await page.fill('input[name="username"]', username);
  await page.fill('input[name="password"]', password);
  await page.click('button:has-text("Login")');
  
  await page.waitForLoadState('networkidle');
}

// Use in tests
test('Test 1 - User can view dashboard', async ({ page }) => {
  await login(page, 'admin', 'password123');
  
  const dashboard = await page.locator('.dashboard').isVisible();
  console.log('✓ Dashboard visible');
});

test('Test 2 - User can view profile', async ({ page }) => {
  await login(page, 'admin', 'password123');
  
  await page.click('a:has-text("Profile")');
  
  const profile = await page.locator('.profile-page').isVisible();
  console.log('✓ Profile visible');
});
```

**Explanation:**
- `login()` function takes page, username, password
- Use in any test
- Same code, less repetition

---

---

# APPROACH 2: MANAGE COOKIES

## What are Cookies?

**Cookies** are data the browser saves to remember you.

When you login, browser stores a cookie with session info. Next time, it sends the cookie automatically.

---

## How Cookies Work

```
Test 1: Login → Browser stores cookie
Test 2: Send cookie → Browser knows who you are (no login needed!)
```

---

## Get Cookies

### After Login

```javascript
// Get all cookies
const cookies = await context.cookies();

// Or get specific cookie
const sessionCookie = await context.cookies();
const loginCookie = sessionCookie.find(c => c.name === 'session_id');
```

---

## Example 1: Get and Store Cookies

### What Happens

Login → Get cookies → Save to file → Use later

**Playwright Code:**
```javascript
import { test } from '@playwright/test';
import * as fs from 'fs';

test('Get cookies after login', async ({ page, context }) => {
  // Go to login page
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Login
  await page.fill('input[name="username"]', 'admin');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button:has-text("Login")');
  
  await page.waitForLoadState('networkidle');
  console.log('✓ Logged in');
  
  // Get cookies
  const cookies = await context.cookies();
  console.log('✓ Cookies obtained:', cookies.length, 'cookies');
  
  // Save to file
  fs.writeFileSync('cookies.json', JSON.stringify(cookies, null, 2));
  console.log('✓ Cookies saved to cookies.json');
  
  // Print cookie details
  cookies.forEach(cookie => {
    console.log(`  Cookie: ${cookie.name} = ${cookie.value}`);
  });
});
```

**Explanation:**
- `context.cookies()` = Get all cookies
- Save as JSON file
- Can load later in other tests

---

## Set Cookies

### Use Stored Cookies

```javascript
// Load cookies from file
const cookies = JSON.parse(fs.readFileSync('cookies.json', 'utf-8'));

// Add cookies to context
await context.addCookies(cookies);
```

---

## Example 2: Load Cookies and Skip Login

### What Happens

Load cookies → Skip login → Navigate to dashboard directly

**Playwright Code:**
```javascript
import { test } from '@playwright/test';
import * as fs from 'fs';

test('Use stored cookies - skip login', async ({ page, context }) => {
  // Check if cookies file exists
  if (fs.existsSync('cookies.json')) {
    console.log('Loading cookies from file...');
    
    // Read cookies
    const cookies = JSON.parse(fs.readFileSync('cookies.json', 'utf-8'));
    
    // Add to context BEFORE navigating
    await context.addCookies(cookies);
    
    console.log('✓ Cookies loaded:', cookies.length, 'cookies');
  }
  
  // Navigate to dashboard (should be logged in due to cookie)
  await page.goto('https://testautomationpractice.blogspot.com/dashboard');
  
  // Verify logged in
  const userGreeting = await page.locator('.user-greeting').textContent();
  console.log('✓ Logged in as:', userGreeting);
  console.log('✓ Skipped login step!');
});
```

**Explanation:**
- Check if cookies file exists
- Load cookies
- Add to context BEFORE navigation
- Now browser knows your session
- Logged in without login page!

---

## Example 3: Check Cookie Details

### What Happens

Login → Get cookies → Display each cookie's details

**Playwright Code:**
```javascript
test('Inspect cookie details', async ({ page, context }) => {
  // Login first
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  await page.fill('input[name="username"]', 'admin');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button:has-text("Login")');
  
  await page.waitForLoadState('networkidle');
  
  // Get cookies
  const cookies = await context.cookies();
  
  console.log('=== Cookie Details ===');
  
  cookies.forEach(cookie => {
    console.log(`\nCookie Name: ${cookie.name}`);
    console.log(`  Value: ${cookie.value.substring(0, 20)}...`);
    console.log(`  Domain: ${cookie.domain}`);
    console.log(`  Path: ${cookie.path}`);
    console.log(`  Expires: ${cookie.expires}`);
    console.log(`  HttpOnly: ${cookie.httpOnly}`);
    console.log(`  Secure: ${cookie.secure}`);
    console.log(`  SameSite: ${cookie.sameSite}`);
  });
});
```

---

## Clear Cookies

### Delete Specific Cookie

```javascript
// Delete one cookie
await context.clearCookies({ name: 'session_id' });

// Delete all cookies
await context.clearCookies();
```

---

## Example 4: Clear Cookies (Logout)

### What Happens

Navigate → Clear cookies → Verify not logged in

**Playwright Code:**
```javascript
test('Clear cookies - logout effect', async ({ page, context }) => {
  // Load cookies (simulating logged-in state)
  if (fs.existsSync('cookies.json')) {
    const cookies = JSON.parse(fs.readFileSync('cookies.json', 'utf-8'));
    await context.addCookies(cookies);
  }
  
  // Go to dashboard
  await page.goto('https://testautomationpractice.blogspot.com/dashboard');
  console.log('✓ Logged in with cookies');
  
  // Clear all cookies
  await context.clearCookies();
  console.log('✓ Cookies cleared');
  
  // Refresh page
  await page.reload();
  
  // Should be redirected to login
  console.log('Current URL:', page.url());
  
  if (page.url().includes('login')) {
    console.log('✓ Logged out successfully');
  }
});
```

---

---

# APPROACH 3: PERSISTENT AUTHENTICATION (ACROSS TESTS)

## What is Persistent Authentication?

**Persistent Auth** = Save login session once, use it in all tests.

Instead of:
- ❌ Every test logs in (slow)

Do:
- ✅ One setup test stores auth, all tests use it (fast)

---

## How It Works

```
Setup Test:
  1. Login
  2. Save cookies to file
  3. Save auth state to file

All Other Tests:
  1. Load auth state
  2. Already logged in
  3. Run test immediately
```

---

## Save Auth State

### After Login

```javascript
// Save auth state to file
await context.storageState({ path: 'auth.json' });
```

---

## Example 1: Save Auth State

### What Happens

Login → Save auth state → Close

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Setup - Save authentication state', async ({ page, context }) => {
  console.log('--- Setting up authentication ---');
  
  // Navigate to login
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Login
  await page.fill('input[name="username"]', 'admin');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button:has-text("Login")');
  
  await page.waitForLoadState('networkidle');
  console.log('✓ Logged in');
  
  // Save auth state with cookies and storage
  await context.storageState({ path: 'auth.json' });
  console.log('✓ Auth state saved to auth.json');
});
```

**What Gets Saved:**
- Cookies
- LocalStorage
- SessionStorage
- Auth tokens

---

## Load Auth State

### In Configuration

```javascript
// In playwright.config.ts
export default {
  use: {
    storageState: 'auth.json', // Auto-load for all tests
  },
};
```

---

## Use In Tests

```javascript
// With storageState in config, tests start logged in!
test('Dashboard is visible', async ({ page }) => {
  // Already logged in!
  await page.goto('https://testautomationpractice.blogspot.com/dashboard');
  
  const dashboard = await page.locator('.dashboard').isVisible();
  console.log('✓ Dashboard visible');
});
```

---

## Example 2: Setup Once, Use Everywhere

### What Happens

Setup test saves auth → Config loads it → All tests use it

**playwright.config.ts:**
```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  use: {
    baseURL: 'https://testautomationpractice.blogspot.com',
    // Auto-load this auth for all tests
    storageState: 'auth.json',
  },
});
```

**tests/auth.setup.ts (Setup Test):**
```javascript
import { test } from '@playwright/test';

test('Setup - Login and save state', async ({ page, context }) => {
  // Navigate and login
  await page.goto('/');
  
  await page.fill('input[name="username"]', 'admin');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button:has-text("Login")');
  
  // Wait for navigation to dashboard
  await page.waitForURL('**/dashboard');
  
  // Save auth state
  await context.storageState({ path: 'auth.json' });
  
  console.log('✓ Auth state saved');
});
```

**tests/dashboard.test.ts (Regular Test):**
```javascript
import { test, expect } from '@playwright/test';

test('View dashboard', async ({ page }) => {
  // Already logged in! auth.json auto-loaded by config
  
  await page.goto('/dashboard');
  
  // Just use the page, no login needed
  const heading = await page.locator('h1').textContent();
  console.log('✓ Dashboard heading:', heading);
});

test('View profile', async ({ page }) => {
  // Also already logged in
  
  await page.goto('/profile');
  
  const profile = await page.locator('.profile').isVisible();
  expect(profile).toBeTruthy();
});
```

**Explanation:**
- Setup test runs first, saves auth
- Config auto-loads auth.json for all tests
- All tests start already logged in
- Much faster!

---

## Example 3: Multiple Auth States (Multiple Users)

### What Happens

Save auth for admin → Save auth for customer → Use different ones

**playwright.config.ts:**
```typescript
export default defineConfig({
  testDir: './tests',
  use: {
    baseURL: 'https://testautomationpractice.blogspot.com',
    storageState: 'auth.admin.json', // Default
  },
});
```

**tests/auth.setup.ts (Setup Tests):**
```javascript
import { test } from '@playwright/test';

test('Setup - Admin login', async ({ page, context }) => {
  await page.goto('/');
  
  // Admin login
  await page.fill('input[name="username"]', 'admin');
  await page.fill('input[name="password"]', 'admin123');
  await page.click('button:has-text("Login")');
  
  await page.waitForURL('**/admin-dashboard');
  
  // Save admin auth
  await context.storageState({ path: 'auth.admin.json' });
  console.log('✓ Admin auth saved');
});

test('Setup - Customer login', async ({ page, context }) => {
  await page.goto('/');
  
  // Customer login
  await page.fill('input[name="username"]', 'customer1');
  await page.fill('input[name="password"]', 'customer123');
  await page.click('button:has-text("Login")');
  
  await page.waitForURL('**/dashboard');
  
  // Save customer auth
  await context.storageState({ path: 'auth.customer.json' });
  console.log('✓ Customer auth saved');
});
```

**tests/admin.test.ts (Use Admin Auth):**
```javascript
import { test } from '@playwright/test';

test.use({ storageState: 'auth.admin.json' });

test('Admin - View all users', async ({ page }) => {
  // Uses admin auth
  await page.goto('/admin/users');
  
  const userList = await page.locator('.user-list').isVisible();
  console.log('✓ Admin can view users');
});
```

**tests/customer.test.ts (Use Customer Auth):**
```javascript
import { test } from '@playwright/test';

test.use({ storageState: 'auth.customer.json' });

test('Customer - View my orders', async ({ page }) => {
  // Uses customer auth
  await page.goto('/my-orders');
  
  const orders = await page.locator('.order-list').isVisible();
  console.log('✓ Customer can view orders');
});
```

**Explanation:**
- Save different auth states for different users
- Use `test.use()` to apply specific auth
- Different tests use different logins
- Perfect for role-based testing!

---

---

# COMPLETE EXAMPLES

## Complete Example 1: Full Auth Workflow

```javascript
import { test } from '@playwright/test';
import * as fs from 'fs';

test('Complete authentication workflow', async ({ page, context }) => {
  console.log('=== Authentication Workflow ===\n');
  
  // Step 1: Navigate to login
  console.log('Step 1: Navigate to login page');
  await page.goto('https://testautomationpractice.blogspot.com/');
  console.log('✓ Login page loaded\n');
  
  // Step 2: Enter credentials
  console.log('Step 2: Enter credentials');
  await page.fill('input[name="username"]', 'admin');
  await page.fill('input[name="password"]', 'password123');
  console.log('✓ Credentials entered\n');
  
  // Step 3: Click login
  console.log('Step 3: Submit login form');
  await page.click('button:has-text("Login")');
  await page.waitForLoadState('networkidle');
  console.log('✓ Login submitted\n');
  
  // Step 4: Verify login success
  console.log('Step 4: Verify login success');
  const userGreeting = await page.locator('.user-greeting').textContent();
  console.log('✓ Logged in as:', userGreeting, '\n');
  
  // Step 5: Get cookies
  console.log('Step 5: Get and save cookies');
  const cookies = await context.cookies();
  fs.writeFileSync('cookies.json', JSON.stringify(cookies, null, 2));
  console.log('✓ Saved', cookies.length, 'cookies\n');
  
  // Step 6: Save auth state
  console.log('Step 6: Save auth state');
  await context.storageState({ path: 'auth.json' });
  console.log('✓ Auth state saved\n');
  
  // Step 7: Show session info
  console.log('Step 7: Session Information');
  console.log('  URL:', page.url());
  console.log('  Title:', await page.title());
  console.log('\n✓ Complete workflow finished');
});
```

---

## Complete Example 2: Setup + Regular Tests

**tests/setup.ts:**
```javascript
import { test } from '@playwright/test';

test('Setup - Authenticate user', async ({ page, context }) => {
  console.log('--- Running Setup ---');
  
  // Login
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  await page.fill('input[name="username"]', 'admin');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button:has-text("Login")');
  
  await page.waitForLoadState('networkidle');
  
  // Save auth
  await context.storageState({ path: 'auth.json' });
  
  console.log('✓ Authentication setup complete\n');
});
```

**tests/user-tests.test.ts:**
```javascript
import { test, expect } from '@playwright/test';

// Use saved auth for all tests in this file
test.use({ storageState: 'auth.json' });

test('User can view dashboard', async ({ page }) => {
  console.log('\nTest 1: Dashboard');
  
  await page.goto('https://testautomationpractice.blogspot.com/dashboard');
  
  const dashboard = await page.locator('.dashboard').isVisible();
  expect(dashboard).toBeTruthy();
  
  console.log('✓ Dashboard visible');
});

test('User can view profile', async ({ page }) => {
  console.log('\nTest 2: Profile');
  
  await page.goto('https://testautomationpractice.blogspot.com/profile');
  
  const profile = await page.locator('.profile-info').isVisible();
  expect(profile).toBeTruthy();
  
  console.log('✓ Profile visible');
});

test('User can view settings', async ({ page }) => {
  console.log('\nTest 3: Settings');
  
  await page.goto('https://testautomationpractice.blogspot.com/settings');
  
  const settings = await page.locator('.settings-page').isVisible();
  expect(settings).toBeTruthy();
  
  console.log('✓ Settings visible');
});
```

**playwright.config.ts:**
```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  
  // Setup file runs first
  webServer: {
    command: 'npm run start',
    port: 3000,
  },
  
  use: {
    baseURL: 'https://testautomationpractice.blogspot.com',
    storageState: 'auth.json',
  },
});
```

---

## Complete Example 3: Multiple Users

**tests/setup.ts:**
```javascript
import { test } from '@playwright/test';

test('Setup - Admin user', async ({ page, context }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  await page.fill('input[name="username"]', 'admin');
  await page.fill('input[name="password"]', 'admin123');
  await page.click('button:has-text("Login")');
  
  await page.waitForLoadState('networkidle');
  
  await context.storageState({ path: 'auth.admin.json' });
  console.log('✓ Admin authenticated');
});

test('Setup - Regular user', async ({ page, context }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  await page.fill('input[name="username"]', 'user1');
  await page.fill('input[name="password"]', 'user123');
  await page.click('button:has-text("Login")');
  
  await page.waitForLoadState('networkidle');
  
  await context.storageState({ path: 'auth.user.json' });
  console.log('✓ User authenticated');
});
```

**tests/admin-features.test.ts:**
```javascript
import { test, expect } from '@playwright/test';

test.use({ storageState: 'auth.admin.json' });

test('Admin can manage users', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/admin');
  
  const manageUsers = await page.locator('button:has-text("Manage Users")').isVisible();
  expect(manageUsers).toBeTruthy();
});
```

**tests/user-features.test.ts:**
```javascript
import { test, expect } from '@playwright/test';

test.use({ storageState: 'auth.user.json' });

test('User can view profile', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/profile');
  
  const profile = await page.locator('.profile').isVisible();
  expect(profile).toBeTruthy();
});
```

---

---

# COOKIE OPERATIONS

## Get Specific Cookie

```javascript
const cookies = await context.cookies();
const sessionCookie = cookies.find(c => c.name === 'session_id');

console.log('Session cookie:', sessionCookie.value);
```

---

## Add Multiple Cookies

```javascript
const newCookies = [
  {
    name: 'user_id',
    value: '12345',
    domain: 'example.com',
    path: '/',
  },
  {
    name: 'theme',
    value: 'dark',
    domain: 'example.com',
    path: '/',
  },
];

await context.addCookies(newCookies);
```

---

## Delete Specific Cookie

```javascript
// Delete one cookie
await context.clearCookies({ name: 'session_id' });

// Delete cookies for domain
await context.clearCookies({ domain: 'example.com' });
```

---

---

# BEST PRACTICES

### ✅ Do This
```javascript
// Store auth after login
await context.storageState({ path: 'auth.json' });

// Use config for persistent auth
test.use({ storageState: 'auth.json' });

// Add cookies BEFORE navigation
await context.addCookies(cookies);
await page.goto('/dashboard');

// Save different auth for different users
// auth.admin.json, auth.user.json

// Use setup test
test('Setup', async ({ context }) => {
  // Login and save
});
```

### ❌ Don't Do This
```javascript
// Don't include cookies in version control
// git ignore: auth.json, cookies.json

// Don't share sensitive tokens publicly
// Never commit cookies with real sessions

// Don't add cookies AFTER navigation
await page.goto('/dashboard');
await context.addCookies(cookies); // Too late!

// Don't hardcode passwords in tests
// Use environment variables instead
```

---

# ENVIRONMENT VARIABLES

### Store Credentials Securely

**tests/auth.setup.ts:**
```javascript
test('Setup', async ({ page, context }) => {
  const username = process.env.TEST_USERNAME;
  const password = process.env.TEST_PASSWORD;
  
  await page.goto('/');
  
  await page.fill('input[name="username"]', username);
  await page.fill('input[name="password"]', password);
  
  await page.click('button:has-text("Login")');
});
```

**.env.local (NOT in git):**
```
TEST_USERNAME=admin
TEST_PASSWORD=secretpassword123
```

**Run with env:**
```bash
npx playwright test
```

---

# SUMMARY TABLE

| Scenario | Use | Method |
|----------|-----|--------|
| **Simple login** | Each test | `await page.fill()` + `click()` |
| **Reuse in test** | Multiple steps | Get cookies, use later |
| **Skip login** | Many tests | Store auth state, load in config |
| **Multiple users** | Role-based | Different auth.user.json files |
| **Session mgmt** | Across runs | `storageState()` |
| **Clear session** | Logout | `context.clearCookies()` |

---

# KEY TAKEAWAYS

- ✅ **Manual Login**: Simple, works always
- ✅ **Cookies**: Get + store for reuse
- ✅ **Auth State**: Save after login, auto-load in tests
- ✅ **Persistent Auth**: Fastest approach for multiple tests
- ✅ **Multiple Users**: Save separate auth for each
- ✅ **Environment Variables**: Store credentials securely
- ✅ **Setup Tests**: Run once, save auth, reuse everywhere
- ✅ **Clear Cookies**: Clear for logout/reset

**Proper auth management = Fast, reliable, realistic tests!** 🎯
