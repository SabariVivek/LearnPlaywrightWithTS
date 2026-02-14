# Playwright Locators

## What are Locators?

**Locators** are ways to find elements on a web page.

Think of it like: **Using an address to find a person's house**

```
Finding an element = Finding "where something is on the page"
```

### Why Use Locators?

Before you can interact with an element (click, type, check), you need to find it first.

```javascript
// Step 1: Find the element (using locator)
const emailInput = page.locator('input[name="email"]');

// Step 2: Interact with it (now that we found it)
await emailInput.fill('test@example.com');
```

---

## Types of Locators

Playwright has **2 categories** of locators:

1. **Built-in Locators** - Easy and recommended
2. **Advanced Locators** - More powerful combinations

---

# BUILT-IN LOCATORS

## 1. Locator by ID

### What Does It Do?
Finds element by its unique **id** attribute.

Think of it like: **Finding house by house number**

### Syntax
```javascript
page.locator('#elementIdName')
```

### Example 1: Find Submit Button by ID

**HTML on website:**
```html
<button id="submitBtn">Submit Form</button>
```

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Find button by ID', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find button using its id
  const submitButton = page.locator('#submitBtn');
  
  // Click it
  await submitButton.click();
  
  console.log('✓ Button found and clicked using ID');
});
```

**Explanation:**
- `#submitBtn` = Find element with id="submitBtn"
- `#` = Means "find by id"
- The button is found and clicked

### Example 2: Find Email Input by ID

**HTML:**
```html
<input id="emailField" type="email" placeholder="Enter email">
```

**Playwright Code:**
```javascript
test('Fill email field by ID', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find email field using ID
  const emailField = page.locator('#emailField');
  
  // Type in it
  await emailField.fill('john@example.com');
  
  console.log('✓ Email field found and filled using ID');
});
```

### GetBy Alternative - Find by TestId

**HTML:**
```html
<button data-testid="submitButton">Submit Form</button>
```

**Playwright Code - Using GetByTestId:**
```javascript
test('Find button by test id', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Using getByTestId (modern way)
  const submitButton = page.getByTestId('submitButton');
  
  // Click it
  await submitButton.click();
  
  console.log('✓ Button found using getByTestId');
});
```

**Explanation:**
- `getByTestId()` = Modern way to find elements
- Looks for `data-testid` attribute
- More reliable than ID
- Recommended for testing
- Difference from `#submitBtn`:
  - `#submitBtn` = Finds by id attribute (for any element)
  - `getByTestId()` = Finds by data-testid attribute (specifically for testing)

---

## 2. Locator by Name

### What Does It Do?
Finds element by its **name** attribute.

Think of it like: **Finding people by their first name in a list**

### Syntax
```javascript
page.locator('input[name="attributeName"]')
```

### Example 1: Find First Name Input

**HTML:**
```html
<input type="text" name="firstname" placeholder="Enter first name">
```

**Playwright Code:**
```javascript
test('Fill first name by name attribute', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find input using name="firstname"
  const firstNameInput = page.locator('input[name="firstname"]');
  
  // Type John
  await firstNameInput.fill('John');
  
  console.log('✓ First name input found using name attribute');
});
```

**Explanation:**
- `input[name="firstname"]` = Find input element with name="firstname"
- `input` = Type of element (input field)
- `[name="firstname"]` = Filter by name attribute
- `.fill('John')` = Type "John" in it

### Example 2: Find Last Name Input

**HTML:**
```html
<input type="text" name="lastname" placeholder="Enter last name">
```

**Playwright Code:**
```javascript
test('Fill last name by name attribute', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find input using name="lastname"
  const lastNameInput = page.locator('input[name="lastname"]');
  
  // Type Doe
  await lastNameInput.fill('Doe');
  
  console.log('✓ Last name input found using name attribute');
});
```

### GetBy Alternative - Find by Label

**HTML:**
```html
<label for="firstName">First Name:</label>
<input id="firstName" type="text" placeholder="Enter first name">
```

**Playwright Code - Using GetByLabel:**
```javascript
test('Find input by label', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Using getByLabel (looks for associated label)
  const firstNameInput = page.getByLabel('First Name:');
  
  // Type John
  await firstNameInput.fill('John');
  
  console.log('✓ Input found using getByLabel');
});
```

**Explanation:**
- `getByLabel()` = Find input by its label text
- Looks for `<label>` tag associated with input
- Very user-friendly
- Reflects how users see the form
- Difference from `input[name="firstname"]`:
  - `input[name="firstname"]` = Finds by name attribute (technical)
  - `getByLabel()` = Finds by visible label text (user-centric)

---

## 3. Locator by Role

### What Does It Do?
Finds element by its **accessibility role** (button, textbox, checkbox, etc.).

Think of it like: **Finding people by their job title**

### Syntax
```javascript
page.locator('role=roleName')
```

### Common Roles
- `button` = Clickable button
- `textbox` = Text input field
- `checkbox` = Checkbox
- `radio` = Radio button
- `link` = Link
- `heading` = Heading

### Example 1: Find Submit Button by Role

**HTML:**
```html
<button type="submit">Submit Form</button>
```

**Playwright Code:**
```javascript
test('Click button using role', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find button by its role
  const submitButton = page.locator('role=button:has-text("Submit Form")');
  
  // Click it
  await submitButton.click();
  
  console.log('✓ Button found using role attribute');
});
```

**Explanation:**
- `role=button` = Find element that acts as a button
- `:has-text("Submit Form")` = And has text "Submit Form"
- This is very reliable for buttons

### Example 2: Find Text Input by Role

**HTML:**
```html
<input type="text" placeholder="Enter your name">
```

**Playwright Code:**
```javascript
test('Fill textbox using role', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find textbox by role
  const nameField = page.locator('role=textbox').first();
  
  // Fill it
  await nameField.fill('Jane Doe');
  
  console.log('✓ Textbox found using role attribute');
});
```

### GetBy Alternative - Find by Role

**HTML:**
```html
<button type="submit">Submit Form</button>
```

**Playwright Code - Using GetByRole:**
```javascript
test('Find button using getByRole', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Using getByRole (modern way, built for accessibility)
  const submitButton = page.getByRole('button', { name: 'Submit Form' });
  
  // Click it
  await submitButton.click();
  
  console.log('✓ Button found using getByRole');
});
```

**Explanation:**
- `getByRole()` = Find by accessibility role
- First parameter = role type (button, textbox, link, etc.)
- Second parameter = { name: 'text' } = filter by visible text
- Very accessible and reliable
- Recommended for modern tests
- Difference from `role=button`:
  - `role=button` = Locator syntax (older way)
  - `getByRole()` = GetBy API (modern, recommended way)

---

## 4. Locator by Text

### What Does It Do?
Finds element by the **text it contains**.

Think of it like: **Finding a button that says "Click Me"**

### Syntax
```javascript
page.locator('text=TextContent')
// or
page.locator(':text("TextContent")')
```

### Example 1: Find Button by Text

**HTML:**
```html
<button>Submit Form</button>
```

**Playwright Code:**
```javascript
test('Click button by text', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find button containing "Submit Form"
  const submitButton = page.locator('text=Submit Form');
  
  // Click it
  await submitButton.click();
  
  console.log('✓ Button found by text content');
});
```

**Explanation:**
- `text=Submit Form` = Find element with this exact text
- Very easy and human-readable
- Works for buttons, links, headings, etc.

### Example 2: Find Link by Text

**HTML:**
```html
<a href="/products">View Products</a>
```

**Playwright Code:**
```javascript
test('Click link by text', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find link containing "View Products"
  const productsLink = page.locator('text=View Products');
  
  // Click it
  await productsLink.click();
  
  console.log('✓ Link found by text');
});
```

### GetBy Alternative - Find by Text (GetByText)

**HTML:**
```html
<button>Submit Form</button>
```

**Playwright Code - Using GetByText:**
```javascript
test('Find button using getByText', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Using getByText (modern way)
  const submitButton = page.getByText('Submit Form');
  
  // Click it
  await submitButton.click();
  
  console.log('✓ Button found using getByText');
});
```

**Explanation:**
- `getByText()` = Find by visible text content
- Simple and straightforward
- Exact match required (unless regex used)
- Very readable and maintainable
- Difference from `text=Submit Form`:
  - `text=Submit Form` = Locator syntax (older)
  - `getByText()` = GetBy API (modern, recommended)

---

## 5. Locator by Placeholder

### What Does It Do?
Finds input element by its **placeholder** text.

Think of it like: **Finding input by hint text inside it**

### Syntax
```javascript
page.locator('input[placeholder="PlaceholderText"]')
```

### Example 1: Find Email by Placeholder

**HTML:**
```html
<input type="email" placeholder="Enter your email">
```

**Playwright Code:**
```javascript
test('Find email input by placeholder', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find input by placeholder text
  const emailInput = page.locator('input[placeholder="Enter your email"]');
  
  // Type email
  await emailInput.fill('test@example.com');
  
  console.log('✓ Email input found by placeholder');
});
```

**Explanation:**
- `input[placeholder="Enter your email"]` = Find input with this placeholder
- Placeholder = The gray hint text in input fields
- Easy to read and understand

### Example 2: Find Password by Placeholder

**HTML:**
```html
<input type="password" placeholder="Enter password">
```

**Playwright Code:**
```javascript
test('Find password input by placeholder', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find password field by placeholder
  const passwordInput = page.locator('input[placeholder="Enter password"]');
  
  // Type password
  await passwordInput.fill('mypassword123');
  
  console.log('✓ Password input found by placeholder');
});
```

### GetBy Alternative - Find by Placeholder (GetByPlaceholder)

**HTML:**
```html
<input type="email" placeholder="Enter your email">
```

**Playwright Code - Using GetByPlaceholder:**
```javascript
test('Find input using getByPlaceholder', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Using getByPlaceholder (modern way)
  const emailInput = page.getByPlaceholder('Enter your email');
  
  // Type email
  await emailInput.fill('test@example.com');
  
  console.log('✓ Input found using getByPlaceholder');
});
```

**Explanation:**
- `getByPlaceholder()` = Find input by placeholder text
- Cleaner syntax than `input[placeholder="..."]`
- More readable and less error-prone
- Difference from `input[placeholder="..."]`:
  - `input[placeholder="..."]` = Attribute selector (technical)
  - `getByPlaceholder()` = GetBy API (modern, dedicated method)

---

## 6. Locator by Label

### What Does It Do?
Finds input element **associated with a label**.

Think of it like: **Finding input by the label name next to it**

### Syntax
```javascript
page.locator('input[aria-label="LabelName"]')
// or
page.locator('role=textbox[aria-label="LabelName"]')
```

### Example 1: Find Checkbox by Label

**HTML:**
```html
<label>
  <input type="checkbox" aria-label="Accept Terms">
  I accept the terms
</label>
```

**Playwright Code:**
```javascript
test('Check checkbox by label', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find checkbox by its label
  const termsCheckbox = page.locator('input[aria-label="Accept Terms"]');
  
  // Check it
  await termsCheckbox.check();
  
  console.log('✓ Checkbox found by label');
});
```

**Explanation:**
- `input[aria-label="Accept Terms"]` = Find input with aria-label
- `aria-label` = Accessibility label for element
- `.check()` = Check the checkbox

### Example 2: Find Gender Radio by Label

**HTML:**
```html
<label>
  <input type="radio" aria-label="Male">
  Male
</label>
```

**Playwright Code:**
```javascript
test('Select radio by label', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find radio by label
  const maleRadio = page.locator('input[aria-label="Male"]');
  
  // Check it
  await maleRadio.check();
  
  console.log('✓ Radio button found by label');
});
```

### GetBy Alternative - Find by Label (Already Covered Above)

**Note:** We already showed `getByLabel()` earlier in the Name section. That's the GetBy alternative for labels.

**Quick Comparison:**
```javascript
// Old way - aria-label
const checkbox = page.locator('input[aria-label="Accept Terms"]');

// New way - getByLabel
const checkbox = page.getByLabel('Accept Terms:');

// getBy approach is:
// - More readable
// - Less prone to errors
// - Recommended for modern tests
```

---

# ADVANCED LOCATORS

## 1. Locator Filter - AND Condition

### What Does It Do?
Finds element that matches **multiple conditions at once** (AND logic).

Think of it like: **Finding people who are tall AND have brown hair**

### Syntax
```javascript
page.locator('element').filter({ option: value })
```

### Example 1: Find Button with Specific Text Among Many Buttons

**HTML:**
```html
<button>Delete</button>
<button>Edit</button>
<button>Submit</button>
```

**Playwright Code:**
```javascript
test('Find specific button with filter', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find button that has text "Submit"
  const submitButton = page.locator('button').filter({ hasText: 'Submit' });
  
  // Click it
  await submitButton.click();
  
  console.log('✓ Specific button found using filter');
});
```

**Explanation:**
- `page.locator('button')` = Find all buttons
- `.filter({ hasText: 'Submit' })` = Keep only ones with "Submit" text
- This combines multiple buttons and filters to exact one

### Example 2: Find Enabled Button Among Disabled Ones

**HTML:**
```html
<button disabled>Disabled Button</button>
<button>Active Button</button>
```

**Playwright Code:**
```javascript
test('Find enabled button', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find button that is enabled
  const enabledButton = page.locator('button').filter({ disabled: false });
  
  // Click it
  await enabledButton.click();
  
  console.log('✓ Enabled button found using filter');
});
```

---

## 2. Locator AND - Combine Multiple Conditions

### What Does It Do?
Combines multiple locators with **AND** logic.

Think of it like: **Find inputs that are BOTH in a form AND have type email**

### Syntax
```javascript
page.locator('selector1 >> and >> selector2')
// or
page.locator('selector1:has(selector2)')
```

### Example 1: Find Email Input in Contact Form

**HTML:**
```html
<form id="contactForm">
  <input type="email" placeholder="Email">
</form>
```

**Playwright Code:**
```javascript
test('Find email in specific form', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find form AND find input inside it
  const contactForm = page.locator('form#contactForm');
  const emailInput = contactForm.locator('input[type="email"]');
  
  // Fill email
  await emailInput.fill('contact@example.com');
  
  console.log('✓ Email input found in specific form');
});
```

**Explanation:**
- `page.locator('form#contactForm')` = Find the form
- `.locator('input[type="email"]')` = Find email input INSIDE that form
- This ensures we get the right input even if multiple exist

### Example 2: Find Button Inside a Div

**HTML:**
```html
<div class="modal">
  <button>Close</button>
</div>
```

**Playwright Code:**
```javascript
test('Find button in specific container', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find div AND button inside it
  const modal = page.locator('div.modal');
  const closeButton = modal.locator('button');
  
  // Click close
  await closeButton.click();
  
  console.log('✓ Button found inside container');
});
```

---

## 3. Locator OR - First Match of Multiple

### What Does It Do?
Tries **multiple options** and uses the first one that exists.

Think of it like: **Find button by "Click Me" OR "Click" OR "Submit"**

### Syntax
```javascript
page.locator('selector1, selector2, selector3')
```

### Example 1: Find Button With Different Text Options

**HTML:**
```html
<button>Submit Form</button>
<!-- or could be -->
<button>Submit</button>
```

**Playwright Code:**
```javascript
test('Find button with multiple text options', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Try to find button with "Submit" OR "Send" OR "Post"
  const button = page.locator('button:has-text("Submit"), button:has-text("Send"), button:has-text("Post")').first();
  
  // Click first one found
  await button.click();
  
  console.log('✓ Button found with multiple options');
});
```

**Explanation:**
- Comma (,) = OR logic
- Looks for "Submit" button, if not found tries "Send", then "Post"
- `.first()` = Takes first match
- Useful when elements might have different text in different environments

### Example 2: Find Input With Multiple Possible Names

**HTML:**
```html
<input name="email" placeholder="Email address">
<!-- or could be -->
<input name="emailAddress" placeholder="Email address">
```

**Playwright Code:**
```javascript
test('Find input with multiple name options', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Try to find input with name="email" OR name="emailAddress"
  const emailInput = page.locator('input[name="email"], input[name="emailAddress"]').first();
  
  // Fill it
  await emailInput.fill('user@example.com');
  
  console.log('✓ Input found with alternative names');
});
```

---

## 4. Locator Not - Exclude Elements

### What Does It Do?
Finds elements **excluding certain ones**.

Think of it like: **Find buttons EXCEPT the disabled ones**

### Syntax
```javascript
page.locator('selector:not(:disabled)')
```

### Example 1: Find Active Buttons (Not Disabled)

**HTML:**
```html
<button disabled>Delete</button>
<button>Submit</button>
<button>Cancel</button>
```

**Playwright Code:**
```javascript
test('Find active buttons (not disabled)', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find buttons that are NOT disabled
  const activeButtons = page.locator('button:not(:disabled)');
  
  // Get count
  const count = await activeButtons.count();
  
  console.log('✓ Active buttons found:', count); // Should be 2
});
```

**Explanation:**
- `button:not(:disabled)` = Find buttons excluding disabled ones
- `:not()` = Exclusion filter
- `:disabled` = Disabled state
- This finds only clickable buttons

### Example 2: Find Visible Elements (Not Hidden)

**HTML:**
```html
<div style="display: none">Hidden</div>
<div>Visible</div>
```

**Playwright Code:**
```javascript
test('Find visible elements', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find all divs and get visible ones
  const visibleElements = page.locator('div:visible');
  
  // Count them
  const count = await visibleElements.count();
  
  console.log('✓ Visible elements found:', count);
});
```

---

## 5. Locator Has Text - Partial Text Match

### What Does It Do?
Finds element containing specific text **anywhere inside it**.

Think of it like: **Finding button that contains word "Submit" in its text**

### Syntax
```javascript
page.locator('selector:has-text("text")')
```

### Example 1: Find Button Containing Text

**HTML:**
```html
<button>Click to Submit Form</button>
```

**Playwright Code:**
```javascript
test('Find button with partial text', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find button that has "Submit" anywhere in text
  const submitButton = page.locator('button:has-text("Submit")');
  
  // Click it
  await submitButton.click();
  
  console.log('✓ Button found with partial text match');
});
```

**Explanation:**
- `:has-text("Submit")` = Element contains this text
- Works even if text is "Click to Submit Form" (Submit is part of it)
- Very flexible for finding elements

### Example 2: Find Link Containing "Products"

**HTML:**
```html
<a href="/shop">Our Products are Here</a>
```

**Playwright Code:**
```javascript
test('Find link with partial text', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find link containing "Products"
  const productsLink = page.locator('a:has-text("Products")');
  
  // Click it
  await productsLink.click();
  
  console.log('✓ Link found with partial text match');
});
```

---

## 6. Locator First and Last

### What Does It Do?
Gets the **first or last** element from multiple matches.

Think of it like: **If you find 5 buttons, get the first one or last one**

### Syntax
```javascript
page.locator('selector').first()
page.locator('selector').last()
```

### Example 1: Get First Button

**HTML:**
```html
<button>Delete</button>
<button>Edit</button>
<button>Submit</button>
```

**Playwright Code:**
```javascript
test('Get first button', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find all buttons, get FIRST one
  const firstButton = page.locator('button').first();
  
  // Click it
  await firstButton.click();
  
  console.log('✓ First button clicked');
});
```

**Explanation:**
- `page.locator('button')` = Find all buttons
- `.first()` = Get the first one
- Useful when you know there are multiple and want specific one

### Example 2: Get Last Input

**HTML:**
```html
<input name="firstname">
<input name="lastname">
<input name="email">
```

**Playwright Code:**
```javascript
test('Get last input', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find all inputs, get LAST one
  const lastInput = page.locator('input').last();
  
  // Fill it
  await lastInput.fill('test@example.com');
  
  console.log('✓ Last input filled (email field)');
});
```

---

## 7. Locator Nth - Get Specific Element

### What Does It Do?
Gets element at **specific position** (1st, 2nd, 3rd, etc).

Think of it like: **Get 2nd button from 5 buttons**

### Syntax
```javascript
page.locator('selector').nth(index)  // index starts at 0
```

### Example 1: Get Second Button

**HTML:**
```html
<button>Delete</button>
<button>Edit</button>      <!-- 2nd button -->
<button>Submit</button>
```

**Playwright Code:**
```javascript
test('Get specific button by position', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find all buttons, get 2nd one (index 1)
  const secondButton = page.locator('button').nth(1);
  
  // Click it
  await secondButton.click();
  
  console.log('✓ Second button clicked');
});
```

**Explanation:**
- Index starts at 0
- `nth(0)` = First element
- `nth(1)` = Second element
- `nth(2)` = Third element

### Example 2: Get Third Input Field

**HTML:**
```html
<input name="firstname">
<input name="lastname">
<input name="email">     <!-- 3rd element -->
```

**Playwright Code:**
```javascript
test('Get third input field', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find all inputs, get 3rd one (index 2)
  const emailInput = page.locator('input').nth(2);
  
  // Fill it
  await emailInput.fill('contact@example.com');
  
  console.log('✓ Third input filled (email)');
});
```

---

# LOCATOR PRIORITY (Best Practices)

## Recommended Order of Locators (Best to Worst)

### 1. ✅ Best - ID
```javascript
page.locator('#elementId')
```
**Why?** Unique, never changes, fastest
**Use when:** Element has an id

---

### 2. ✅ Very Good - Role + Text
```javascript
page.locator('role=button:has-text("Submit")')
```
**Why?** Semantic, accessible, reliable
**Use when:** Finding buttons, links, inputs

---

### 3. ✅ Very Good - Text
```javascript
page.locator('text=Submit Form')
```
**Why?** Human-readable, easy to understand
**Use when:** Looking for visible text on page

---

### 4. ✅ Good - Name Attribute
```javascript
page.locator('input[name="firstname"]')
```
**Why?** Specific to form elements, stable
**Use when:** Finding form inputs

---

### 5. ✅ Good - Placeholder
```javascript
page.locator('input[placeholder="Enter name"]')
```
**Why?** Easy to identify inputs
**Use when:** Input has placeholder text

---

### 6. ✅ Fair - Role Filter
```javascript
page.locator('button').filter({ hasText: 'Submit' })
```
**Why?** Good for multiple similar elements
**Use when:** Need to filter among many elements

---

### 7. ❌ Avoid - Generic Selectors
```javascript
page.locator('button:last-of-type')
```
**Why?** Fragile, breaks easily with HTML changes
**Use only when:** No other option available

---

## Complete Priority Table

| Priority | Locator Type | Example | Stability |
|----------|-------------|---------|-----------|
| 1 | ID | `#submitBtn` | Excellent |
| 2 | Role + Text | `role=button:has-text("Submit")` | Excellent |
| 3 | Text | `text=Submit` | Very Good |
| 4 | Name | `input[name="email"]` | Very Good |
| 5 | Placeholder | `input[placeholder="Enter name"]` | Good |
| 6 | Filter | `button.filter({ hasText: 'Submit' })` | Good |
| 7 | Role | `role=textbox` | Fair |
| 8 | Multiple Conditions | Combined locators | Fair |

---

## How to Choose the Best Locator

```
START HERE:
    ↓
Does element have ID?
    ↓ YES → Use ID!
    ↓ NO
    ↓
Can you find by Role + Text?
    ↓ YES → Use Role + Text!
    ↓ NO
    ↓
Can you find by visible Text?
    ↓ YES → Use Text!
    ↓ NO
    ↓
Is it a form input with name?
    ↓ YES → Use Name!
    ↓ NO
    ↓
Does it have placeholder text?
    ↓ YES → Use Placeholder!
    ↓ NO
    ↓
Use Filter or Multiple Conditions
```

---

## Complete Real-World Example

```javascript
import { test } from '@playwright/test';

test('Complete form with various locators', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // 1. Use ID (Best)
  const form = page.locator('#registrationForm');
  
  // 2. Use Name (Very Good)
  const firstNameInput = page.locator('input[name="firstname"]');
  await firstNameInput.fill('John');
  
  // 3. Use Placeholder (Good)
  const lastNameInput = page.locator('input[placeholder="Enter last name"]');
  await lastNameInput.fill('Doe');
  
  // 4. Use Text (Very Good)
  const maleRadio = page.locator('text=Male');
  await maleRadio.check();
  
  // 5. Use Role + Filter (Good)
  const countrySelect = page.locator('role=combobox').filter({ hasText: 'Country' });
  await countrySelect.selectOption('USA');
  
  // 6. Use Text for Button (Very Good)
  const submitButton = page.locator('button:has-text("Submit")');
  await submitButton.click();
  
  console.log('✓ Form submitted with multiple locator types');
});
```

---

## Common Locator Mistakes to Avoid

### ❌ Wrong - Too Generic
```javascript
page.locator('button')  // Which button? 10 on page!
```

### ✅ Right - Specific
```javascript
page.locator('button:has-text("Submit")')  // Clear which button
```

---

### ❌ Wrong - Fragile Index
```javascript
page.locator('form').nth(2)  // Breaks if order changes
```

### ✅ Right - Semantic
```javascript
page.locator('form#registrationForm')  // Stable identifier
```

---

### ❌ Wrong - Depending on Position
```javascript
page.locator('input').nth(0)  // Fragile positioning
```

### ✅ Right - Using Attributes
```javascript
page.locator('input[name="firstname"]')  // Stable attribute
```

---

## Summary

### Built-in Locators to Use
1. **ID** - `#elementId` (Best)
2. **Text** - `text=TextContent` (Best)
3. **Name** - `input[name="fieldName"]` (Very Good)
4. **Placeholder** - `input[placeholder="Hint"]` (Good)
5. **Role** - `role=button` (Good)
6. **Label** - `input[aria-label="Label"]` (Good)

### Advanced Locators to Use
1. **Filter** - `.filter({ hasText: 'text' })` (Good)
2. **First/Last** - `.first()`, `.last()` (Good)
3. **Nth** - `.nth(index)` (Fair)
4. **Not** - `:not(:disabled)` (Good)
5. **Has Text** - `:has-text("text")` (Good)
6. **Multiple** - `selector1, selector2` (Fair)

---

# GETBY METHODS - Modern Approach

## What are GetBy Methods?

**GetBy methods** are modern, recommended alternatives to regular locators.

Think of it like: **New way to find elements (better, more readable, more maintainable)**

### Why Use GetBy?
- **More readable** - Clear intent
- **Better accessibility** - Finds elements like users do
- **Less error-prone** - Dedicated methods for each case
- **Recommended by Playwright** - Modern best practice
- **Better performance** - Optimized implementations

---

## GetBy Methods - Complete List

### 1. getByRole() - Find by Accessibility Role

**What it does:** Finds element by its accessibility role.

**Example:**
```javascript
test('Find elements by role', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find button
  const submitButton = page.getByRole('button', { name: 'Submit' });
  await submitButton.click();
  
  // Find link
  const homeLink = page.getByRole('link', { name: 'Home' });
  await homeLink.click();
  
  // Find textbox
  const emailInput = page.getByRole('textbox', { name: 'Email' });
  await emailInput.fill('test@example.com');
  
  // Find checkbox
  const agreeCheckbox = page.getByRole('checkbox', { name: 'I agree' });
  await agreeCheckbox.check();
  
  console.log('✓ Elements found by role');
});
```

---

### 2. getByText() - Find by Visible Text

**What it does:** Finds element containing exact or partial text.

**Example:**
```javascript
test('Find elements by text', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Exact text match
  const button = page.getByText('Submit Form');
  await button.click();
  
  // Partial match (case insensitive)
  const link = page.getByText('View Products', { exact: false });
  await link.click();
  
  console.log('✓ Elements found by text');
});
```

---

### 3. getByLabel() - Find by Label Text

**What it does:** Finds input by associated label.

**Example:**
```javascript
test('Find inputs by label', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find input by its label
  const emailInput = page.getByLabel('Email Address:');
  await emailInput.fill('user@example.com');
  
  const passwordInput = page.getByLabel('Password:');
  await passwordInput.fill('myPassword123');
  
  const agreeCheckbox = page.getByLabel('I agree to terms');
  await agreeCheckbox.check();
  
  console.log('✓ Inputs found by label');
});
```

---

### 4. getByPlaceholder() - Find by Placeholder Text

**What it does:** Finds input by its placeholder text.

**Example:**
```javascript
test('Find inputs by placeholder', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find by placeholder
  const nameInput = page.getByPlaceholder('Enter your name');
  await nameInput.fill('John Doe');
  
  const emailInput = page.getByPlaceholder('Email address');
  await emailInput.fill('john@example.com');
  
  const searchBox = page.getByPlaceholder('Search products...');
  await searchBox.fill('laptop');
  
  console.log('✓ Inputs found by placeholder');
});
```

---

### 5. getByTestId() - Find by Test ID (Recommended for Testing)

**What it does:** Finds element by data-testid attribute. **BEST FOR TESTING!**

**Example:**
```javascript
test('Find elements by test id', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Most reliable for automated tests
  const submitButton = page.getByTestId('submit-btn');
  await submitButton.click();
  
  const userProfile = page.getByTestId('user-profile-section');
  await userProfile.hover();
  
  const deleteButton = page.getByTestId('delete-action');
  await deleteButton.click();
  
  console.log('✓ Elements found by test id');
});
```

**Why getByTestId is Best:**
- Developers can add `data-testid` specifically for testing
- Not dependent on visual text (which can change)
- Not affected by CSS styling changes
- Most stable and reliable
- Professional testing approach

---

### 6. getByAltText() - Find by Alt Text (Images)

**What it does:** Finds image by its alt text.

**Example:**
```javascript
test('Find images by alt text', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find image by alt text
  const productImage = page.getByAltText('Laptop image');
  await productImage.click();
  
  const logo = page.getByAltText('Company logo');
  const isVisible = await logo.isVisible();
  
  console.log('Logo visible:', isVisible);
});
```

---

### 7. getByTitle() - Find by Title Attribute

**What it does:** Finds element by its title attribute (hover text).

**Example:**
```javascript
test('Find elements by title', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find by title (hover tooltip)
  const helpIcon = page.getByTitle('Click for help');
  await helpIcon.click();
  
  const infoButton = page.getByTitle('More information');
  await infoButton.hover();
  
  console.log('✓ Elements found by title');
});
```

---

## Complete Example - Using GetBy Methods

```javascript
import { test } from '@playwright/test';

test('Complete form using all GetBy methods', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // 1. GetByLabel - find form inputs
  const firstNameInput = page.getByLabel('First Name:');
  await firstNameInput.fill('John');
  
  // 2. GetByPlaceholder - find search
  const searchInput = page.getByPlaceholder('Search products');
  await searchInput.fill('laptop');
  await searchInput.press('Enter');
  
  // 3. GetByRole - find buttons and selects
  const submitButton = page.getByRole('button', { name: 'Submit' });
  const countrySelect = page.getByRole('combobox', { name: 'Country' });
  await countrySelect.selectOption('USA');
  
  // 4. GetByText - find links and headings
  const homeLink = page.getByText('Home');
  await homeLink.click();
  
  // 5. GetByTestId - most reliable for testing
  const userForm = page.getByTestId('user-form');
  const deleteButton = page.getByTestId('delete-user');
  
  // 6. GetByAltText - find images
  const productImage = page.getByAltText('Product thumbnail');
  await productImage.click();
  
  // 7. GetByTitle - find tooltips
  const helpIcon = page.getByTitle('Get help');
  await helpIcon.hover();
  
  await submitButton.click();
  
  console.log('✓ Form completed using all GetBy methods');
});
```

---

## GetBy vs Regular Locators Comparison

| Task | Regular Locator | GetBy Method | Recommended |
|------|-----------------|--------------|-------------|
| Find by ID | `#elementId` | N/A (no getById) | Use regular |
| Find by role | `role=button` | `getByRole()` | **GetBy** |
| Find by text | `text=Submit` | `getByText()` | **GetBy** |
| Find by label | `input[aria-label="..."` | `getByLabel()` | **GetBy** |
| Find by placeholder | `input[placeholder="..."` | `getByPlaceholder()` | **GetBy** |
| Find by test id | `[data-testid="..."` | `getByTestId()` | **GetBy** |
| Find by alt text | `img[alt="..."` | `getByAltText()` | **GetBy** |
| Find by title | `[title="..."` | `getByTitle()` | **GetBy** |

---

## Priority - GetBy First!

### NEW RECOMMENDED PRIORITY ORDER

1. ✅ **GetByTestId()** - `getByTestId('my-button')` (BEST FOR TESTING)
2. ✅ **GetByRole()** - `getByRole('button', { name: 'Submit' })` (Most accessible)
3. ✅ **GetByLabel()** - `getByLabel('Email:')` (User-centric)
4. ✅ **GetByPlaceholder()** - `getByPlaceholder('Enter email')` (Clear intent)
5. ✅ **GetByText()** - `getByText('Submit')` (Simple visibility)
6. ⚠️ **Regular Locators** - `page.locator()` (Fallback only)

---

## Quick Reference - GetBy Cheat Sheet

```javascript
// By role (with name filter)
page.getByRole('button', { name: 'Submit' })
page.getByRole('checkbox', { name: 'I agree' })
page.getByRole('link', { name: 'Home' })

// By text
page.getByText('Submit Form')
page.getByText(/partial.*match/i)  // Case insensitive regex

// By label
page.getByLabel('Email Address:')

// By placeholder
page.getByPlaceholder('Enter password')

// By test id (BEST for testing!)
page.getByTestId('submit-button')

// By alt text (images)
page.getByAltText('Product image')

// By title (tooltips)
page.getByTitle('Click for help')
```

---

### Golden Rule
**Always choose **GetBy methods first**, then regular locators only as fallback!**

GetBy methods are:
- ✅ More readable
- ✅ More maintainable  
- ✅ More accessible
- ✅ Recommended by Playwright team
- ✅ Better for long-term test stability

**Modern testing best practice: Use GetBy > Regular Locators** 🎯
