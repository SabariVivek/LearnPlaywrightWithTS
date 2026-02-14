# Playwright Page Object Model (POM)

## What is Page Object Model?

**POM** = Organize test code by separating pages into classes/objects.

Think of it like: **Each webpage gets its own manager that handles all interactions**

```
Without POM:
Test File = Mix of page logic + test logic (messy)

With POM:
Page Class (handles page interactions)
Test File (handles test logic only)
```

---

## Why Use Page Object Model?

### Without POM (Bad)

```javascript
// Test file mixed with selectors and logic
test('Login test', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Selectors scattered everywhere
  await page.fill('input[name="username"]', 'admin');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button:has-text("Login")');
  
  // More selectors
  const greeting = await page.locator('.user-greeting').textContent();
  
  // Mix of logic and assertions
  console.log('Logged in as:', greeting);
});

test('Logout test', async ({ page }) => {
  // Repeat selectors again!
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  await page.fill('input[name="username"]', 'admin');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button:has-text("Login")');
  
  // Logout selector
  await page.click('button:has-text("Logout")');
});
```

Problems:
- ❌ Selectors repeated in every test
- ❌ Hard to maintain (change selector = change all tests)
- ❌ Test code is cluttered
- ❌ Can't reuse page logic

### With POM (Good)

```javascript
// Page Object
class LoginPage {
  async login(page, username, password) {
    await page.fill('input[name="username"]', username);
    await page.fill('input[name="password"]', password);
    await page.click('button:has-text("Login")');
  }
  
  async logout(page) {
    await page.click('button:has-text("Logout")');
  }
}

// Test file - clean and simple
test('Login test', async ({ page }) => {
  const loginPage = new LoginPage();
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  await loginPage.login(page, 'admin', 'password123');
});

test('Logout test', async ({ page }) => {
  const loginPage = new LoginPage();
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  await loginPage.login(page, 'admin', 'password123');
  await loginPage.logout(page);
});
```

Benefits:
- ✅ Selectors in one place (easy to change)
- ✅ Reusable methods
- ✅ Clean test code
- ✅ Professional structure

---

## Basic POM Structure

```
project/
├── pages/
│   ├── LoginPage.ts
│   ├── DashboardPage.ts
│   └── ProfilePage.ts
├── tests/
│   ├── login.test.ts
│   ├── dashboard.test.ts
│   └── profile.test.ts
```

---

---

# BASIC PAGE OBJECT

## Structure

```javascript
class PageName {
  // Properties (selectors)
  readonly selector1 = 'selector';
  
  // Methods (actions)
  async method1() { }
  
  // Assertions/getters
  async getValue() { }
}
```

---

## Example 1: Simple Login Page Object

### What Happens

Create class → Define selectors → Create methods → Use in tests

**pages/LoginPage.ts:**
```typescript
import { Page } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  
  // Selectors
  readonly usernameInput = 'input[name="username"]';
  readonly passwordInput = 'input[name="password"]';
  readonly loginButton = 'button:has-text("Login")';
  readonly errorMessage = '.error-message';
  
  constructor(page: Page) {
    this.page = page;
  }
  
  // Methods
  async goto() {
    await this.page.goto('https://testautomationpractice.blogspot.com/');
  }
  
  async fillUsername(username: string) {
    await this.page.fill(this.usernameInput, username);
  }
  
  async fillPassword(password: string) {
    await this.page.fill(this.passwordInput, password);
  }
  
  async clickLogin() {
    await this.page.click(this.loginButton);
  }
  
  // Combined method
  async login(username: string, password: string) {
    await this.fillUsername(username);
    await this.fillPassword(password);
    await this.clickLogin();
  }
  
  // Getter/verification
  async getErrorMessage(): Promise<string> {
    return await this.page.locator(this.errorMessage).textContent() || '';
  }
  
  async isErrorVisible(): Promise<boolean> {
    return await this.page.locator(this.errorMessage).isVisible();
  }
}
```

**tests/login.test.ts:**
```typescript
import { test } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';

test('Successful login', async ({ page }) => {
  const loginPage = new LoginPage(page);
  
  await loginPage.goto();
  await loginPage.login('admin', 'password123');
  
  console.log('✓ Logged in successfully');
});

test('Invalid password', async ({ page }) => {
  const loginPage = new LoginPage(page);
  
  await loginPage.goto();
  await loginPage.login('admin', 'wrongpassword');
  
  const errorVisible = await loginPage.isErrorVisible();
  console.log('Error shown:', errorVisible);
});
```

**Explanation:**
- `constructor(page)` = Store page reference
- Selectors as properties = Easy to change
- Methods for each action
- Getters for verification
- Tests just call methods = Clean!

---

## Example 2: Dashboard Page Object

### What Happens

Dashboard with multiple elements → Create page object → Tests use it

**pages/DashboardPage.ts:**
```typescript
import { Page } from '@playwright/test';

export class DashboardPage {
  readonly page: Page;
  
  // Elements
  readonly userGreeting = '.user-greeting';
  readonly logoutButton = 'button:has-text("Logout")';
  readonly profileLink = 'a:has-text("Profile")';
  readonly settingsLink = 'a:has-text("Settings")';
  readonly dashboardTitle = 'h1';
  
  constructor(page: Page) {
    this.page = page;
  }
  
  async goto() {
    await this.page.goto('https://testautomationpractice.blogspot.com/dashboard');
  }
  
  async getUserGreeting(): Promise<string> {
    return await this.page.locator(this.userGreeting).textContent() || '';
  }
  
  async clickLogout() {
    await this.page.click(this.logoutButton);
  }
  
  async goToProfile() {
    await this.page.click(this.profileLink);
  }
  
  async goToSettings() {
    await this.page.click(this.settingsLink);
  }
  
  async getDashboardTitle(): Promise<string> {
    return await this.page.locator(this.dashboardTitle).textContent() || '';
  }
  
  async isDashboardVisible(): Promise<boolean> {
    return await this.page.locator(this.dashboardTitle).isVisible();
  }
}
```

**tests/dashboard.test.ts:**
```typescript
import { test, expect } from '@playwright/test';
import { DashboardPage } from '../pages/DashboardPage';

test('Dashboard displays correctly', async ({ page }) => {
  const dashboard = new DashboardPage(page);
  
  await dashboard.goto();
  
  const isVisible = await dashboard.isDashboardVisible();
  expect(isVisible).toBeTruthy();
  
  const title = await dashboard.getDashboardTitle();
  console.log('Dashboard title:', title);
});

test('User can navigate to profile', async ({ page }) => {
  const dashboard = new DashboardPage(page);
  
  await dashboard.goto();
  await dashboard.goToProfile();
  
  console.log('✓ Navigated to profile');
});
```

---

---

# ADVANCED POM PATTERNS

## Pattern 1: Locator Builder Method

### What Happens

Use method to build selectors dynamically

**pages/ShoppingPage.ts:**
```typescript
import { Page } from '@playwright/test';

export class ShoppingPage {
  readonly page: Page;
  
  constructor(page: Page) {
    this.page = page;
  }
  
  // Dynamic selector building
  private getProductByName(productName: string) {
    return `//div[@class='product' and contains(., '${productName}')]`;
  }
  
  async addProductToCart(productName: string) {
    const productSelector = this.getProductByName(productName);
    const addButton = `${productSelector}//button[text()='Add to Cart']`;
    
    await this.page.click(addButton);
  }
  
  async getProductPrice(productName: string): Promise<string> {
    const productSelector = this.getProductByName(productName);
    const priceLocator = `${productSelector}//span[@class='price']`;
    
    return await this.page.locator(priceLocator).textContent() || '';
  }
}
```

**Usage:**
```typescript
test('Add laptops to cart', async ({ page }) => {
  const shopping = new ShoppingPage(page);
  
  await shopping.addProductToCart('Laptop Pro');
  await shopping.addProductToCart('Laptop Basic');
  
  const price1 = await shopping.getProductPrice('Laptop Pro');
  const price2 = await shopping.getProductPrice('Laptop Basic');
  
  console.log('Prices:', price1, price2);
});
```

---

## Pattern 2: Page Navigation (Return New Page)

### What Happens

Click button → Navigate to new page → Return new page object

**pages/HomePage.ts:**
```typescript
import { Page } from '@playwright/test';
import { LoginPage } from './LoginPage';

export class HomePage {
  readonly page: Page;
  
  readonly loginLink = 'a:has-text("Login")';
  
  constructor(page: Page) {
    this.page = page;
  }
  
  async goto() {
    await this.page.goto('https://testautomationpractice.blogspot.com/');
  }
  
  // Returns new page object
  async clickLogin(): Promise<LoginPage> {
    await this.page.click(this.loginLink);
    return new LoginPage(this.page);
  }
}
```

**Usage:**
```typescript
test('Navigate from home to login', async ({ page }) => {
  const home = new HomePage(page);
  
  await home.goto();
  
  // clickLogin returns LoginPage object
  const login = await home.clickLogin();
  
  // Now use LoginPage methods
  await login.login('admin', 'password123');
});
```

---

## Pattern 3: Base Page Class

### What Happens

Create base class → Extend for common functionality → All pages inherit

**pages/BasePage.ts:**
```typescript
import { Page } from '@playwright/test';

export class BasePage {
  protected page: Page;
  readonly baseURL = 'https://testautomationpractice.blogspot.com';
  
  constructor(page: Page) {
    this.page = page;
  }
  
  // Common methods
  async goto(path: string) {
    await this.page.goto(this.baseURL + path);
  }
  
  async click(selector: string) {
    await this.page.click(selector);
  }
  
  async fill(selector: string, value: string) {
    await this.page.fill(selector, value);
  }
  
  async getText(selector: string): Promise<string> {
    return await this.page.locator(selector).textContent() || '';
  }
  
  async isVisible(selector: string): Promise<boolean> {
    return await this.page.locator(selector).isVisible();
  }
  
  async waitForSelector(selector: string) {
    await this.page.waitForSelector(selector);
  }
}
```

**pages/LoginPage.ts (extends BasePage):**
```typescript
import { BasePage } from './BasePage';

export class LoginPage extends BasePage {
  readonly usernameInput = 'input[name="username"]';
  readonly passwordInput = 'input[name="password"]';
  readonly loginButton = 'button:has-text("Login")';
  
  async goto() {
    await super.goto('/'); // Use base class method
  }
  
  async login(username: string, password: string) {
    await this.fill(this.usernameInput, username); // Use base method
    await this.fill(this.passwordInput, password);
    await this.click(this.loginButton);
  }
}

// All methods like click, fill, getText available!
```

---

---

# COMPLETE POM EXAMPLE

## Project Structure

```
pages/
  ├── BasePage.ts
  ├── LoginPage.ts
  ├── DashboardPage.ts
  └── ProfilePage.ts
tests/
  └── complete-flow.test.ts
```

---

## BasePage.ts

```typescript
import { Page } from '@playwright/test';

export class BasePage {
  protected page: Page;
  readonly baseURL = 'https://testautomationpractice.blogspot.com';
  
  constructor(page: Page) {
    this.page = page;
  }
  
  async goto(path: string = '') {
    await this.page.goto(this.baseURL + path);
  }
  
  async click(selector: string) {
    await this.page.click(selector);
  }
  
  async fill(selector: string, value: string) {
    await this.page.fill(selector, value);
  }
  
  async getText(selector: string): Promise<string> {
    return await this.page.locator(selector).textContent() || '';
  }
  
  async isVisible(selector: string): Promise<boolean> {
    return await this.page.locator(selector).isVisible();
  }
  
  async waitForLoadState() {
    await this.page.waitForLoadState('networkidle');
  }
}
```

---

## LoginPage.ts

```typescript
import { BasePage } from './BasePage';

export class LoginPage extends BasePage {
  readonly usernameInput = 'input[name="username"]';
  readonly passwordInput = 'input[name="password"]';
  readonly loginButton = 'button:has-text("Login")';
  readonly errorMessage = '.error-message';
  
  async goto() {
    await super.goto('/');
  }
  
  async login(username: string, password: string) {
    await this.fill(this.usernameInput, username);
    await this.fill(this.passwordInput, password);
    await this.click(this.loginButton);
    await this.waitForLoadState();
  }
  
  async getErrorMessage(): Promise<string> {
    return await this.getText(this.errorMessage);
  }
  
  async isErrorVisible(): Promise<boolean> {
    return await this.isVisible(this.errorMessage);
  }
}
```

---

## DashboardPage.ts

```typescript
import { BasePage } from './BasePage';

export class DashboardPage extends BasePage {
  readonly userGreeting = '.user-greeting';
  readonly logoutButton = 'button:has-text("Logout")';
  readonly profileLink = 'a:has-text("Profile")';
  readonly settingsLink = 'a:has-text("Settings")';
  
  async goto() {
    await super.goto('/dashboard');
  }
  
  async getUserGreeting(): Promise<string> {
    return await this.getText(this.userGreeting);
  }
  
  async goToProfile() {
    await this.click(this.profileLink);
    await this.waitForLoadState();
  }
  
  async goToSettings() {
    await this.click(this.settingsLink);
    await this.waitForLoadState();
  }
  
  async logout() {
    await this.click(this.logoutButton);
  }
}
```

---

## ProfilePage.ts

```typescript
import { BasePage } from './BasePage';

export class ProfilePage extends BasePage {
  readonly userNameField = 'input[name="username"]';
  readonly emailField = 'input[name="email"]';
  readonly saveButton = 'button:has-text("Save")';
  readonly successMessage = '.success-message';
  
  async goto() {
    await super.goto('/profile');
  }
  
  async updateProfile(data: { username?: string; email?: string }) {
    if (data.username) {
      await this.fill(this.userNameField, data.username);
    }
    if (data.email) {
      await this.fill(this.emailField, data.email);
    }
    
    await this.click(this.saveButton);
    await this.waitForLoadState();
  }
  
  async getSaveMessage(): Promise<string> {
    return await this.getText(this.successMessage);
  }
}
```

---

## complete-flow.test.ts

```typescript
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';
import { ProfilePage } from '../pages/ProfilePage';

test('Complete user flow', async ({ page }) => {
  console.log('=== Complete User Flow Test ===\n');
  
  // Step 1: Login
  console.log('Step 1: Login');
  const login = new LoginPage(page);
  await login.goto();
  await login.login('admin', 'password123');
  console.log('✓ Logged in\n');
  
  // Step 2: View Dashboard
  console.log('Step 2: Dashboard');
  const dashboard = new DashboardPage(page);
  await dashboard.goto();
  
  const greeting = await dashboard.getUserGreeting();
  console.log('Greeting:', greeting);
  console.log('✓ Dashboard viewed\n');
  
  // Step 3: Update Profile
  console.log('Step 3: Update Profile');
  const profile = new ProfilePage(page);
  await profile.goto();
  
  await profile.updateProfile({
    username: 'admin_updated',
    email: 'admin@example.com'
  });
  
  const message = await profile.getSaveMessage();
  console.log('Message:', message);
  console.log('✓ Profile updated\n');
  
  // Step 4: Logout
  console.log('Step 4: Logout');
  await dashboard.goto();
  await dashboard.logout();
  console.log('✓ Logged out\n');
  
  console.log('=== Test Complete ===');
});
```

---

---

# BEST PRACTICES

### ✅ Do This
```typescript
// Clear, descriptive selector names
readonly submitButton = 'button[type="submit"]';
readonly emailInput = 'input[name="email"]';

// Methods for actions
async fillEmail(email: string) {
  await this.page.fill(this.emailInput, email);
}

// Methods for verification
async isEmailVisible(): Promise<boolean> {
  return await this.page.locator(this.emailInput).isVisible();
}

// Combine related actions
async login(username: string, password: string) {
  await this.fillUsername(username);
  await this.fillPassword(password);
  await this.clickLogin();
}

// Use BasePage for common functionality
export class LoginPage extends BasePage {
  // Inherits goto, click, fill, etc.
}

// Return page objects for navigation
async clickLogin(): Promise<DashboardPage> {
  await this.click(this.loginButton);
  return new DashboardPage(this.page);
}
```

### ❌ Don't Do This
```typescript
// Don't expose selectors in tests
test('Login', async ({ page }) => {
  await page.fill('input[name="username"]', 'admin'); // Wrong!
});

// Don't put test logic in page objects
async login() {
  // DON'T: Logic and assertions in POM
  await this.page.fill(this.userInput, 'admin');
  const isVisible = await this.page.locator(this.dashboard).isVisible();
  if (!isVisible) {
    throw new Error('Login failed');
  }
}

// Don't make methods too specific
async fillUsernameAndPasswordAndClickLogin(u: string, p: string) {
  // Too much
}

// Don't put wait logic in tests
test('Login', async ({ page }) => {
  const login = new LoginPage(page);
  await login.login('admin', 'password123');
  
  await page.waitForTimeout(2000); // NO! - Add to POM instead
});

// Don't mix assertions in POM
async login() {
  // DON'T: Don't assert in POM
  expect(this.isLoggedIn()).toBeTruthy();
}
```

---

# COMPLETE EXAMPLE WITH FORM

## pages/FormPage.ts

```typescript
import { BasePage } from './BasePage';

export class FormPage extends BasePage {
  // Form elements
  readonly firstNameInput = 'input[name="firstName"]';
  readonly lastNameInput = 'input[name="lastName"]';
  readonly emailInput = 'input[name="email"]';
  readonly countrySelect = 'select[name="country"]';
  readonly agreeCheckbox = 'input[type="checkbox"]';
  readonly submitButton = 'button:has-text("Submit")';
  readonly successMessage = '.success';
  
  async goto() {
    await super.goto('/form');
  }
  
  async fillForm(data: {
    firstName: string;
    lastName: string;
    email: string;
    country: string;
  }) {
    await this.fill(this.firstNameInput, data.firstName);
    await this.fill(this.lastNameInput, data.lastName);
    await this.fill(this.emailInput, data.email);
    
    await this.page.selectOption(this.countrySelect, data.country);
  }
  
  async checkAgree() {
    await this.page.check(this.agreeCheckbox);
  }
  
  async submit() {
    await this.click(this.submitButton);
    await this.waitForLoadState();
  }
  
  async getSuccessMessage(): Promise<string> {
    return await this.getText(this.successMessage);
  }
  
  async submitForm(data: {
    firstName: string;
    lastName: string;
    email: string;
    country: string;
  }) {
    await this.fillForm(data);
    await this.checkAgree();
    await this.submit();
  }
}
```

---

---

# KEY TAKEAWAYS

- ✅ **POM** = Organize pages into classes
- ✅ **Selectors** = Keep in POM, not in tests
- ✅ **Methods** = One per action/assertion
- ✅ **BasePage** = Extend for common functionality
- ✅ **Reuse** = Same POM in multiple tests
- ✅ **Maintainability** = Change selector in POM, all tests updated
- ✅ **Readability** = Tests focus on logic, not selectors
- ✅ **Navigation** = Return page objects for flow
- ✅ **Clean** = Separate test logic from page logic

**Proper POM = Maintainable, professional, scalable tests!** 🎯
