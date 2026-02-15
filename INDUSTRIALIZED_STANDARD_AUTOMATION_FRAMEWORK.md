# 🏭 Industrialized Standard Automation Framework

Building an enterprise automation framework requires careful planning and adherence to best practices. This guide consolidates all critical points to remember when creating a professional, scalable testing framework.

---

## 1. ARCHITECTURE & SCALABILITY

### Modular Design
```typescript
// ✅ DO: Separate concerns clearly
src/
├── pages/                 // Page Objects
├── tests/                 // Test files
├── fixtures/              // Custom fixtures
├── utils/                 // Helper functions
├── config/                // Configuration
└── constants/             // Constants, test data
```

```typescript
// ✅ DO: Create reusable base classes
export class BasePage {
  protected page: Page;
  protected timeout = 30000;

  async click(selector: string) {
    await this.page.locator(selector).click();
  }

  async fill(selector: string, text: string) {
    await this.page.locator(selector).fill(text);
  }

  async waitForElement(selector: string) {
    await this.page.locator(selector).waitFor({ state: 'visible' });
  }
}
```

### Configuration Management

```typescript
// ✅ DO: Environment-specific configuration
// config/config.ts
export const config = {
  development: {
    baseURL: 'http://localhost:3000',
    timeout: 30000,
    retries: 2,
  },
  staging: {
    baseURL: 'https://staging.example.com',
    timeout: 45000,
    retries: 3,
  },
  production: {
    baseURL: 'https://example.com',
    timeout: 60000,
    retries: 1,
  },
};

const env = process.env.ENV || 'development';
export const activeConfig = config[env as keyof typeof config];
```

---

## 2. PAGE OBJECT MODEL (POM) - MANDATORY

### Strict POM Implementation

```typescript
// ✅ DO: Keep locators and methods in page object
export class LoginPage extends BasePage {
  // Page-specific locators
  readonly emailInput = 'input[id="email"]';
  readonly passwordInput = 'input[id="password"]';
  readonly loginButton = 'button:has-text("Sign In")';
  readonly errorMessage = '.error-message';
  readonly forgotPasswordLink = 'a:has-text("Forgot Password")';

  // Page-specific methods
  async fillEmail(email: string) {
    await this.fill(this.emailInput, email);
  }

  async fillPassword(password: string) {
    await this.fill(this.passwordInput, password);
  }

  async clickLoginButton() {
    await this.click(this.loginButton);
  }

  async loginAsUser(email: string, password: string) {
    // Compound action - combines multiple steps
    await this.fillEmail(email);
    await this.fillPassword(password);
    await this.clickLoginButton();
  }

  async getErrorMessage(): Promise<string> {
    return await this.page.locator(this.errorMessage).textContent() || '';
  }
}
```

```typescript
// ❌ DON'T: Expose implementation details in tests
test('login', async ({ page }) => {
  await page.locator('input[id="email"]').fill('user@example.com');
  await page.locator('input[id="password"]').fill('password123');
  await page.locator('button:has-text("Sign In")').click();
  // Selectors scattered everywhere!
});

// ✅ DO: Use page objects in tests
test('login', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.loginAsUser('user@example.com', 'password123');
  await expect(page).toHaveURL('/dashboard');
});
```

### Naming Conventions

```typescript
// ✅ DO: Clear, intention-revealing names
export class CheckoutPage {
  async fillShippingAddress(address: Address) { }
  async selectShippingMethod(method: string) { }
  async proceedToPayment() { }
  async completeCheckout() { }
  async verifyOrderConfirmation(): Promise<string> { }
  async getOrderTotal(): Promise<number> { }
}

// ❌ DON'T: Vague names
export class CheckoutPage {
  async fill(data: any) { }
  async select(option: string) { }
  async proceed() { }
  async verify() { }
}
```

---

## 3. TEST DATA MANAGEMENT

### Separate Test Data

```typescript
// ✅ DO: Store test data separately
// data/testData.json
{
  "validUsers": [
    {
      "email": "admin@example.com",
      "password": "AdminPass123!",
      "role": "admin"
    },
    {
      "email": "user@example.com",
      "password": "UserPass123!",
      "role": "user"
    }
  ],
  "invalidCredentials": [
    { "email": "invalid@example.com", "password": "wrong" }
  ]
}

// utils/dataLoader.ts
export function loadTestData(category: string) {
  const data = require('../data/testData.json');
  return data[category];
}
```

```typescript
// ❌ DON'T: Hardcode data in tests
test('login', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.loginAsUser('admin@example.com', 'AdminPass123!');
});

// ✅ DO: Load from external source
test('login with valid credentials', async ({ page }) => {
  const users = loadTestData('validUsers');
  const admin = users[0];
  const loginPage = new LoginPage(page);
  await loginPage.loginAsUser(admin.email, admin.password);
});
```

### Data Factories

```typescript
// ✅ DO: Generate realistic test data
export class TestDataFactory {
  static generateUser() {
    return {
      email: `user_${Date.now()}@example.com`,
      firstName: 'Test',
      lastName: 'User',
      password: 'SecurePass123!',
    };
  }

  static generateProduct() {
    return {
      name: `Product ${Math.random()}`,
      sku: `SKU-${Date.now()}`,
      price: Math.floor(Math.random() * 1000),
      description: 'Test product',
    };
  }

  static generateOrder(userId: string) {
    return {
      userId,
      orderId: `ORD-${Date.now()}`,
      total: Math.floor(Math.random() * 500),
      items: 3,
    };
  }
}
```

### Clean Data

```typescript
// ✅ DO: Clean up test data after execution
test.afterEach(async ({ page }) => {
  // Delete test user
  await page.goto('/admin/users');
  const testUser = await page.locator('text=test_user_*').first();
  if (testUser) {
    await testUser.click();
    await page.locator('button:has-text("Delete")').click();
    await page.locator('button:has-text("Confirm")').click();
  }
});
```

---

## 4. FIXTURES & SETUP/TEARDOWN

### Playwright Fixtures

```typescript
// ✅ DO: Use test.extend for custom fixtures
import { test as base } from '@playwright/test';

type MyFixtures = {
  loggedInPage: Page;
  testUser: { email: string; password: string };
  dbConnection: Connection;
};

export const test = base.extend<MyFixtures>({
  // Session-level fixture (expensive setup)
  dbConnection: [async ({}, use) => {
    const conn = new Connection();
    await conn.connect();
    await use(conn);
    await conn.close();
  }, { scope: 'worker' }],

  // Test-level fixture (per-test setup)
  testUser: async ({}, use) => {
    const user = TestDataFactory.generateUser();
    await user.save();
    await use(user);
    await user.delete();
  },

  // Build on other fixtures
  loggedInPage: async ({ page, testUser }, use) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await loginPage.loginAsUser(testUser.email, testUser.password);
    await use(page);
  },
});
```

### Hooks Strategy

```typescript
// ✅ DO: Use hooks for test lifecycle
test.beforeAll(async () => {
  // One-time setup: expensive operations
  console.log('Creating test database schema...');
  await createTestSchema();
  console.log('Creating initial test data...');
  await seedInitialData();
});

test.beforeEach(async ({ page }) => {
  // Per-test setup
  console.log('Cleaning up test state...');
  await page.context().clearCookies();
  // Reset specific data
  await resetTestData();
});

test.afterEach(async ({ page }, testInfo) => {
  // Per-test cleanup
  if (testInfo.status !== 'passed') {
    // Save screenshot on failure
    await page.screenshot({
      path: `failure-${testInfo.testId}.png`,
    });
  }
  // Delete test data
  await cleanupTestData();
});

test.afterAll(async () => {
  // Final cleanup
  console.log('Dropping test database...');
  await dropTestSchema();
});
```

---

## 5. ERROR HANDLING & RELIABILITY

### Retry Logic

```typescript
// ✅ DO: Use built-in Playwright retries
export default defineConfig({
  use: {
    // Auto-retry assertions
    navigationTimeout: 30000,
    actionTimeout: 10000,
  },
  webServer: {
    command: 'npm run dev',
    timeout: 120000,
    reuseExistingServer: !process.env.CI,
  },
  retries: process.env.CI ? 2 : 0,
});

// ✅ DO: Custom retry for external APIs
export async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  initialDelay = 1000
): Promise<T> {
  let delay = initialDelay;
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      console.log(`Retry ${i + 1}/${maxRetries}, waiting ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
      delay *= 2; // Exponential backoff
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Soft Assertions

```typescript
// ✅ DO: Collect multiple failures
test('user details verification', async ({ page }) => {
  const errors: string[] = [];

  // Verify each field, collect errors
  const name = await page.locator('[data-testid="name"]').textContent();
  if (name !== 'John Doe') {
    errors.push(`Name mismatch: expected "John Doe", got "${name}"`);
  }

  const email = await page.locator('[data-testid="email"]').textContent();
  if (!email?.includes('@')) {
    errors.push(`Invalid email format: ${email}`);
  }

  const status = await page.locator('[data-testid="status"]').textContent();
  if (status !== 'Active') {
    errors.push(`Status mismatch: expected "Active", got "${status}"`);
  }

  // Report all failures at once
  if (errors.length > 0) {
    throw new Error(`Multiple verification failures:\n${errors.join('\n')}`);
  }
});
```

### Timeout Strategy

```typescript
// ✅ DO: Configure intentional timeouts
export default defineConfig({
  use: {
    navigationTimeout: 30000,  // Global navigation
    actionTimeout: 10000,       // Global action
  },
});

test('wait with timeout', async ({ page }) => {
  // Use global timeout
  await page.goto('https://example.com');

  // Override for specific waits
  await page.locator('button:has-text("Submit")').click({
    timeout: 60000,  // External API, longer wait
  });

  // Don't use general timeout
  // ❌ await page.waitForTimeout(5000);  // Never!
  // ✅ await page.waitForFunction(() => true);
});
```

---

## 6. NETWORK MOCKING & INDEPENDENCE

### Mock External Services

```typescript
// ✅ DO: Mock all third-party services
test('payment processing with mocked gateway', async ({ page }) => {
  // Mock Stripe
  await page.route('**/stripe.com/**', route => route.abort());

  // Mock payment endpoint
  await page.route('**/api/process-payment', route => {
    route.fulfill({
      status: 200,
      body: JSON.stringify({
        success: true,
        transactionId: 'txn_test_123',
      }),
    });
  });

  await page.goto('https://example.com/checkout');
  await page.locator('button:has-text("Pay")').click();

  await expect(page.locator('text=Payment successful')).toBeVisible();
});

// ✅ DO: Mock analytics
await page.route('**/google-analytics/**', route => route.abort());
await page.route('**/analytics.google.com/**', route => route.abort());
await page.route('**/doubleclick.net/**', route => route.abort());
```

### Test Independence

```typescript
// ✅ DO: Tests work offline
test('search functionality', async ({ page }) => {
  // Mock search API
  await page.route('**/api/search**', route => {
    route.fulfill({
      status: 200,
      body: JSON.stringify({
        results: [
          { id: 1, title: 'Product 1' },
          { id: 2, title: 'Product 2' },
        ],
      }),
    });
  });

  await page.goto('https://example.com');
  await page.locator('input[placeholder="Search"]').fill('laptop');
  await page.locator('button:has-text("Search")').click();

  await expect(page.locator('text=Product 1')).toBeVisible();
  // Test passes even if network is down!
});
```

---

## 7. PERFORMANCE & OPTIMIZATION

### Parallelization

```typescript
// playwright.config.ts
// ✅ DO: Configure parallel workers
export default defineConfig({
  workers: process.env.CI ? 8 : 2,  // More workers in CI
  timeout: 30 * 1000,
  expect: {
    timeout: 5 * 1000,
  },
  // Run some tests serially if needed
  fullyParallel: false,
});
```

```typescript
// ✅ DO: Mark tests as serial when needed
test.describe.serial('Dashboard workflow', () => {
  test('step 1: login', async ({ page }) => { });
  test('step 2: navigate menus', async ({ page }) => { });
  test('step 3: view reports', async ({ page }) => { });
});
```

### Resource Cleanup

```typescript
// ✅ DO: Clean up after each test
test.afterEach(async ({ page, context }) => {
  // Clear cookies
  await context.clearCookies();

  // Close all pages except current
  for (const p of context.pages()) {
    if (p !== page) await p.close();
  }

  // Clear storage
  await page.evaluate(() => {
    localStorage.clear();
    sessionStorage.clear();
  });
});
```

---

## 8. REPORTING & VISIBILITY

### HTML Reports

```typescript
// playwright.config.ts
export default defineConfig({
  reporter: [
    ['html'],  // Beautiful interactive report
    ['list'],  // Console output
  ],
});
```

```bash
# View report
npx playwright show-report
```

### Video Recording

```typescript
// playwright.config.ts
export default defineConfig({
  use: {
    video: 'retain-on-failure',  // Only save on failure
    videoDir: 'test-results/videos',
  },
});
```

### Tracing

```typescript
// playwright.config.ts
export default defineConfig({
  use: {
    trace: 'on-first-retry',  // Capture on retry
  },
});

// Manual trace control
test('workflow with tracing', async ({ page }) => {
  await page.context().tracing.start({
    screenshots: true,
    snapshots: true,
  });

  // Test steps...

  await page.context().tracing.stop({
    path: `trace-${Date.now()}.zip`,
  });
});
```

### CI/CD Integration

```yaml
# .github/workflows/test.yml
name: Playwright Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run test
      
      # Upload reports
      - if: always()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

---

## 9. CODE QUALITY STANDARDS

### TypeScript Strict Mode

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
  }
}
```

### Linting & Formatting

```json
// .eslintrc.json
{
  "extends": ["eslint:recommended", "plugin:@typescript-eslint/recommended"],
  "rules": {
    "no-console": "warn",
    "@typescript-eslint/no-explicit-any": "error",
    "prefer-const": "error"
  }
}
```

```bash
# Pre-commit hook (husky)
npm install husky --save-dev
npx husky install
npx husky add .husky/pre-commit "npm run lint && npm run test"
```

---

## 10. TEAM COLLABORATION

### Onboarding Documentation

```markdown
# Getting Started Guide

## Prerequisites
- Node.js 16+
- Git

## Setup (5 minutes)
1. Clone repository
2. `npm install`
3. `npx playwright install`

## Run First Test (5 minutes)
```bash
npm run test tests/example.spec.ts
```

## Common Commands
- `npm run test` - Run all tests
- `npm run test -- --ui` - UI mode
- `npm run test -- --debug` - Debug mode
- `npm run lint` - Check code
```

### Shared Standards

```typescript
// Enforce common setup across team
// shared/fixtures.ts
export const test = base.extend<SharedFixtures>({
  // All team members use same fixtures
  loggedInPage: async ({ page }, use) => { },
  testData: async ({}, use) => { },
  apiClient: async ({}, use) => { },
});
```

---

## 11. SECURITY & SECRETS

### No Passwords in Code

```typescript
// ✅ DO: Use environment variables
const credentials = {
  email: process.env.TEST_EMAIL || 'default@example.com',
  password: process.env.TEST_PASSWORD || '',
};

// .env.local (never commit)
TEST_EMAIL=admin@example.com
TEST_PASSWORD=SecurePass123

// .github/workflows/test.yml
env:
  TEST_EMAIL: ${{ secrets.TEST_EMAIL }}
  TEST_PASSWORD: ${{ secrets.TEST_PASSWORD }}
```

### No Logging Sensitive Data

```typescript
// ✅ DO: Mask sensitive information
function maskSensitive(data: any): any {
  if (data.password) data.password = '***';
  if (data.email) data.email = data.email.replace(/(.{2}).*(@.*)/, '$1***$2');
  return data;
}

console.log('User data:', maskSensitive(userData));
```

---

## 12. COMMON MISTAKES TO AVOID

```typescript
// ❌ DON'T: Hardcode selectors
test('login', async ({ page }) => {
  await page.locator('input[id="email"]').fill('user@example.com');
});

// ✅ DO: Use page objects
test('login', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.fillEmail('user@example.com');
});

// ❌ DON'T: Use generic waits
await page.waitForTimeout(5000);

// ✅ DO: Wait for specific elements
await page.waitForSelector('button:has-text("Submit")');

// ❌ DON'T: Tests that depend on execution order
test('A: create user', async ({ page }) => {
  // Create user
});
test('B: fetch user', async ({ page }) => {
  // Depends on A running first!
});

// ✅ DO: Independent tests
test('create and fetch user', async ({ page }) => {
  // Setup, test, cleanup all in one
});

// ❌ DON'T: Skip cleanup
test('adds product to cart', async ({ page }) => {
  // Product stays in cart!
});

// ✅ DO: Clean up in afterEach
test.afterEach(async ({ page }) => {
  await page.evaluate(() => localStorage.clear());
});
```

---

## 🎯 PRIORITY CHECKLIST

### PHASE 1: ESSENTIAL (Week 1-2)
- [ ] Page Object Model architecture
- [ ] Fixtures and hooks setup
- [ ] Environment configuration
- [ ] Basic error handling
- [ ] HTML reports
- [ ] No hardcoded test data
- [ ] Independent tests

### PHASE 2: IMPORTANT (Week 3-4)
- [ ] Network mocking
- [ ] CI/CD integration
- [ ] TypeScript strict mode
- [ ] Linting/formatting
- [ ] Parallel execution
- [ ] Video recording
- [ ] Secrets management

### PHASE 3: ADVANCED (Month 2+)
- [ ] Tracing setup
- [ ] Performance monitoring
- [ ] Multi-user workflows
- [ ] Accessibility testing
- [ ] Custom reporting
- [ ] Load testing integration
- [ ] Security scanning

---

## ✅ GOLDEN RULES

1. **One Source of Truth** - Locators in POM, config centralized
2. **Test Independence** - Tests work in any order offline
3. **Fast Feedback** - <10 minutes for full suite (parallelized)
4. **Visibility Always** - Reports, videos, traces available
5. **Security Strict** - No secrets in code, ever
6. **Maintainability High** - Future you will thank present you
7. **Reliability Critical** - Flaky tests worse than no tests
8. **Team Aligned** - Standards enforced, knowledge shared

---

**Remember: Build for scale from day one. Shortcuts come back to haunt you.**
