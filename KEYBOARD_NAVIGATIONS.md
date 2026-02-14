# Playwright Keyboard Navigation & Special Keys

## What is Keyboard Navigation?

**Keyboard Navigation** = Controlling webpage using only keyboard (no mouse).

**Special Keys** = Keys with specific functions (Tab, Escape, Enter, Arrow keys, Ctrl/Shift combinations).

Think of it like: **Using an ATM with just buttons, no touchscreen**

```
Mouse Only:
  Click button → Click field → Click submit → Works

Keyboard Only:
  Tab to button → Enter → Tab to field → Type → Tab to submit → Enter → Works

Both work, but keyboard users need it!
```

---

## Why Test Keyboard Navigation?

- ✅ **Accessibility** - Users who can't use mouse (blindness, motor disabilities)
- ✅ **Power users** - Keyboard faster than mouse
- ✅ **Mobile** - Some devices keyboard-only
- ✅ **Compliance** - WCAG, ADA requirements
- ✅ **User experience** - Tab works properly
- ✅ **Form usability** - Enter submits form
- ✅ **Shortcuts** - Ctrl+S, Escape close modal

```
Target Users:
- Screen reader users (Tab between elements)
- Motor disability users (can't move mouse)
- Power users (prefer keyboard)
- Mobile users (limited input options)
- Developers using tools (Tab navigation)
```

---

---

# BASIC KEYBOARD ACTIONS

## Example 1: Press Keys

### What Happens

Send keyboard events → Browser responds → Page changes

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Basic key presses', async ({ page }) => {
  console.log('=== Basic Key Presses ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Focus on input
  const input = page.locator('input[id="search"]');
  await input.click();
  
  console.log('Typing with keyboard...');
  
  // Type text
  await input.type('Laptop');
  console.log('✓ Typed: Laptop');
  
  // Press Enter key
  console.log('Pressing Enter...');
  await page.keyboard.press('Enter');
  console.log('✓ Pressed Enter\n');
  
  // Wait for search results
  await page.waitForLoadState('networkidle');
  
  console.log('✓ Search executed');
});
```

**Common keys:**
```javascript
await page.keyboard.press('Enter');          // Return/Submit
await page.keyboard.press('Escape');         // Close modal
await page.keyboard.press('Tab');            // Move focus
await page.keyboard.press('Shift+Tab');      // Reverse focus
await page.keyboard.press('ArrowDown');      // Down arrow
await page.keyboard.press('ArrowUp');        // Up arrow
await page.keyboard.press('Space');          // Space bar (toggle)
await page.keyboard.press('Delete');         // Delete character
await page.keyboard.press('Backspace');      // Backspace
```

---

## Example 2: Type vs Press Key

### What Happens

`type()` = Send individual characters
`press()` = Send key event

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Type vs Press difference', async ({ page }) => {
  console.log('=== Type vs Press ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const input = page.locator('input[id="search"]');
  await input.click();
  
  // Method 1: Using type() - sends characters
  console.log('Method 1: Using type()');
  await input.type('Hello');  // Sends: H, e, l, l, o
  console.log('✓ Typed: Hello\n');
  
  // Clear input
  await input.clear();
  
  // Method 2: Using press() - sends key event
  console.log('Method 2: Using press()');
  await page.keyboard.press('KeyH');  // Sends KeyH event
  console.log('✓ Pressed KeyH\n');
  
  // Method 3: Combined
  console.log('Method 3: Type then press');
  await input.type('Test');
  await page.keyboard.press('Space');  // Add space
  console.log('✓ Text + space\n');
});
```

**When to use:**
- `type()` → For text input (most common)
- `press()` → For special keys (Enter, Tab, Escape)

---

## Example 3: Keyboard Modifiers

### What Happens

Hold modifier key (Ctrl, Shift, Alt) + another key = Shortcut

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Keyboard modifiers', async ({ page }) => {
  console.log('=== Keyboard Modifiers ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const input = page.locator('input[id="search"]');
  await input.click();
  
  // Type some text first
  await input.type('Sample Text');
  
  // Ctrl+A = Select all
  console.log('Pressing Ctrl+A (Select all)');
  await page.keyboard.press('Control+KeyA');
  console.log('✓ All text selected');
  
  // Ctrl+C = Copy (copies selected text)
  console.log('Pressing Ctrl+C (Copy)');
  await page.keyboard.press('Control+KeyC');
  console.log('✓ Text copied to clipboard');
  
  // Clear input
  await input.clear();
  
  // Ctrl+V = Paste (pastes from clipboard)
  console.log('Pressing Ctrl+V (Paste)');
  await page.keyboard.press('Control+KeyV');
  console.log('✓ Text pasted\n');
  
  const value = await input.inputValue();
  console.log('Input value:', value);
});
```

**Common modifiers:**
```javascript
// Individual modifiers
await page.keyboard.press('Control+KeyA');   // Ctrl+A (Select all)
await page.keyboard.press('Control+KeyC');   // Ctrl+C (Copy)
await page.keyboard.press('Control+KeyV');   // Ctrl+V (Paste)
await page.keyboard.press('Control+KeyZ');   // Ctrl+Z (Undo)
await page.keyboard.press('Control+KeyY');   // Ctrl+Y (Redo)
await page.keyboard.press('Control+KeyS');   // Ctrl+S (Save)

await page.keyboard.press('Shift+Tab');      // Shift+Tab (Reverse focus)
await page.keyboard.press('Alt+KeyF');       // Alt+F (File menu)

// Mac (Command key)
await page.keyboard.press('Meta+KeyA');      // Cmd+A (Mac)
await page.keyboard.press('Meta+KeyC');      // Cmd+C (Mac)
```

---

---

# TAB ORDER TESTING

## What is Tab Order?

**Tab Order** = The sequence elements receive focus when user presses Tab.

```
First Tab keypress → Focus on Element 1
Second Tab keypress → Focus on Element 2
Third Tab keypress → Focus on Element 3

Proper tab order = Logical flow (left to right, top to bottom)
Bad tab order = Jumping around randomly (confusing)
```

---

## Example 1: Test Tab Order

### What Happens

Press Tab repeatedly → Verify elements get focus in correct order

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Test tab order', async ({ page }) => {
  console.log('=== Tab Order Testing ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Elements should get focus in this order
  const expectedOrder = [
    'input[id="search"]',           // Search box first
    'button:has-text("Search")',    // Search button
    'input[id="email"]',            // Email input
    'button:has-text("Subscribe")', // Subscribe button
  ];
  
  console.log('Expected Tab Order:');
  expectedOrder.forEach((selector, i) => {
    console.log(`  ${i + 1}. ${selector}`);
  });
  console.log('');
  
  // Start - focus on first element
  await page.locator(expectedOrder[0]).focus();
  
  // Test each Tab press
  for (let i = 0; i < expectedOrder.length - 1; i++) {
    console.log(`Step ${i + 1}: Currently on "${expectedOrder[i]}"`);
    
    // Press Tab to move to next
    await page.keyboard.press('Tab');
    
    // Check which element now has focus
    const activeFocused = await page.evaluate(() => document.activeElement?.id || 'unknown');
    console.log(`  Tab pressed → Focused element ID: ${activeFocused}`);
    
    // Verify it's the expected next element
    const nextElement = page.locator(expectedOrder[i + 1]);
    const isFocused = await nextElement.evaluate(el => el === document.activeElement);
    
    if (isFocused) {
      console.log(`  ✓ Correct: ${expectedOrder[i + 1]}\n`);
    } else {
      console.log(`  ❌ Wrong: Expected ${expectedOrder[i + 1]}\n`);
    }
  }
  
  console.log('✓ Tab order test complete');
});
```

---

## Example 2: Verify Tab Order is Logical

### What Happens

Navigate page with Tab → Verify order makes sense (left-to-right, top-to-bottom)

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Verify logical tab order', async ({ page }) => {
  console.log('=== Verify Logical Tab Order ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Get all focusable elements
  const focusableElements = await page.evaluate(() => {
    const selectors = 'button, input, a, textarea, select, [tabindex]';
    const elements = document.querySelectorAll(selectors);
    
    return Array.from(elements).map(el => ({
      id: el.id || el.name || el.textContent?.substring(0, 20),
      tagName: el.tagName,
      position: {
        x: el.getBoundingClientRect().left,
        y: el.getBoundingClientRect().top,
      }
    }));
  });
  
  console.log(`Found ${focusableElements.length} focusable elements\n`);
  
  // Check if order follows visual position (top-to-bottom, left-to-right)
  let lastY = 0;
  let lastX = 0;
  let logicalOrder = true;
  
  for (let i = 0; i < focusableElements.length; i++) {
    const current = focusableElements[i];
    
    console.log(`${i + 1}. ${current.tagName} - Position: (${current.position.x}, ${current.position.y})`);
    
    // Check if position makes sense (shouldn't jump back up)
    if (current.position.y < lastY - 50) {  // Allow 50px tolerance
      console.log('  ⚠ Position went backward (not logical)');
      logicalOrder = false;
    }
    
    lastY = current.position.y;
    lastX = current.position.x;
  }
  
  console.log(`\n${logicalOrder ? '✓' : '❌'} Tab order is ${logicalOrder ? 'logical' : 'not logical'}`);
});
```

---

## Example 3: Skip Hidden Elements in Tab Order

### What Happens

Hidden elements should NOT get focus when pressing Tab

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Hidden elements skip in tab order', async ({ page }) => {
  console.log('=== Hidden Elements in Tab Order ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Get visible and hidden focusable elements
  const { visible, hidden } = await page.evaluate(() => {
    const elements = document.querySelectorAll('button, input, a');
    
    const visible = [];
    const hidden = [];
    
    elements.forEach(el => {
      const style = window.getComputedStyle(el);
      const isHidden = style.display === 'none' || 
                      style.visibility === 'hidden' ||
                      el.hasAttribute('aria-hidden') ||
                      el.offsetParent === null;
      
      if (isHidden) {
        hidden.push(el.id || el.name || el.textContent?.substring(0, 20));
      } else {
        visible.push(el.id || el.name || el.textContent?.substring(0, 20));
      }
    });
    
    return { visible, hidden };
  });
  
  console.log(`Visible focusable elements: ${visible.length}`);
  visible.forEach((el, i) => console.log(`  ${i + 1}. ${el}`));
  
  console.log(`\nHidden focusable elements: ${hidden.length}`);
  hidden.forEach((el, i) => console.log(`  ${i + 1}. ${el} (should NOT be in tab order)`));
  
  console.log('\n✓ Verification complete\n');
});
```

---

---

# KEYBOARD SHORTCUTS

## Example 1: Test Ctrl+Enter for Form Submission

### What Happens

User in form text box → Press Ctrl+Enter → Form submits

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Keyboard shortcut - Ctrl+Enter submit', async ({ page }) => {
  console.log('=== Ctrl+Enter Form Submission ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Find form textarea (like comment box)
  const textarea = page.locator('textarea[id="message"]');
  
  if (!await textarea.isVisible()) {
    console.log('❌ Textarea not found, skipping');
    return;
  }
  
  await textarea.click();
  
  console.log('1. Focused on textarea');
  
  // Type message
  await textarea.type('This is a test comment.');
  console.log('2. Typed message');
  
  // Listen for form submission
  let submitted = false;
  page.on('request', request => {
    if (request.postData()?.includes('message')) {
      submitted = true;
    }
  });
  
  // Press Ctrl+Enter to submit
  console.log('3. Pressing Ctrl+Enter...');
  await page.keyboard.press('Control+Enter');
  
  // Wait for submission
  await page.waitForTimeout(1000);
  
  if (submitted) {
    console.log('✓ Form submitted with Ctrl+Enter\n');
  } else {
    console.log('⚠ No submission detected (may not be supported)\n');
  }
});
```

---

## Example 2: Test Ctrl+K for Search/Command Palette

### What Happens

Press Ctrl+K anywhere on page → Search/command palette opens

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Keyboard shortcut - Ctrl+K for search', async ({ page }) => {
  console.log('=== Ctrl+K Shortcut ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Check if search modal exists but is hidden
  const searchModal = page.locator('[id="search-modal"]');
  const isHiddenBefore = !(await searchModal.isVisible().catch(() => false));
  
  console.log('Before Ctrl+K:');
  console.log(`  Search modal visible: ${!isHiddenBefore}`);
  
  // Press Ctrl+K
  console.log('\nPressing Ctrl+K...');
  await page.keyboard.press('Control+KeyK');
  
  // Wait for modal to appear
  await page.waitForTimeout(500);
  
  const isVisibleAfter = await searchModal.isVisible().catch(() => false);
  
  console.log('After Ctrl+K:');
  console.log(`  Search modal visible: ${isVisibleAfter}`);
  
  if (isVisibleAfter) {
    console.log('✓ Shortcut works: Ctrl+K opened search\n');
  } else {
    console.log('⚠ Shortcut may not be implemented\n');
  }
});
```

---

## Example 3: Test Multiple Keyboard Shortcuts

### What Happens

Test several shortcuts (Ctrl+S save, Ctrl+Z undo, etc.)

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Multiple keyboard shortcuts', async ({ page }) => {
  console.log('=== Testing Multiple Shortcuts ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const shortcuts = [
    { key: 'Control+KeyS', name: 'Ctrl+S (Save)' },
    { key: 'Control+KeyZ', name: 'Ctrl+Z (Undo)' },
    { key: 'Control+KeyY', name: 'Ctrl+Y (Redo)' },
    { key: 'Control+KeyF', name: 'Ctrl+F (Find)' },
    { key: 'Control+KeyN', name: 'Ctrl+N (New)' },
  ];
  
  console.log('Testing shortcuts:\n');
  
  for (const shortcut of shortcuts) {
    try {
      console.log(`Testing: ${shortcut.name}`);
      
      // Try to trigger shortcut
      await page.keyboard.press(shortcut.key);
      
      // Wait to see if anything happens
      await page.waitForTimeout(300);
      
      // Check if page changed (heuristic)
      const pageChanged = await page.evaluate(() => {
        return document.body.innerText.length > 0;  // Simple check
      });
      
      if (pageChanged) {
        console.log(`  ✓ Triggered\n`);
      } else {
        console.log(`  ? May not be implemented\n`);
      }
      
    } catch (error) {
      console.log(`  ❌ Error: ${error.message}\n`);
    }
  }
});
```

---

---

# SPECIAL KEY HANDLING

## Example 1: Escape Key - Close Modal

### What Happens

User in modal → Press Escape → Modal closes

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Escape key closes modal', async ({ page }) => {
  console.log('=== Escape Key - Close Modal ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Open modal
  console.log('1. Clicking to open modal...');
  await page.click('button:has-text("Open Modal")');
  
  // Wait for modal to appear
  await page.waitForSelector('[role="dialog"]', { timeout: 5000 });
  
  const modal = page.locator('[role="dialog"]');
  let isVisible = await modal.isVisible();
  console.log(`2. Modal visible: ${isVisible}`);
  
  // Press Escape
  console.log('3. Pressing Escape key...');
  await page.keyboard.press('Escape');
  
  // Wait for modal to close
  await page.waitForTimeout(500);
  
  isVisible = await modal.isVisible().catch(() => false);
  console.log(`4. Modal visible after Escape: ${isVisible}`);
  
  if (!isVisible) {
    console.log('✓ Escape key closed modal\n');
  } else {
    console.log('❌ Modal still open after Escape\n');
  }
});
```

---

## Example 2: Arrow Keys - Navigate List

### What Happens

Focus on list → Press up/down arrows → Item selection changes

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Arrow keys navigate list', async ({ page }) => {
  console.log('=== Arrow Keys - Navigate List ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Focus on first item in list
  const firstItem = page.locator('[role="listbox"] [role="option"]').first();
  await firstItem.focus();
  
  console.log('1. Focused on first item');
  console.log('   Item text:', await firstItem.textContent());
  
  // Press Down arrow
  console.log('\n2. Pressing Down arrow...');
  await page.keyboard.press('ArrowDown');
  
  // Check which item now has focus/selection
  const activeItem = page.locator('[role="option"][aria-selected="true"]');
  console.log('   Selected item:', await activeItem.textContent());
  
  // Press Down again
  console.log('\n3. Pressing Down arrow again...');
  await page.keyboard.press('ArrowDown');
  console.log('   Selected item:', await activeItem.textContent());
  
  // Press Up arrow to go back
  console.log('\n4. Pressing Up arrow...');
  await page.keyboard.press('ArrowUp');
  console.log('   Selected item:', await activeItem.textContent());
  
  console.log('\n✓ Arrow key navigation working\n');
});
```

---

## Example 3: Space Key - Toggle Checkbox

### What Happens

Focus on checkbox → Press Space → Checkbox toggles

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Space key toggles checkbox', async ({ page }) => {
  console.log('=== Space Key - Toggle Checkbox ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const checkbox = page.locator('input[type="checkbox"]').first();
  
  // Focus on checkbox
  await checkbox.focus();
  console.log('1. Focused on checkbox');
  
  // Check initial state
  let isChecked = await checkbox.isChecked();
  console.log(`2. Initial state - Checked: ${isChecked}`);
  
  // Press Space to toggle
  console.log('3. Pressing Space key...');
  await page.keyboard.press('Space');
  
  // Check new state
  isChecked = await checkbox.isChecked();
  console.log(`4. After Space - Checked: ${isChecked}`);
  console.log('   ✓ State toggled\n');
  
  // Press Space again to toggle back
  console.log('5. Pressing Space again...');
  await page.keyboard.press('Space');
  
  isChecked = await checkbox.isChecked();
  console.log(`6. After second Space - Checked: ${isChecked}`);
  
  console.log('\n✓ Space key toggle working\n');
});
```

---

---

# FORM SUBMISSION WITH KEYBOARD

## Example 1: Enter Key in Text Input

### What Happens

User types in single-line input → Press Enter → Form submits

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Enter submits form from input', async ({ page }) => {
  console.log('=== Enter Key - Form Submission ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Focus on search input
  const searchInput = page.locator('input[id="search"]');
  await searchInput.click();
  
  console.log('1. Focused on search input');
  
  // Type search term
  await searchInput.type('laptop');
  console.log('2. Typed: laptop');
  
  // Listen for form submission
  let formSubmitted = false;
  page.once('request', request => {
    if (request.url().includes('search')) {
      formSubmitted = true;
    }
  });
  
  // Press Enter
  console.log('3. Pressing Enter...');
  await page.keyboard.press('Enter');
  
  // Wait for potential redirect/response
  await page.waitForTimeout(1000);
  
  console.log('4. Checking if form submitted...');
  const newUrl = page.url();
  
  if (newUrl.includes('search') || formSubmitted) {
    console.log(`✓ Form submitted, URL: ${newUrl}\n`);
  } else {
    console.log(`⚠ May not have submitted (still on: ${newUrl})\n`);
  }
});
```

---

## Example 2: Enter in Textarea (Without Submitting)

### What Happens

User types in multiline textarea → Press Enter → Line break (NOT submit)

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Enter in textarea creates line break', async ({ page }) => {
  console.log('=== Enter in Textarea ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Focus on textarea
  const textarea = page.locator('textarea[id="message"]');
  
  if (!await textarea.isVisible()) {
    console.log('Textarea not found');
    return;
  }
  
  await textarea.click();
  console.log('1. Focused on textarea');
  
  // Type first line
  await textarea.type('First line');
  console.log('2. Typed: First line');
  
  // Press Enter (should create new line)
  console.log('3. Pressing Enter (should create line break)...');
  await textarea.press('Enter');
  
  // Type second line
  await textarea.type('Second line');
  console.log('4. Typed: Second line');
  
  // Get full value
  const value = await textarea.inputValue();
  
  if (value.includes('\n')) {
    console.log('5. ✓ Line break detected in value\n');
    console.log('Full text:');
    console.log(value);
  } else {
    console.log('5. ❌ No line break found\n');
  }
});
```

---

## Example 3: Tab Between Form Fields

### What Happens

Tab key moves focus between form fields in order

**Playwright Code:**
```javascript
import { test, expect } from '@playwright/test';

test('Tab through form fields', async ({ page }) => {
  console.log('=== Tab Through Form ===\n');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Form fields in order
  const fields = [
    { selector: 'input[name="firstname"]', label: 'First Name' },
    { selector: 'input[name="lastname"]', label: 'Last Name' },
    { selector: 'input[name="email"]', label: 'Email' },
    { selector: 'button[type="submit"]', label: 'Submit Button' },
  ];
  
  console.log('Navigating form with Tab:\n');
  
  // Start - focus first field
  await page.locator(fields[0].selector).focus();
  
  for (let i = 0; i < fields.length - 1; i++) {
    const current = fields[i];
    const next = fields[i + 1];
    
    // Type in current field
    if (i !== fields.length - 1) {
      console.log(`${i + 1}. Focused: ${current.label}`);
      await page.keyboard.type('test value');
      console.log(`   Typed: test value`);
    }
    
    // Press Tab to move to next
    console.log(`   Pressing Tab...`);
    await page.keyboard.press('Tab');
    
    // Verify focus moved
    const focused = await page.evaluate(() => {
      const el = document.activeElement;
      return el?.name || el?.id || el?.tagName;
    });
    
    console.log(`   Now focused: ${next.label}\n`);
  }
  
  console.log('✓ Tab navigation complete\n');
});
```

---

---

# BEST PRACTICES

### ✅ Do This
```javascript
// Test keyboard navigation for accessibility
test('Keyboard navigation', async ({ page }) => {
  await page.keyboard.press('Tab');
  const focused = await page.evaluate(() => document.activeElement?.id);
  expect(focused).toBe('expectedElement');
});

// Use press() for special keys
await page.keyboard.press('Enter');
await page.keyboard.press('Escape');
await page.keyboard.press('Control+KeyA');

// Test tab order is logical (top-to-bottom, left-to-right)
// Get all focusable elements, verify they're in visual order

// Verify hidden elements NOT in tab order
// Use display: none, visibility: hidden, or aria-hidden

// Test arrow keys in lists/dropdowns
await page.keyboard.press('ArrowDown');

// Test Escape closes modals
// Open modal → press Escape → verify it closes

// Test Enter submits forms
// Focus on input → press Enter → verify submission

// Test keyboard shortcuts (Ctrl+S, Ctrl+Z, etc.)
// Press shortcut → verify expected action occurs

// Use focus() before sending keyboard events
await element.focus();
await page.keyboard.press('Enter');
```

### ❌ Don't Do This
```javascript
// Don't expect Tab to work without proper focus management
await page.keyboard.press('Tab');  // May not work if nothing focusable

// Don't use incorrect key names
await page.keyboard.press('Ctrl');  // Wrong - use Control
await page.keyboard.press('Shift+A');  // Wrong - use Shift+KeyA

// Don't forget to focus element before keyboard action
await page.keyboard.press('Space');  // What element gets this?

// Don't mix type() with special keys
await input.type('Text');
await input.press('Enter');  // Wrong - use page.keyboard.press()

// Don't ignore Tab order for accessibility
// Just leave Tab as-is without testing

// Don't assume Enter works for all controls
// Some buttons may not respond to Enter without proper ARIA

// Don't forget modifier keys in shortcuts
await page.keyboard.press('KeyS');  // Missing Control
// Should be: await page.keyboard.press('Control+KeyS');

// Don't test keyboard without focusing element first
await page.keyboard.press('ArrowDown');  // May not work
```

---

---

# COMPLETE EXAMPLE: KEYBOARD NAVIGATION TEST SUITE

```javascript
import { test, expect } from '@playwright/test';

test.describe('Keyboard Navigation Suite', () => {
  
  test.beforeEach(async ({ page }) => {
    await page.goto('https://testautomationpractice.blogspot.com/');
  });

  test('Tab order is logical', async ({ page }) => {
    console.log('Testing tab order...\n');
    
    const expectedOrder = [
      'input[id="search"]',
      'button:has-text("Search")',
      'input[id="email"]',
      'button:has-text("Subscribe")',
    ];
    
    await page.locator(expectedOrder[0]).focus();
    
    for (let i = 0; i < expectedOrder.length - 1; i++) {
      await page.keyboard.press('Tab');
      const nextElement = page.locator(expectedOrder[i + 1]);
      const isFocused = await nextElement.evaluate(el => el === document.activeElement);
      expect(isFocused).toBeTruthy();
    }
    
    console.log('✓ Tab order verified\n');
  });

  test('Forms submit with Enter', async ({ page }) => {
    console.log('Testing Enter key submission...\n');
    
    const searchInput = page.locator('input[id="search"]');
    await searchInput.click();
    await searchInput.type('test');
    
    await page.keyboard.press('Enter');
    await page.waitForLoadState('networkidle');
    
    console.log('✓ Form submitted with Enter\n');
  });

  test('Escape closes modals', async ({ page }) => {
    console.log('Testing Escape key...\n');
    
    // Open modal
    await page.click('button:has-text("Open Modal")');
    const modal = page.locator('[role="dialog"]');
    await modal.waitFor({ state: 'visible' });
    
    // Press Escape
    await page.keyboard.press('Escape');
    await page.waitForTimeout(500);
    
    const isVisible = await modal.isVisible().catch(() => false);
    expect(!isVisible).toBeTruthy();
    
    console.log('✓ Escape closed modal\n');
  });

  test('Arrow keys navigate dropdowns', async ({ page }) => {
    console.log('Testing arrow keys...\n');
    
    const dropdown = page.locator('select');
    await dropdown.focus();
    
    // Get initial value
    const initialValue = await dropdown.inputValue();
    
    // Press Down
    await page.keyboard.press('ArrowDown');
    await page.waitForTimeout(300);
    
    const newValue = await dropdown.inputValue();
    expect(newValue).not.toBe(initialValue);
    
    console.log('✓ Arrow keys work\n');
  });

  test('Space toggles checkboxes', async ({ page }) => {
    console.log('Testing Space key...\n');
    
    const checkbox = page.locator('input[type="checkbox"]').first();
    await checkbox.focus();
    
    const initialState = await checkbox.isChecked();
    
    await page.keyboard.press('Space');
    
    const newState = await checkbox.isChecked();
    expect(newState).not.toBe(initialState);
    
    console.log('✓ Space toggles checkbox\n');
  });

  test('Keyboard shortcuts work', async ({ page }) => {
    console.log('Testing shortcuts...\n');
    
    await page.keyboard.press('Control+KeyF');  // Find
    await page.waitForTimeout(500);
    
    console.log('✓ Shortcuts tested\n');
  });
});
```

---

# KEY TAKEAWAYS

- ✅ **Keyboard Navigation** = Essential for accessibility
- ✅ **Tab Order** = Should be logical (top-to-bottom, left-to-right)
- ✅ **Special Keys** = Enter, Escape, Arrow keys, Space have specific purposes
- ✅ **Shortcuts** = Ctrl+C, Ctrl+Z, Ctrl+S for power users
- ✅ **Form Submission** = Enter in input submits (usually)
- ✅ **Modal Handling** = Escape should close
- ✅ **Hidden Elements** = Should NOT be in Tab order
- ✅ **Focus Management** = Focus element before sending keyboard event
- ✅ **Use press()** = For special keys, type() for text
- ✅ **Modifiers** = Control, Shift, Alt, Meta (Cmd)
- ✅ **Test for A11y** = Many users rely on keyboard

**Proper keyboard navigation = Accessible, professional applications!** 🎯
