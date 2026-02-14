# Playwright Dialogs & Popups

## What are Dialogs & Popups?

**Dialogs** are small windows that appear on top of the webpage asking for user interaction.

**Popups** are browser windows or new pages that open from the current page.

Think of it like: **Someone tapping you on the shoulder while you're working** - You stop and deal with them before continuing.

```
Webpage runs → Dialog appears → Test must handle it → Test continues
```

---

## Why Handle Dialogs?

By default, when a dialog appears:
- ❌ Test doesn't know what to do
- ❌ Test gets stuck/fails
- ❌ Dialog blocks the page

You must tell Playwright how to handle dialogs:
- ✅ Accept or dismiss them
- ✅ Read the message
- ✅ Send text (for prompts)
- ✅ Continue test

---

## Three Types of Dialogs

### 1. **Alert Dialog**
- Shows message
- Has only "OK" button
- Press OK to dismiss

### 2. **Confirm Dialog**
- Shows message
- Has "OK" and "Cancel" buttons
- Choose which to click

### 3. **Prompt Dialog**
- Shows message
- Has text input field
- Enter text and click OK/Cancel

---

---

# ALERT DIALOGS

## What is Alert Dialog?

**Alert** = Simple message box with only "OK" button.

Think of it like: **Someone telling you important news, you say "I understand" and move on**

---

## How to Handle Alerts

### Step 1: Listen for Dialog

```javascript
// Before action that triggers alert
page.once('dialog', dialog => {
  console.log('Dialog appeared:', dialog.message());
  // Handle it here
});
```

### Step 2: Perform Action

```javascript
await page.click('button'); // This triggers alert
```

### Step 3: Handle Dialog

```javascript
dialog.accept(); // Click OK
```

---

## Example 1: Simple Alert

### What Happens

Click button → Alert appears → Accept alert → Continue test

**HTML:**
```html
<button onclick="alert('Welcome to our site!')">Show Alert</button>
```

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Handle simple alert', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Listen for dialog BEFORE it appears
  page.once('dialog', dialog => {
    console.log('Alert message:', dialog.message());
    dialog.accept(); // Click OK
  });
  
  // Click button that shows alert
  await page.click('button:has-text("Show Alert")');
  
  console.log('✓ Alert handled');
});
```

**Explanation:**
- `page.once('dialog')` = Listen for first dialog
- `dialog.message()` = Get text from alert
- `dialog.accept()` = Click OK button
- Alert disappears, test continues

---

## Example 2: Alert with Verification

### What Happens

Click button → Alert shows → Verify message → Accept → Continue

**Playwright Code:**
```javascript
test('Verify alert message', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Store dialog data
  let alertMessage = '';
  
  page.once('dialog', dialog => {
    alertMessage = dialog.message();
    dialog.accept();
  });
  
  await page.click('button:has-text("Show Alert")');
  
  // Verify the message
  console.log('Alert said:', alertMessage);
  
  if (alertMessage.includes('Welcome')) {
    console.log('✓ Correct alert message');
  }
});
```

**Explanation:**
- Capture dialog message
- Verify it contains expected text
- Then accept
- Useful for testing exact messages

---

## Example 3: Multiple Alerts

### What Happens

Click button → Multiple alerts appear one after another → Handle each

**Playwright Code:**
```javascript
test('Handle multiple alerts', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Handle ALL dialogs (not just first one)
  page.on('dialog', dialog => {
    console.log('Alert:', dialog.message());
    dialog.accept();
  });
  
  // Click button that shows multiple alerts
  await page.click('button:has-text("Show Multiple Alerts")');
  
  // Wait a moment for all alerts to be processed
  await page.waitForTimeout(1000);
  
  console.log('✓ All alerts handled');
});
```

**Explanation:**
- `page.on('dialog')` = Listen for ALL dialogs
- Not `page.once()` which only handles first
- Each dialog auto-accepted
- More robust than handling one at a time

---

---

# CONFIRM DIALOGS

## What is Confirm Dialog?

**Confirm** = Message with "OK" and "Cancel" buttons.

Think of it like: **Someone asking "Are you sure?" and you say YES or NO**

---

## How to Handle Confirms

### Accept Confirm (Click OK)

```javascript
page.once('dialog', dialog => {
  dialog.accept();
});
```

### Dismiss Confirm (Click Cancel)

```javascript
page.once('dialog', dialog => {
  dialog.dismiss();
});
```

---

## Example 1: Accept Confirm

### What Happens

Click delete → Confirm "Are you sure?" → Click OK → Item deleted

**HTML:**
```html
<button onclick="if(confirm('Delete this item?')) { deleteItem(); }">
  Delete Item
</button>
```

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Confirm dialog - accept', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Listen for confirm and accept
  page.once('dialog', dialog => {
    console.log('Confirm message:', dialog.message());
    dialog.accept(); // Click OK
  });
  
  // Click delete button
  await page.click('button:has-text("Delete Item")');
  
  // Verify item was deleted
  const itemExists = await page.locator('.item-1').isVisible();
  console.log('Item deleted:', !itemExists);
});
```

**Explanation:**
- `dialog.accept()` = Click OK
- Item gets deleted
- Test verifies deletion

---

## Example 2: Dismiss Confirm

### What Happens

Click delete → Confirm "Are you sure?" → Click Cancel → Item NOT deleted

**Playwright Code:**
```javascript
test('Confirm dialog - dismiss', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Listen for confirm and dismiss (cancel)
  page.once('dialog', dialog => {
    console.log('Confirm message:', dialog.message());
    dialog.dismiss(); // Click Cancel
  });
  
  // Click delete button
  await page.click('button:has-text("Delete Item")');
  
  // Verify item still exists (not deleted)
  const itemExists = await page.locator('.item-1').isVisible();
  console.log('Item still exists:', itemExists);
});
```

**Explanation:**
- `dialog.dismiss()` = Click Cancel
- Item NOT deleted
- Test verifies item still there

---

## Example 3: Different Actions Based on Dialog Type

### What Happens

Different dialogs need different handling - accept/dismiss conditionally

**Playwright Code:**
```javascript
test('Handle different confirms', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  page.on('dialog', dialog => {
    const message = dialog.message();
    
    if (message.includes('Delete')) {
      // Accept delete confirmation
      dialog.accept();
      console.log('✓ Delete accepted');
    } else if (message.includes('Logout')) {
      // Dismiss logout confirmation
      dialog.dismiss();
      console.log('✓ Logout cancelled');
    } else {
      // Default: accept all others
      dialog.accept();
    }
  });
  
  await page.click('button:has-text("Delete")');
  await page.click('button:has-text("Logout")');
  
  console.log('✓ All dialogs handled correctly');
});
```

**Explanation:**
- Check message text
- Make decision based on content
- Handle each type differently
- Use `page.on()` to handle multiple

---

---

# PROMPT DIALOGS

## What is Prompt Dialog?

**Prompt** = Dialog with message + text input field + OK/Cancel buttons.

Think of it like: **Someone asking "What's your name?" and you type answer**

---

## How to Handle Prompts

### Accept with Text

```javascript
page.once('dialog', dialog => {
  dialog.accept('John'); // Type text and click OK
});
```

### Accept Without Text

```javascript
page.once('dialog', dialog => {
  dialog.accept(); // Click OK (empty text)
});
```

### Dismiss (Cancel)

```javascript
page.once('dialog', dialog => {
  dialog.dismiss(); // Click Cancel
});
```

---

## Example 1: Prompt with Input

### What Happens

Click button → Prompt asks for name → Enter name → Click OK → Name used

**HTML:**
```html
<button onclick="
  const name = prompt('What is your name?');
  if (name) greetUser(name);
">
  Get Name
</button>
```

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Prompt dialog - enter text', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Listen for prompt and send text
  page.once('dialog', dialog => {
    console.log('Prompt asks:', dialog.message());
    dialog.accept('John Doe'); // Type name and click OK
  });
  
  // Click button that shows prompt
  await page.click('button:has-text("Get Name")');
  
  // Verify name was used
  const greeting = await page.locator('.greeting').textContent();
  console.log('Greeting:', greeting);
  console.log('✓ Name entered and used');
});
```

**Explanation:**
- `dialog.accept('text')` = Type text and click OK
- Page receives the name
- Test continues

---

## Example 2: Prompt with Default Value

### What Happens

Prompt shows default text → Clear it → Enter new text → Click OK

**HTML:**
```html
<button onclick="
  const email = prompt('Email:', 'default@example.com');
">
  Get Email
</button>
```

**Playwright Code:**
```javascript
test('Prompt with default value', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  page.once('dialog', dialog => {
    console.log('Default value shown');
    // Send new value (replaces default)
    dialog.accept('user@example.com');
  });
  
  await page.click('button:has-text("Get Email")');
  
  console.log('✓ New email submitted');
});
```

**Explanation:**
- Prompt shows `default@example.com`
- We send `user@example.com`
- New value is used

---

## Example 3: Dismiss Prompt (Cancel)

### What Happens

Prompt asks for input → Click Cancel → Nothing submitted

**Playwright Code:**
```javascript
test('Prompt - cancel', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  page.once('dialog', dialog => {
    console.log('Prompt message:', dialog.message());
    dialog.dismiss(); // Click Cancel
  });
  
  await page.click('button:has-text("Get Email")');
  
  // Verify nothing was submitted
  const email = await page.locator('.user-email').textContent();
  console.log('Email:', email); // Should be empty or default
  console.log('✓ Prompt cancelled');
});
```

**Explanation:**
- `dialog.dismiss()` = Click Cancel
- Input not submitted
- Page continues without new value

---

---

# POPUP WINDOWS

## What are Popup Windows?

**Popups** are new browser windows or tabs that open (usually via `window.open()`).

Think of it like: **New window opens while current window stays open**

---

## How to Handle Popups

### Listen for New Page

```javascript
// Wait for popup to open
const popup = await page.waitForEvent('popup');
```

### Use Popup Like Normal Page

```javascript
await popup.fill('input', 'text');
await popup.click('button');
```

### Close Popup

```javascript
await popup.close();
```

---

## Example 1: Click Link That Opens Popup

### What Happens

Click link → New window opens → Do stuff in popup → Close it

**HTML:**
```html
<a href="javascript:window.open('page2.html', 'popup')">
  Open Popup
</a>
```

**Playwright Code:**
```javascript
import { test } from '@playwright/test';

test('Handle popup window', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Wait for popup to open
  const popupPromise = page.waitForEvent('popup');
  
  // Click link that opens popup
  await page.click('a:has-text("Open Popup")');
  
  // Get popup page
  const popup = await popupPromise;
  
  console.log('✓ Popup opened');
  console.log('Popup URL:', popup.url());
  
  // Do stuff in popup
  const title = await popup.title();
  console.log('Popup title:', title);
  
  // Close popup
  await popup.close();
  
  console.log('✓ Popup closed');
});
```

**Explanation:**
- `page.waitForEvent('popup')` = Wait for new window
- `popupPromise` = Gets the popup page
- Can now use popup like any other page
- `popup.close()` = Close the window

---

## Example 2: Interact with Popup

### What Happens

Click link → Popup opens → Fill form in popup → Submit → Close

**Playwright Code:**
```javascript
test('Interact with popup form', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Wait for popup
  const popupPromise = page.waitForEvent('popup');
  
  // Trigger popup
  await page.click('a:has-text("Open Form Popup")');
  
  const popup = await popupPromise;
  console.log('✓ Form popup opened');
  
  // Fill form in popup
  await popup.fill('input[name="username"]', 'testuser');
  await popup.fill('input[name="password"]', 'passwd123');
  
  // Submit form
  await popup.click('button:has-text("Submit")');
  
  // Wait for response
  await popup.waitForLoadState('networkidle');
  
  // Verify submission
  const message = await popup.locator('.confirmation').textContent();
  console.log('Popup response:', message);
  
  // Close popup
  await popup.close();
  
  console.log('✓ Form submitted and popup closed');
});
```

**Explanation:**
- Get popup page from promise
- Fill form in popup (use `popup.` not `page.`)
- Click submit in popup
- Verify result in popup
- Close when done

---

## Example 3: Multiple Popups

### What Happens

Click button → Multiple popups open → Handle each one → Close all

**Playwright Code:**
```javascript
test('Handle multiple popups', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const popups = [];
  
  // Listen for all popups
  page.on('popup', popup => {
    console.log('Popup opened:', popup.url());
    popups.push(popup);
  });
  
  // Click button that opens multiple popups
  await page.click('button:has-text("Open All Popups")');
  
  // Wait for all popups
  await page.waitForTimeout(1000);
  
  // Do something with each popup
  for (let i = 0; i < popups.length; i++) {
    const popup = popups[i];
    console.log(`Popup ${i + 1}:`, popup.url());
    
    // Interact with popup
    const title = await popup.title();
    console.log(`Title: ${title}`);
    
    // Close it
    await popup.close();
  }
  
  console.log(`✓ All ${popups.length} popups handled`);
});
```

**Explanation:**
- `page.on('popup')` = Listen for ALL popups
- Store each popup in array
- Loop through and handle each
- Close each popup

---

## Example 4: Check Popup Content

### What Happens

Popup opens → Read content → Verify → Close

**Playwright Code:**
```javascript
test('Verify popup content', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  const popupPromise = page.waitForEvent('popup');
  
  await page.click('a:has-text("Open Popup")');
  
  const popup = await popupPromise;
  
  // Get popup content
  const heading = await popup.locator('h1').textContent();
  const description = await popup.locator('p').textContent();
  
  console.log('Popup heading:', heading);
  console.log('Popup description:', description);
  
  // Verify content
  if (heading.includes('Welcome')) {
    console.log('✓ Correct heading');
  }
  
  if (description.includes('information')) {
    console.log('✓ Correct description');
  }
  
  await popup.close();
});
```

**Explanation:**
- Read content from popup like normal page
- Verify it matches expectations
- Close when done

---

---

# COMBINATION: DIALOGS + POPUPS

## Example: Dialog in Popup

### What Happens

Popup opens → Dialog appears in popup → Handle dialog → Close popup

**Playwright Code:**
```javascript
test('Popup with dialog inside', async ({ page }) => {
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Wait for popup
  const popupPromise = page.waitForEvent('popup');
  
  await page.click('a:has-text("Open Popup with Dialog")');
  
  const popup = await popupPromise;
  console.log('✓ Popup opened');
  
  // Listen for dialog in popup
  popup.once('dialog', dialog => {
    console.log('Dialog in popup:', dialog.message());
    dialog.accept();
  });
  
  // This action triggers dialog in popup
  await popup.click('button:has-text("Show Alert")');
  
  // Verify after dialog
  const result = await popup.locator('.result').textContent();
  console.log('Result:', result);
  
  // Close popup
  await popup.close();
  
  console.log('✓ Dialog handled and popup closed');
});
```

**Explanation:**
- Get popup page
- Listen for dialog on POPUP (popup.once, not page.once)
- Handle dialog like normal
- Close popup

---

---

# COMPLETE EXAMPLES

## Complete Example 1: E-Commerce Checkout

**Scenario:** Complete checkout with confirm dialog

```javascript
test('Ecommerce checkout with confirm', async ({ page }) => {
  console.log('--- Starting checkout test ---');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Add item to cart
  console.log('Adding item to cart...');
  await page.click('button:has-text("Add to Cart")');
  
  // Go to checkout
  console.log('Going to checkout...');
  await page.click('a:has-text("Checkout")');
  
  // Fill shipping info
  console.log('Filling shipping info...');
  await page.fill('input[name="address"]', '123 Main St');
  
  // Click Place Order - shows confirm dialog
  console.log('Placing order...');
  
  page.once('dialog', dialog => {
    const message = dialog.message();
    console.log('Confirm dialog:', message);
    
    if (message.includes('Confirm order')) {
      dialog.accept(); // Yes, place order
    }
  });
  
  await page.click('button:has-text("Place Order")');
  
  // Wait for confirmation page
  await page.waitForLoadState('networkidle');
  
  // Verify order placed
  const orderNumber = await page.locator('.order-number').textContent();
  console.log('✓ Order placed:', orderNumber);
  
  console.log('--- Checkout complete ---');
});
```

---

## Complete Example 2: User Registration with Popup

**Scenario:** Register user, popup opens, enter details, confirm

```javascript
test('User registration with popup', async ({ page }) => {
  console.log('--- Starting registration test ---');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Click Register button
  console.log('Opening registration popup...');
  const popupPromise = page.waitForEvent('popup');
  
  await page.click('button:has-text("Register")');
  
  const popup = await popupPromise;
  console.log('✓ Registration form opened in popup');
  
  // Setup dialog handler in popup (for confirmation)
  popup.once('dialog', dialog => {
    console.log('Confirmation:', dialog.message());
    dialog.accept();
  });
  
  // Fill registration form
  await popup.fill('input[name="username"]', 'testuser123');
  await popup.fill('input[name="email"]', 'test@example.com');
  await popup.fill('input[name="password"]', 'SecurePass123!');
  
  console.log('Form filled, submitting...');
  await popup.click('button:has-text("Register")');
  
  // Wait for confirmation
  await popup.waitForLoadState('networkidle');
  
  // Check success
  const success = await popup.locator('.success-message').isVisible();
  
  if (success) {
    console.log('✓ Registration successful');
  }
  
  // Close popup
  await popup.close();
  console.log('✓ Popup closed');
  
  console.log('--- Registration complete ---');
});
```

---

## Complete Example 3: Complex Dialog Handling

**Scenario:** Multiple dialogs, different types, handle each appropriately

```javascript
test('Complex dialog workflow', async ({ page }) => {
  console.log('--- Starting complex dialog test ---');
  
  await page.goto('https://testautomationpractice.blogspot.com/');
  
  // Handle ALL dialogs that appear
  page.on('dialog', dialog => {
    const type = dialog.type(); // 'alert', 'confirm', 'prompt', 'beforeunload'
    const message = dialog.message();
    
    console.log(`${type}: ${message}`);
    
    if (type === 'alert') {
      dialog.accept();
    } else if (type === 'confirm') {
      if (message.includes('Delete')) {
        dialog.dismiss(); // Don't delete
      } else {
        dialog.accept();
      }
    } else if (type === 'prompt') {
      dialog.accept('AutoFilled');
    }
  });
  
  // Trigger various dialogs
  console.log('Triggering dialogs...');
  await page.click('button:has-text("Show Alert")');
  await page.waitForTimeout(500);
  
  await page.click('button:has-text("Show Confirm")');
  await page.waitForTimeout(500);
  
  await page.click('button:has-text("Show Prompt")');
  await page.waitForTimeout(500);
  
  console.log('✓ All dialogs handled');
  console.log('--- Test complete ---');
});
```

---

---

# DIALOG PROPERTIES & METHODS

## Dialog Methods

```javascript
page.on('dialog', dialog => {
  // Get information
  dialog.type();           // 'alert', 'confirm', 'prompt', 'beforeunload'
  dialog.message();        // The dialog message text
  
  // Handle it
  dialog.accept('text');   // Click OK (with optional text for prompt)
  dialog.dismiss();        // Click Cancel
});
```

---

# BEST PRACTICES

### ✅ Do This
```javascript
// Set up handler BEFORE action
page.once('dialog', dialog => {
  dialog.accept();
});
await page.click('button'); // Action that triggers dialog

// Use page.on() for multiple dialogs
page.on('dialog', dialog => {
  dialog.accept();
});

// Check dialog type/message
page.once('dialog', dialog => {
  if (dialog.type() === 'confirm') {
    dialog.accept();
  }
});
```

### ❌ Don't Do This
```javascript
// Wrong: Handler after action (too late!)
await page.click('button'); // Dialog appears now!
page.once('dialog', dialog => {
  dialog.accept(); // Never runs
});

// Wrong: Forgetting to handle dialog
await page.click('button'); // Dialog appears, test hangs!

// Wrong: Using page.on() once
page.once('dialog', dialog => { // Only handles first!
  dialog.accept();
});
await page.click('button1');
await page.click('button2'); // Second dialog not handled
```

---

# SUMMARY TABLE

| Scenario | Use | Method |
|----------|-----|--------|
| **Single alert** | `page.once()` | `dialog.accept()` |
| **Single confirm** | `page.once()` | `dialog.accept()` or `dialog.dismiss()` |
| **Single prompt** | `page.once()` | `dialog.accept('text')` |
| **Multiple dialogs** | `page.on()` | Check type, then handle |
| **Popup window** | `waitForEvent('popup')` | Use popup like normal page |
| **Dialog in popup** | `popup.once('dialog')` | Handle on popup, not page |

---

# KEY TAKEAWAYS

- ✅ Setup dialog handler **BEFORE** action
- ✅ Use `page.once()` for single dialog
- ✅ Use `page.on()` for multiple dialogs
- ✅ Check `dialog.type()` for conditional handling
- ✅ Use `dialog.accept()` to click OK
- ✅ Use `dialog.dismiss()` to click Cancel
- ✅ Use `dialog.accept('text')` for prompts
- ✅ Popups are new pages - use `waitForEvent('popup')`
- ✅ Dialog in popup? Handle on popup, not page
- ✅ Close popups when done

**Proper dialog/popup handling = Smooth, automated tests!** 🎯
