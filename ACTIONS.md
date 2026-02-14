# Playwright Actions

## What are Actions?

**Actions** are things you do with the mouse or keyboard on a webpage.

Think of it like: **All the things you can do with a real browser**

```
Actions = Click, Type, Hover, Upload file, Select, etc.
```

---

# MOUSE ACTIONS

## 1. Click - Simple Click

### What Does It Do?
Clicks an element once (like clicking a button).

### Example 1: Click a Button

**HTML:**
```html
<button id="submitBtn">Submit</button>
```

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Click a button', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find button
  const button = page.locator('button:has-text("Submit")');
  
  // Click it
  await button.click();
  
  console.log('✓ Button clicked');
});
```

**Explanation:**
- `page.locator()` = Find the element
- `.click()` = Click it (like mouse left-click)
- Simple and straightforward

### Example 2: Click a Link

**HTML:**
```html
<a href="/products">View Products</a>
```

**Playwright Code:**
```javascript
test('Click a link', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find and click link
  const productsLink = page.locator('a:has-text("View Products")');
  await productsLink.click();
  
  console.log('✓ Link clicked');
});
```

---

## 2. Double Click - Click Twice Quickly

### What Does It Do?
Clicks an element twice very quickly (like double-clicking a file).

### Example 1: Double Click to Edit

**HTML:**
```html
<div class="editable-text">Click to edit</div>
```

**Playwright Code:**
```javascript
test('Double click to edit', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find element
  const editableText = page.locator('.editable-text');
  
  // Double click it
  await editableText.dblclick();
  
  console.log('✓ Element double clicked');
});
```

**Explanation:**
- `.dblclick()` = Double click (two clicks quickly)
- Often opens edit mode
- Like double-clicking a file to open

### Example 2: Double Click to Select Text

**HTML:**
```html
<p>Select this text</p>
```

**Playwright Code:**
```javascript
test('Double click to select word', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Double click word
  const paragraph = page.locator('p');
  await paragraph.dblclick();
  
  console.log('✓ Text selected by double click');
});
```

---

## 3. Right Click - Context Menu

### What Does It Do?
Right-clicks an element to show context menu.

### Example 1: Right Click to Open Menu

**HTML:**
```html
<div class="item">Right click me</div>
```

**Playwright Code:**
```javascript
test('Right click to open menu', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find element
  const item = page.locator('.item');
  
  // Right click it
  await item.click({ button: 'right' });
  
  console.log('✓ Right click performed');
});
```

**Explanation:**
- `.click({ button: 'right' })` = Right click
- Shows context menu
- Can then click menu options

### Example 2: Right Click and Select Delete

**HTML:**
```html
<div class="file">document.pdf</div>
```

**Playwright Code:**
```javascript
test('Right click and delete', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const file = page.locator('.file');
  
  // Right click
  await file.click({ button: 'right' });
  
  // Click delete option in menu
  const deleteOption = page.locator('text=Delete');
  await deleteOption.click();
  
  console.log('✓ Item deleted via right click');
});
```

---

## 4. Hover - Mouse Over

### What Does It Do?
Moves mouse over an element without clicking (hover effect).

### Example 1: Hover to Show Tooltip

**HTML:**
```html
<button>Hover me</button>
<!-- Tooltip shows on hover -->
```

**Playwright Code:**
```javascript
test('Hover to show tooltip', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const button = page.locator('button:has-text("Hover me")');
  
  // Hover over button
  await button.hover();
  
  // Wait for tooltip
  const tooltip = page.locator('.tooltip');
  await tooltip.waitFor({ state: 'visible' });
  
  console.log('✓ Tooltip appeared on hover');
});
```

**Explanation:**
- `.hover()` = Move mouse over element
- Triggers hover CSS styles
- Tooltips appear on hover
- No click happens

### Example 2: Hover Menu to Show Options

**HTML:**
```html
<div class="menu-item">
  Products
  <div class="submenu" style="display: none">
    <a>Electronics</a>
    <a>Clothing</a>
  </div>
</div>
```

**Playwright Code:**
```javascript
test('Hover menu item', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const menuItem = page.locator('.menu-item');
  
  // Hover over menu
  await menuItem.hover();
  
  // Submenu appears
  const submenu = page.locator('.submenu');
  await submenu.waitFor({ state: 'visible' });
  
  // Click submenu option
  const electronics = page.locator('text=Electronics');
  await electronics.click();
  
  console.log('✓ Submenu clicked');
});
```

---

## 5. Focus - Set Focus on Element

### What Does It Do?
Sets focus on an element (like clicking in a text box).

### Example 1: Focus on Input Field

**HTML:**
```html
<input type="text" placeholder="Enter name">
```

**Playwright Code:**
```javascript
test('Focus on input field', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const input = page.locator('input[placeholder="Enter name"]');
  
  // Focus on input (cursor goes there)
  await input.focus();
  
  // Now type (it goes in this input)
  await page.keyboard.type('John');
  
  console.log('✓ Input focused and text typed');
});
```

**Explanation:**
- `.focus()` = Set focus on element
- Cursor appears in input
- Keyboard input goes to focused element
- Like clicking in a text box

### Example 2: Focus and Validate

**HTML:**
```html
<input type="email" placeholder="Email">
```

**Playwright Code:**
```javascript
test('Focus triggers validation', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const emailInput = page.locator('input[type="email"]');
  
  // Focus
  await emailInput.focus();
  
  // Type invalid email
  await emailInput.fill('invalid-email');
  
  // Blur (unfocus) triggers validation
  await emailInput.blur();
  
  // Check error message
  const error = page.locator('.error-message');
  await error.waitFor({ state: 'visible' });
  
  console.log('✓ Validation triggered on blur');
});
```

---

## 6. Scroll - Move Page Up/Down

### What Does It Do?
Scrolls the page up/down to view different content.

### Example 1: Scroll Down

**Playwright Code:**
```javascript
test('Scroll down page', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Scroll down by 500 pixels
  await page.evaluate(() => {
    window.scrollBy(0, 500);
  });
  
  console.log('✓ Scrolled down 500px');
});
```

**Explanation:**
- `page.evaluate()` = Run JavaScript on page
- `window.scrollBy(x, y)` = Scroll by pixels
- First number = horizontal
- Second number = vertical
- Positive = down/right

### Example 2: Scroll to Element

**HTML:**
```html
<button id="bottomButton" style="margin-top: 2000px">
  Click me at bottom
</button>
```

**Playwright Code:**
```javascript
test('Scroll to element', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const button = page.locator('#bottomButton');
  
  // Scroll element into view
  await button.scrollIntoViewIfNeeded();
  
  // Now click it
  await button.click();
  
  console.log('✓ Scrolled to element and clicked');
});
```

**Explanation:**
- `.scrollIntoViewIfNeeded()` = Scroll to see element
- Automatically scrolls just enough
- Then you can click it
- Very useful for long pages

---

---

# KEYBOARD ACTIONS

## 1. Type - Write Text

### What Does It Do?
Types text character by character (like typing on keyboard).

### Example 1: Type in Text Field

**HTML:**
```html
<input type="text" name="firstname" placeholder="Enter first name">
```

**Playwright Code:**
```javascript
test('Type text in field', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const input = page.locator('input[name="firstname"]');
  
  // Type text
  await input.type('John', { delay: 100 }); // 100ms between each character
  
  console.log('✓ Text typed slowly');
});
```

**Explanation:**
- `.type()` = Type text character by character
- `{ delay: 100 }` = Wait 100ms between characters
- Creates realistic human typing effect
- Useful for animations

### Example 2: Type Password

**HTML:**
```html
<input type="password" name="password" placeholder="Enter password">
```

**Playwright Code:**
```javascript
test('Type password', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const passwordInput = page.locator('input[type="password"]');
  
  // Type password
  await passwordInput.type('mySecret123');
  
  console.log('✓ Password typed');
});
```

---

## 2. Fill - Set Value Directly

### What Does It Do?
Sets text value directly (faster than typing, no animation).

### Example 1: Fill Multiple Fields

**HTML:**
```html
<input type="text" name="firstname" placeholder="First Name">
<input type="text" name="lastname" placeholder="Last Name">
<input type="email" name="email" placeholder="Email">
```

**Playwright Code:**
```javascript
test('Fill multiple fields', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Fill each field directly
  await page.locator('input[name="firstname"]').fill('John');
  await page.locator('input[name="lastname"]').fill('Doe');
  await page.locator('input[name="email"]').fill('john@example.com');
  
  console.log('✓ All fields filled instantly');
});
```

**Explanation:**
- `.fill()` = Set value directly
- No typing animation
- Instant result
- Much faster than `.type()`
- Best for testing, not real user simulation

### Example 2: Fill and Submit

**Playwright Code:**
```javascript
test('Fill form and submit', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Fill form
  await page.locator('input[name="firstname"]').fill('Jane');
  await page.locator('input[name="lastname"]').fill('Smith');
  
  // Submit
  await page.locator('button[type="submit"]').click();
  
  console.log('✓ Form submitted');
});
```

---

## 3. Press Key - Single Key Press

### What Does It Do?
Presses a single key (Enter, Escape, Tab, etc).

### Example 1: Press Enter to Submit

**HTML:**
```html
<input type="text" id="searchBox" placeholder="Search...">
```

**Playwright Code:**
```javascript
test('Press Enter key', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const searchBox = page.locator('#searchBox');
  
  // Type search term
  await searchBox.fill('laptop');
  
  // Press Enter to search
  await searchBox.press('Enter');
  
  console.log('✓ Search submitted with Enter key');
});
```

**Explanation:**
- `.press('Enter')` = Press Enter key
- Submits form in many cases
- Like pressing Enter on keyboard

### Example 2: Press Escape to Close Modal

**HTML:**
```html
<div class="modal">Modal content</div>
```

**Playwright Code:**
```javascript
test('Press Escape to close modal', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const modal = page.locator('.modal');
  
  // Modal is open
  await modal.waitFor({ state: 'visible' });
  
  // Press Escape
  await page.keyboard.press('Escape');
  
  // Modal closes
  await modal.waitFor({ state: 'hidden' });
  
  console.log('✓ Modal closed with Escape key');
});
```

---

## 4. Common Keys to Press

```javascript
await page.keyboard.press('Enter');      // Submit/Confirm
await page.keyboard.press('Escape');     // Close/Cancel
await page.keyboard.press('Tab');        // Move to next field
await page.keyboard.press('Shift+Tab');  // Move to previous field
await page.keyboard.press('ArrowUp');    // Arrow up
await page.keyboard.press('ArrowDown');  // Arrow down
await page.keyboard.press('Backspace');  // Delete character
await page.keyboard.press('Delete');     // Delete character
await page.keyboard.press('Home');       // Go to start
await page.keyboard.press('End');        // Go to end
```

### Example: Navigate with Tab

**HTML:**
```html
<input id="first" placeholder="First">
<input id="second" placeholder="Second">
<input id="third" placeholder="Third">
```

**Playwright Code:**
```javascript
test('Navigate with Tab key', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const firstInput = page.locator('#first');
  
  // Focus first input
  await firstInput.focus();
  await firstInput.fill('Value1');
  
  // Tab to second
  await page.keyboard.press('Tab');
  await page.keyboard.type('Value2');
  
  // Tab to third
  await page.keyboard.press('Tab');
  await page.keyboard.type('Value3');
  
  console.log('✓ Navigated with Tab key');
});
```

---

## 5. Keyboard Combinations - Ctrl, Shift, Alt

### Example 1: Ctrl+A - Select All

**Playwright Code:**
```javascript
test('Select all with Ctrl+A', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const input = page.locator('input[name="firstname"]');
  
  // Type text
  await input.fill('John Doe');
  
  // Focus
  await input.focus();
  
  // Select all (Ctrl+A)
  await page.keyboard.press('Control+a');
  
  console.log('✓ All text selected');
});
```

**Explanation:**
- `'Control+a'` = Ctrl+A combination
- Selects all text in field

### Example 2: Ctrl+C - Copy

**Playwright Code:**
```javascript
test('Copy text with Ctrl+C', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const input = page.locator('input[name="firstname"]');
  await input.fill('Text to Copy');
  
  // Select all
  await input.focus();
  await page.keyboard.press('Control+a');
  
  // Copy
  await page.keyboard.press('Control+c');
  
  console.log('✓ Text copied');
});
```

### Example 3: Ctrl+V - Paste

**Playwright Code:**
```javascript
test('Paste text with Ctrl+V', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // This gets clipboard content and pastes
  await page.locator('input[name="firstname"]').focus();
  
  // Paste
  await page.keyboard.press('Control+v');
  
  console.log('✓ Text pasted');
});
```

---

---

# FORM ACTIONS

## 1. Checkbox - Check/Uncheck

### What Does It Do?
Checks or unchecks a checkbox.

### Example 1: Check Single Checkbox

**HTML:**
```html
<label>
  <input type="checkbox" name="agreeTerms">
  I agree to terms
</label>
```

**Playwright Code:**
```javascript
test('Check checkbox', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const checkbox = page.locator('input[name="agreeTerms"]');
  
  // Check it
  await checkbox.check();
  
  console.log('✓ Checkbox checked');
});
```

**Explanation:**
- `.check()` = Check the checkbox
- Only works for checkboxes
- Automatically clicks if not checked

### Example 2: Uncheck Checkbox

**Playwright Code:**
```javascript
test('Uncheck checkbox', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const checkbox = page.locator('input[name="agreeTerms"]');
  
  // First check it
  await checkbox.check();
  
  // Then uncheck it
  await checkbox.uncheck();
  
  console.log('✓ Checkbox unchecked');
});
```

### Example 3: Check Multiple Checkboxes

**HTML:**
```html
<input type="checkbox" value="male" name="gender">
<input type="checkbox" value="female" name="gender">
<input type="checkbox" value="other" name="gender">
```

**Playwright Code:**
```javascript
test('Check multiple checkboxes', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Check all checkboxes
  const checkboxes = page.locator('input[name="gender"]');
  
  // Check each one
  const count = await checkboxes.count();
  for (let i = 0; i < count; i++) {
    await checkboxes.nth(i).check();
  }
  
  console.log('✓ All checkboxes checked');
});
```

---

## 2. Radio Button - Select Option

### What Does It Do?
Selects a radio button (only one can be selected at a time).

### Example 1: Select Radio Button

**HTML:**
```html
<label>
  <input type="radio" name="gender" value="male">
  Male
</label>
<label>
  <input type="radio" name="gender" value="female">
  Female
</label>
```

**Playwright Code:**
```javascript
test('Select radio button', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find and click Male radio
  const maleRadio = page.locator('input[value="male"]');
  await maleRadio.check();
  
  console.log('✓ Male radio selected');
});
```

**Explanation:**
- `.check()` = Select the radio button
- Only one radio per group can be selected
- Clicking one automatically unchecks others

### Example 2: Switch Radio Selection

**Playwright Code:**
```javascript
test('Switch radio selection', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Select Male
  const maleRadio = page.locator('input[value="male"]');
  await maleRadio.check();
  console.log('✓ Male selected');
  
  // Change to Female (Male auto-unchecks)
  const femaleRadio = page.locator('input[value="female"]');
  await femaleRadio.check();
  console.log('✓ Female selected');
});
```

---

## 3. Dropdown - Select Option

### What Does It Do?
Selects an option from a dropdown/select menu.

### Example 1: Select by Visible Text

**HTML:**
```html
<select name="country">
  <option value="">Select Country</option>
  <option value="usa">United States</option>
  <option value="uk">United Kingdom</option>
  <option value="india">India</option>
</select>
```

**Playwright Code:**
```javascript
test('Select option by text', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const countrySelect = page.locator('select[name="country"]');
  
  // Select by visible text
  await countrySelect.selectOption('India');
  
  console.log('✓ India selected from dropdown');
});
```

**Explanation:**
- `.selectOption()` = Select from dropdown
- Pass visible text or value
- Automatically clicks and selects

### Example 2: Select by Value

**Playwright Code:**
```javascript
test('Select option by value', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const countrySelect = page.locator('select[name="country"]');
  
  // Select by value attribute
  await countrySelect.selectOption('usa'); // value="usa"
  
  console.log('✓ USA selected by value');
});
```

### Example 3: Select Multiple Options

**HTML:**
```html
<select name="skills" multiple>
  <option>JavaScript</option>
  <option>Python</option>
  <option>Java</option>
  <option>C#</option>
</select>
```

**Playwright Code:**
```javascript
test('Select multiple options', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const skillsSelect = page.locator('select[name="skills"]');
  
  // Select multiple options
  await skillsSelect.selectOption(['JavaScript', 'Python']);
  
  console.log('✓ Multiple options selected');
});
```

---

---

# FILE UPLOAD ACTIONS

## 1. Single File Upload

### What Does It Do?
Uploads a single file from computer.

### Example 1: Upload Single File

**HTML:**
```html
<input type="file" id="singleFile" accept=".pdf,.doc">
```

**Playwright Code:**
```javascript
test('Upload single file', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Upload file
  const fileInput = page.locator('#singleFile');
  await fileInput.setInputFiles('path/to/your/file.pdf');
  
  console.log('✓ Single file uploaded');
});
```

**Explanation:**
- `.setInputFiles()` = Select file for upload
- Pass file path as string
- File is selected in input
- Submit form to upload

### Example 2: Upload and Submit

**Playwright Code:**
```javascript
test('Upload file and submit', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Select file
  const fileInput = page.locator('input[type="file"]');
  await fileInput.setInputFiles('documents/resume.pdf');
  
  // Submit form
  const submitButton = page.locator('button[type="submit"]');
  await submitButton.click();
  
  console.log('✓ File uploaded and form submitted');
});
```

---

## 2. Multiple File Upload

### What Does It Do?
Uploads multiple files at once.

### Example 1: Upload Multiple Files

**HTML:**
```html
<input type="file" id="multipleFiles" multiple>
```

**Playwright Code:**
```javascript
test('Upload multiple files', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const fileInput = page.locator('#multipleFiles');
  
  // Upload multiple files
  await fileInput.setInputFiles([
    'documents/file1.pdf',
    'documents/file2.pdf',
    'documents/file3.doc'
  ]);
  
  console.log('✓ Multiple files uploaded');
});
```

**Explanation:**
- Pass array of file paths
- All files uploaded together
- Input must have `multiple` attribute

### Example 2: Upload Multiple and Submit

**Playwright Code:**
```javascript
test('Upload multiple files and submit', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const fileInput = page.locator('input[type="file"]');
  
  // Select multiple files
  await fileInput.setInputFiles([
    'images/photo1.jpg',
    'images/photo2.jpg',
    'images/photo3.jpg'
  ]);
  
  // Submit
  await page.locator('button[type="submit"]').click();
  
  console.log('✓ Multiple files uploaded and submitted');
});
```

---

## 3. Virtual/Dummy File Upload

### What Does It Do?
Creates and uploads dummy files without real files on disk.

### Example 1: Create Virtual File

**Playwright Code:**
```javascript
test('Upload virtual file', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const fileInput = page.locator('input[type="file"]');
  
  // Create virtual file (doesn't need to exist on disk)
  await fileInput.setInputFiles({
    name: 'test-file.txt',
    mimeType: 'text/plain',
    buffer: Buffer.from('Hello World')
  });
  
  console.log('✓ Virtual file uploaded');
});
```

**Explanation:**
- Create file object with content
- `name` = File name
- `mimeType` = File type
- `buffer` = File content
- No real file needed

### Example 2: Upload CSV Virtual File

**Playwright Code:**
```javascript
test('Upload virtual CSV file', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const fileInput = page.locator('input[type="file"]');
  
  // Create CSV data
  const csvContent = 'Name,Email,Phone\nJohn,john@example.com,123456\nJane,jane@example.com,789012';
  
  // Upload virtual CSV
  await fileInput.setInputFiles({
    name: 'data.csv',
    mimeType: 'text/csv',
    buffer: Buffer.from(csvContent)
  });
  
  console.log('✓ Virtual CSV file uploaded');
});
```

### Example 3: Upload Multiple Virtual Files

**Playwright Code:**
```javascript
test('Upload multiple virtual files', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const fileInput = page.locator('input[type="file"]');
  
  // Create multiple virtual files
  await fileInput.setInputFiles([
    {
      name: 'file1.txt',
      mimeType: 'text/plain',
      buffer: Buffer.from('Content of file 1')
    },
    {
      name: 'file2.txt',
      mimeType: 'text/plain',
      buffer: Buffer.from('Content of file 2')
    }
  ]);
  
  console.log('✓ Multiple virtual files uploaded');
});
```

---

---

# DRAG AND DROP

## 1. Drag and Drop

### What Does It Do?
Drags one element and drops it on another.

### Example 1: Simple Drag and Drop

**HTML:**
```html
<div id="draggable" class="box">Drag me</div>
<div id="target" class="drop-zone">Drop here</div>
```

**Playwright Code:**
```javascript
test('Drag and drop element', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const draggable = page.locator('#draggable');
  const target = page.locator('#target');
  
  // Drag and drop
  await draggable.dragTo(target);
  
  console.log('✓ Element dragged and dropped');
});
```

**Explanation:**
- `.dragTo()` = Drag to another element
- Takes source and destination
- Single line command
- Handles all drag/drop steps

### Example 2: Reorder List Items

**HTML:**
```html
<ul id="sortable">
  <li>Item 1</li>
  <li>Item 2</li>
  <li>Item 3</li>
</ul>
```

**Playwright Code:**
```javascript
test('Reorder list items', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const item1 = page.locator('li:has-text("Item 1")');
  const item3 = page.locator('li:has-text("Item 3")');
  
  // Drag Item 1 to Item 3 position
  await item1.dragTo(item3);
  
  console.log('✓ List items reordered');
});
```

### Example 3: Drag to Trash

**HTML:**
```html
<div class="file">document.pdf</div>
<div class="trash">Trash</div>
```

**Playwright Code:**
```javascript
test('Drag file to trash', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const file = page.locator('.file');
  const trash = page.locator('.trash');
  
  // Drag file to trash
  await file.dragTo(trash);
  
  // Verify file is gone
  await file.waitFor({ state: 'hidden' });
  
  console.log('✓ File deleted by dragging to trash');
});
```

---

---

# ADVANCED ACTIONS

## 1. Clear Input - Delete Content

### What Does It Do?
Clears all text from an input field.

### Example 1: Clear Text Field

**HTML:**
```html
<input type="text" id="searchBox" placeholder="Search">
```

**Playwright Code:**
```javascript
test('Clear input field', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const searchBox = page.locator('#searchBox');
  
  // Type something
  await searchBox.fill('Initial text');
  
  // Clear it
  await searchBox.clear();
  
  console.log('✓ Input field cleared');
});
```

**Explanation:**
- `.clear()` = Remove all text
- Field becomes empty
- Like selecting all and deleting

### Example 2: Clear and Re-enter

**Playwright Code:**
```javascript
test('Clear and re-enter text', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const input = page.locator('input[name="email"]');
  
  // First entry
  await input.fill('wrong@email.com');
  
  // Clear
  await input.clear();
  
  // New entry
  await input.fill('correct@email.com');
  
  console.log('✓ Text replaced');
});
```

---

## 2. Get Value - Read Input Content

### What Does It Do?
Gets the current value/text of a form element.

### Example 1: Get Text from Input

**Playwright Code:**
```javascript
test('Get value from input', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const input = page.locator('input[name="firstname"]');
  
  // Set value
  await input.fill('John');
  
  // Get value
  const value = await input.inputValue();
  
  console.log('Value is:', value); // Output: "John"
});
```

**Explanation:**
- `.inputValue()` = Get input field value
- Returns the text as string
- Useful for assertions

### Example 2: Compare Input Values

**Playwright Code:**
```javascript
test('Compare two inputs', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const input1 = page.locator('input[name="password"]');
  const input2 = page.locator('input[name="confirmPassword"]');
  
  // Fill both
  await input1.fill('MyPassword123');
  await input2.fill('MyPassword123');
  
  // Get values
  const val1 = await input1.inputValue();
  const val2 = await input2.inputValue();
  
  // Check if they match
  if (val1 === val2) {
    console.log('✓ Passwords match');
  }
});
```

---

## 3. Get Text - Read Element Content

### What Does It Do?
Gets the visible text content of any element.

### Example 1: Get Button Text

**Playwright Code:**
```javascript
test('Get button text', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const button = page.locator('button');
  
  // Get text
  const text = await button.textContent();
  
  console.log('Button says:', text);
});
```

### Example 2: Get Multiple Elements Text

**Playwright Code:**
```javascript
test('Get all product names', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const products = page.locator('.product-name');
  
  // Get all texts
  const names = await products.allTextContents();
  
  console.log('Products:', names);
});
```

---

## 4. Set Attribute - Change Element Property

### What Does It Do?
Changes an HTML attribute of an element.

### Example 1: Enable Disabled Button

**HTML:**
```html
<button disabled>Submit</button>
```

**Playwright Code:**
```javascript
test('Enable disabled button', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const button = page.locator('button');
  
  // Remove disabled attribute
  await button.evaluate(el => el.removeAttribute('disabled'));
  
  // Now click it
  await button.click();
  
  console.log('✓ Disabled button clicked');
});
```

---

## 5. Triple Click - Select All Text

### What Does It Do?
Triple clicks to select all text in element.

### Example 1: Select Paragraph Text

**HTML:**
```html
<p>This is a paragraph</p>
```

**Playwright Code:**
```javascript
test('Triple click to select all', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const paragraph = page.locator('p');
  
  // Triple click selects all text
  await paragraph.click({ clickCount: 3 });
  
  console.log('✓ All text selected');
});
```

**Explanation:**
- `{ clickCount: 3 }` = Click 3 times quickly
- Selects all text in element
- Same as triple-clicking with mouse

---

## 6. Wait Actions

### Example 1: Wait for Element to Appear

**Playwright Code:**
```javascript
test('Wait for element to appear', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Wait for button to be visible (up to 30 seconds)
  const button = page.locator('button:has-text("Submit")');
  await button.waitFor({ state: 'visible' });
  
  console.log('✓ Button appeared');
});
```

### Example 2: Wait for Element to Disappear

**Playwright Code:**
```javascript
test('Wait for element to disappear', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Wait for loading spinner to disappear
  const spinner = page.locator('.loading-spinner');
  await spinner.waitFor({ state: 'hidden' });
  
  console.log('✓ Loading complete');
});
```

---

## OTHER ACTIONS YOU MIGHT HAVE MISSED

### 1. Click Multiple Times Programmatically

```javascript
const button = page.locator('button');

// Click 5 times
for (let i = 0; i < 5; i++) {
  await button.click();
  await page.waitForTimeout(500); // Wait 500ms between clicks
}
```

### 2. Input with Auto-complete

```javascript
const input = page.locator('input[name="city"]');

// Type character by character (triggers autocomplete)
await input.click();
for (const char of 'New') {
  await page.keyboard.type(char);
  await page.waitForTimeout(200);
}

// Click suggestion
await page.locator('text=New York').click();
```

### 3. Mouse Move to Coordinates

```javascript
// Move to specific x, y position
await page.mouse.move(100, 200);
await page.mouse.click();
```

### 4. Slow Down All Actions

```javascript
// Add delay between all actions (useful for recording video)
await page.goto('https://testautomationpractice.blogspot.com/');

// Manually add delay
await page.waitForTimeout(500); // Wait 500ms
```

### 5. Get Element Count

```javascript
const buttons = page.locator('button');
const count = await buttons.count();
console.log('Number of buttons:', count);
```

### 6. Check if Element Exists

```javascript
const element = page.locator('.optional-element');
const isPresent = await element.count() > 0;
console.log('Element exists:', isPresent);
```

### 7. Get Element Position and Size

```javascript
const button = page.locator('button');
const box = await button.boundingBox();

console.log('X:', box.x);
console.log('Y:', box.y);
console.log('Width:', box.width);
console.log('Height:', box.height);
```

---

# COMPLETE ACTION EXAMPLE

```javascript
import { test } from '@playwright/test';

test('Complete workflow with all actions', async ({ page }) => {
  // Navigate
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Mouse: Hover and click
  const menuItem = page.locator('.menu-item');
  await menuItem.hover();
  await menuItem.click();
  
  // Keyboard: Type and press Enter
  const searchBox = page.locator('input[placeholder="Search"]');
  await searchBox.fill('laptop');
  await searchBox.press('Enter');
  
  // Scroll to element
  const productButton = page.locator('button:has-text("Add to Cart")');
  await productButton.scrollIntoViewIfNeeded();
  
  // Checkbox
  const agreeCheckbox = page.locator('input[name="agree"]');
  await agreeCheckbox.check();
  
  // Dropdown
  const countrySelect = page.locator('select[name="country"]');
  await countrySelect.selectOption('USA');
  
  // File upload
  const fileInput = page.locator('input[type="file"]');
  await fileInput.setInputFiles({
    name: 'test.txt',
    mimeType: 'text/plain',
    buffer: Buffer.from('Test content')
  });
  
  // Submit form
  const submitButton = page.locator('button[type="submit"]');
  await submitButton.click();
  
  console.log('✓ All actions completed');
});
```

---

## Summary of All Actions

| Action | What It Does | Method |
|--------|-------------|--------|
| **Click** | Single click | `.click()` |
| **Double Click** | Double click | `.dblclick()` |
| **Right Click** | Right click menu | `.click({ button: 'right' })` |
| **Hover** | Mouse over | `.hover()` |
| **Focus** | Set focus | `.focus()` |
| **Type** | Type text slowly | `.type()` |
| **Fill** | Set value | `.fill()` |
| **Clear** | Delete content | `.clear()` |
| **Check** | Check checkbox | `.check()` |
| **Uncheck** | Uncheck checkbox | `.uncheck()` |
| **Select** | Choose dropdown | `.selectOption()` |
| **Press** | Press key | `.press()` |
| **Upload File** | Choose file | `.setInputFiles()` |
| **Drag & Drop** | Drag to element | `.dragTo()` |
| **Scroll** | Move page | `.scrollIntoViewIfNeeded()` |
| **Get Value** | Read input | `.inputValue()` |
| **Get Text** | Read element | `.textContent()` |

**Perfect! All major Playwright actions in one guide!** 🎯
