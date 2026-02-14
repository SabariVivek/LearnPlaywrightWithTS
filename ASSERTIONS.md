# Playwright Assertions

## What are Assertions?

**Assertions** are checks or verifications in your tests. They compare actual results with expected results.

Think of it like: **Checking if something is true or false**

### Simple Analogy
- **Expected**: Product price should be $99
- **Actual**: Product shows $99
- **Assertion**: Check if both match ✓ (Test passes)

---

## Types of Assertions

Playwright has **2 main types** of assertions:

### 1. **Auto-Retrying Assertions** (Recommended)
These assertions automatically wait and retry until the condition passes or timeout is reached.

**When to use**: Always, unless you have a specific reason not to.

```javascript
// Auto-retrying - waits up to 30 seconds
await expect(page.locator('text=Success')).toBeVisible();
// Will keep checking every millisecond until visible or timeout
```

### 2. **Non-Retrying Assertions**
These assertions check the condition immediately. If it fails, they fail immediately without retrying.

**When to use**: Rarely, usually when you want instant feedback.

```javascript
// Non-retrying - checks immediately
expect(5).toBe(5);
expect(items.length).toBeLessThan(10);
```

**Important Note**: Prefer auto-retrying assertions whenever possible to avoid flaky tests.

---

## Common Auto-Retrying Assertions

### 1. Check Count of Elements

```javascript
// Check if exactly 5 products are displayed
await expect(page.locator('.product-item')).toHaveCount(5);
```

**Real Example**: Testing e-commerce website
```javascript
import { test, expect } from '@playwright/test';

test('Verify product count on page', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Check if 10 products are shown
  await expect(page.locator('.product')).toHaveCount(10);
  
  console.log('✓ Exactly 10 products found');
});
```

---

### 2. Check if Element is Enabled

```javascript
// Check if button is clickable (enabled)
await expect(page.locator('button#submit')).toBeEnabled();
```

**Real Example**: Testing form submission
```javascript
test('Submit button should be enabled when form is filled', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Initially button is disabled
  await expect(page.locator('button#submit')).toBeDisabled();
  
  // Fill form
  await page.fill('input[name="firstname"]', 'John');
  
  // After filling, button becomes enabled
  await expect(page.locator('button#submit')).toBeEnabled();
  
  console.log('✓ Button enabled correctly');
});
```

---

### 3. Check if Element is Disabled

```javascript
// Check if element is NOT clickable
await expect(page.locator('button#submit')).toBeDisabled();
```

**Real Example**: Testing locked features
```javascript
test('Premium features should be disabled for free users', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Premium button should be disabled
  await expect(page.locator('button#premium')).toBeDisabled();
  
  console.log('✓ Premium button is disabled');
});
```

---

### 4. Check if Element is Visible

```javascript
// Check if element is displayed on screen
await expect(page.locator('.success-message')).toBeVisible();
```

**Real Example**: Testing form submission success
```javascript
test('Success message appears after form submission', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Fill and submit form
  await page.fill('input[name="firstname"]', 'Jane');
  await page.click('button[type="submit"]');
  
  // Check if success message appears
  await expect(page.locator('text=Form submitted successfully')).toBeVisible();
  
  console.log('✓ Success message visible');
});
```

---

### 5. Check if Element is Hidden

```javascript
// Check if element is NOT visible (hidden)
await expect(page.locator('.error-message')).toBeHidden();
```

**Real Example**: Testing validation messages
```javascript
test('Error message should be hidden when form is valid', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Error message should not be visible initially
  await expect(page.locator('.error-message')).toBeHidden();
  
  console.log('✓ Error message is hidden');
});
```

---

### 6. Check if Element Contains Text

```javascript
// Check if element has specific text
await expect(page.locator('h1')).toHaveText('Welcome to Shop');
```

**Real Example**: Testing page titles
```javascript
test('Page should display correct title', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Check page heading
  await expect(page.locator('h1')).toHaveText('Practice Test Automation');
  
  console.log('✓ Page title is correct');
});
```

---

### 7. Check Element Attribute Value

```javascript
// Check if element has specific attribute value
await expect(page.locator('input[name="email"]')).toHaveAttribute('type', 'email');
```

**Real Example**: Testing input field types
```javascript
test('Email input should have correct type', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Check that email field has type="email"
  await expect(page.locator('input[name="email"]')).toHaveAttribute('type', 'email');
  
  console.log('✓ Email input has correct type');
});
```

---

### 8. Check Element ID Value

```javascript
// Check if element has specific id
await expect(page.locator('button')).toHaveId('submitBtn');
```

**Real Example**: Testing form button IDs
```javascript
test('Submit button should have correct id', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Check button id
  await expect(page.locator('button[type="submit"]')).toHaveId('submitBtn');
  
  console.log('✓ Button has correct id');
});
```

---

### 9. Check Element Class Value

```javascript
// Check if element has specific CSS class
await expect(page.locator('button')).toHaveClass('primary-btn');
```

**Real Example**: Testing button styling
```javascript
test('Active button should have active class', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Check if active button has active class
  await expect(page.locator('button.active')).toHaveClass('active-state');
  
  console.log('✓ Button has correct class');
});
```

---

### 10. Check Page URL

```javascript
// Check if page URL matches expected value
await expect(page).toHaveURL('https://testautomationpractice.blogspot.com/');
```

**Real Example**: Testing navigation
```javascript
test('After login, user should be on dashboard', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Login
  await page.fill('input[name="username"]', 'admin');
  await page.fill('input[name="password"]', 'password');
  await page.click('button[type="submit"]');
  
  // Check if redirected to dashboard
  await expect(page).toHaveURL('**/dashboard');
  
  console.log('✓ Redirected to dashboard');
});
```

---

### 11. Check Page Title

```javascript
// Check if page title (browser tab title) matches
await expect(page).toHaveTitle('Practice Test Automation');
```

**Real Example**: Testing browser tab title
```javascript
test('Page title should be correct', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Check browser tab title
  await expect(page).toHaveTitle('Automation Practice Site');
  
  console.log('✓ Page title is correct');
});
```

---

## Non-Retrying Assertions

### 1. Check Value Equals

```javascript
// Check if value is exactly equal
expect(5).toBe(5);
expect('hello').toBe('hello');
```

**Example**: Testing calculation
```javascript
test('Calculate product total', async ({ page }) => {
  const price = 99;
  const quantity = 2;
  const total = price * quantity;
  
  // Check if total is 198
  expect(total).toBe(198);
  
  console.log('✓ Total calculated correctly');
});
```

---

### 2. Check Number is Less Than

```javascript
// Check if number is less than value
expect(5).toBeLessThan(10);
```

**Example**: Testing minimum items requirement
```javascript
test('Number of items in cart should be less than 100', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const cartItems = await page.locator('.cart-item').count();
  
  // Check if items are less than 100
  expect(cartItems).toBeLessThan(100);
  
  console.log('✓ Cart items within limit');
});
```

---

## Negative Matchers (Opposite Checks)

You can check for the **opposite** by adding `.not` before the matcher:

```javascript
// These check for opposite conditions:
expect(value).not.toEqual(0);
await expect(locator).not.toHaveText('Error');
await expect(page).not.toHaveURL('/login');
```

### Examples

```javascript
test('Elements should NOT have error', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Check if error message is NOT visible
  await expect(page.locator('.error')).not.toBeVisible();
  
  // Check if button is NOT disabled
  await expect(page.locator('button#submit')).not.toBeDisabled();
  
  // Check if field does NOT have value
  await expect(page.locator('input[name="email"]')).not.toHaveValue('invalid@');
  
  console.log('✓ No errors found');
});
```

---

## Custom Error Messages

You can add custom messages to make failures clearer:

```javascript
await expect(locator, 'Custom Error Message').toHaveText('some text');
```

**Example**: Testing with custom messages
```javascript
test('Login form validation', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Fill form
  await page.fill('input[name="email"]', 'test@example.com');
  await page.click('button[type="submit"]');
  
  // Check with custom error message
  await expect(
    page.locator('.success-message'),
    'Expected success message after login'
  ).toBeVisible();
  
  console.log('✓ Custom message shown if fails');
});
```

If this assertion fails, you'll see: **"Expected success message after login"**

---

## Soft Assertions

**By default**, when an assertion fails, the test stops immediately.

**Soft assertions** allow the test to continue even if they fail. The test is marked as failed, but other assertions still run.

### Hard Assertion (Default - Test Stops)
```javascript
await expect(page.locator('text=Success')).toBeVisible();
// If this fails, test stops here
await page.goto('https://example2.com');
// This line never runs
```

### Soft Assertion (Test Continues)
```javascript
await expect.soft(page.locator('text=Success')).toBeVisible();
// If this fails, test continues
await page.goto('https://example2.com');
// This line DOES run even if above assertion failed
```

**Real Example**: Testing multiple things but not stopping on first failure
```javascript
test('Soft assertions - test continues on failure', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // These are soft - test continues even if they fail
  await expect.soft(page.locator('h1')).toHaveText('Welcome');
  await expect.soft(page.locator('.logo')).toBeVisible();
  await expect.soft(page.locator('button#submit')).toBeEnabled();
  
  // This runs regardless of above results
  await page.fill('input[name="email"]', 'test@example.com');
  
  console.log('✓ All soft assertions checked');
  // Test marked as failed if any soft assertion failed
});
```

---

## Complete Real-World Example

```javascript
import { test, expect } from '@playwright/test';

test('Complete form submission with all assertions', async ({ page }) => {
  // Navigate
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Auto-retrying assertions
  await expect(page).toHaveTitle('Automation Practice Site');
  await expect(page.locator('h1')).toHaveText('Practice Test Automation');
  
  // Check elements exist
  await expect(page.locator('input[name="firstname"]')).toBeEnabled();
  
  // Fill form
  await page.fill('input[name="firstname"]', 'John');
  await page.fill('input[name="lastname"]', 'Doe');
  await page.fill('input[name="email"]', 'john@example.com');
  
  // Check country dropdown has email attribute
  await expect(page.locator('select[name="country"]')).toHaveAttribute('name', 'country');
  
  // Select options
  await page.selectOption('select[name="country"]', 'USA');
  
  // Check checkbox
  await page.check('input[value="male"]');
  
  // Check if submit button is enabled
  await expect(page.locator('button[type="submit"]')).toBeEnabled();
  
  // Submit
  await page.click('button[type="submit"]');
  
  // Verify success
  await expect(page.locator('text=Form submitted successfully')).toBeVisible();
  
  // Verify error message is hidden
  await expect(page.locator('.error-message')).not.toBeVisible();
  
  // Non-retrying assertion for calculations
  const formCount = await page.locator('form').count();
  expect(formCount).toBeGreaterThan(0);
  
  console.log('✓ All assertions passed');
});
```

---

## Assertion Quick Reference

| Assertion | What It Checks | Example |
|-----------|----------------|---------|
| `toHaveCount()` | Number of matching elements | `toHaveCount(5)` |
| `toBeEnabled()` | Element is clickable | `toBeEnabled()` |
| `toBeDisabled()` | Element is not clickable | `toBeDisabled()` |
| `toBeVisible()` | Element is displayed | `toBeVisible()` |
| `toBeHidden()` | Element is not visible | `toBeHidden()` |
| `toHaveText()` | Element has text | `toHaveText('Hello')` |
| `toHaveAttribute()` | Element has attribute value | `toHaveAttribute('type', 'email')` |
| `toHaveId()` | Element has specific id | `toHaveId('submit')` |
| `toHaveClass()` | Element has CSS class | `toHaveClass('active')` |
| `toHaveURL()` | Page URL matches | `toHaveURL('/dashboard')` |
| `toHaveTitle()` | Page title matches | `toHaveTitle('Home')` |
| `toBe()` | Values are equal | `toBe(5)` |
| `toBeLessThan()` | Number is less | `toBeLessThan(10)` |

---

## Summary

- **Auto-retrying assertions** = Use these (they wait and retry)
- **Non-retrying assertions** = Use rarely (immediate checks only)
- **Negative matchers** = Add `.not` to check opposite
- **Custom messages** = Make failures clear
- **Soft assertions** = Test continues on failure (use `expect.soft()`)

**Best Practice**: Always use auto-retrying assertions first, add `.not` for negatives, and use soft assertions only when needed! 🎯
