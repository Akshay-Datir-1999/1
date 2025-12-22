# Playwright Interview Questions & Answers - Complete Guide

## Table of Contents
1. [Basics & Installation](#basics--installation)
2. [Execution & Test Control](#execution--test-control)
3. [Assertions, Reports & Utilities](#assertions-reports--utilities)
4. [Browser & Context Handling](#browser--context-handling)
5. [Advanced Automation Concepts](#advanced-automation-concepts)
6. [Framework, Architecture & Best Practices](#framework-architecture--best-practices)
7. [Stability, Debugging & Mobile](#stability-debugging--mobile)
8. [Framework Design & Real-World Usage](#framework-design--real-world-usage)

---

## Basics & Installation

### What is Playwright? Why Playwright?

**Theory:**
Playwright is a modern end-to-end testing framework developed by Microsoft. It's designed specifically for testing modern web applications. Think of it as a tool that controls browsers programmatically - like having a robot that can click buttons, fill forms, and verify content on websites.

**Why choose Playwright:**
- **Single API for multiple browsers**: One code works for Chrome, Firefox, and Safari
- **Auto-waiting**: Automatically waits for elements to be ready (no more sleep statements)
- **Fast execution**: Tests run in parallel by default
- **Powerful tooling**: Built-in debugging, screenshots, videos, and trace viewing
- **Modern architecture**: Handles SPAs, PWAs, and modern web frameworks well
- **No flakiness**: Reliable element detection and smart waiting mechanisms

**Real-world analogy:** If Selenium is like driving a manual car (you control everything), Playwright is like a self-driving car (it handles most complexity automatically).

```javascript
// Simple example showing Playwright's clean syntax
const { test, expect } = require('@playwright/test');

test('my first test', async ({ page }) => {
  await page.goto('https://example.com');
  await expect(page).toHaveTitle(/Example/);
  await page.click('text=More information');
});
```

---

### 1. How do you install Playwright and browsers?

**Theory:**
Playwright installation involves two steps: installing the Node.js package and downloading browser binaries. The browsers are specific versions bundled with Playwright to ensure consistency across different machines.

**Installation process:**
1. Install the npm package (the Playwright library)
2. Install browser binaries (Chromium, Firefox, WebKit)

```bash
# Method 1: Fresh installation (recommended for new projects)
npm init playwright@latest
# This creates a project structure with config and example tests

# Method 2: Add to existing project
npm install -D @playwright/test
npx playwright install

# Install specific browsers only
npx playwright install chromium
npx playwright install firefox webkit

# Install with dependencies (for CI/CD)
npx playwright install --with-deps
```

**What happens behind the scenes:**
- Browser binaries are downloaded to `~/.cache/ms-playwright/` on Linux/Mac
- Each browser is about 100-300 MB
- Browsers are versioned and isolated from your system browsers

---

### 2. How do you run tests in different ways in Playwright?

**Theory:**
Playwright provides flexible test execution options. You can run all tests, specific tests, in different browsers, with different levels of parallelization, and in debug mode.

```bash
# Run all tests
npx playwright test

# Run single test file
npx playwright test tests/login.spec.js

# Run tests in specific folder
npx playwright test tests/authentication/

# Run tests matching a pattern
npx playwright test login

# Run with test name grep
npx playwright test -g "should login"

# Run in headed mode (see the browser)
npx playwright test --headed

# Run in specific browser
npx playwright test --project=chromium
npx playwright test --project=firefox
npx playwright test --project=webkit

# Run with specific number of workers
npx playwright test --workers=1  # sequential
npx playwright test --workers=4  # 4 parallel threads

# Debug mode with Playwright Inspector
npx playwright test --debug

# Run in UI mode (interactive)
npx playwright test --ui

# Run specific test by line number
npx playwright test login.spec.js:25
```

---

### 3. How do you generate reports in Playwright?

**Theory:**
Playwright has built-in reporting capabilities. Reports help you visualize test results, see screenshots/videos of failures, and track test execution history. The HTML reporter is the most commonly used as it provides an interactive interface.

**Report types:**
- **HTML**: Interactive web-based report (default)
- **JSON**: Machine-readable format for CI/CD integration
- **JUnit**: Standard format for Jenkins and other CI tools
- **List**: Simple console output
- **Dot**: Minimal console output

```javascript
// playwright.config.js
module.exports = {
  reporter: [
    ['html', { open: 'never', outputFolder: 'playwright-report' }],
    ['json', { outputFile: 'test-results.json' }],
    ['junit', { outputFile: 'junit-results.xml' }],
    ['list']
  ]
};
```

```bash
# After test execution, open HTML report
npx playwright show-report

# Reports are automatically generated in playwright-report folder
# You can also generate custom reports during tests
```

---

### 4. What is `async` and `await` in Playwright?

**Theory:**
`async` and `await` are JavaScript features for handling asynchronous operations. In Playwright, almost everything is asynchronous because browser operations take time - loading pages, clicking buttons, waiting for elements.

**Understanding async/await:**
- **async**: Marks a function as asynchronous, returns a Promise
- **await**: Pauses execution until the Promise resolves
- **Without await**: Code continues immediately (can cause race conditions)

**Real-world analogy:** Ordering food at a restaurant:
- **Without await**: "I want pizza" (you walk away immediately, don't wait)
- **With await**: "I want pizza" (you wait until it's ready before leaving)

```javascript
// Wrong - without await (will fail)
test('wrong way', async ({ page }) => {
  page.goto('https://example.com'); // doesn't wait!
  page.click('button'); // might fail - page not loaded yet
});

// Correct - with await
test('correct way', async ({ page }) => {
  await page.goto('https://example.com'); // waits for page load
  await page.click('button'); // waits for click to complete
  
  const title = await page.title(); // waits and gets result
  console.log(title);
});

// Multiple operations in sequence
test('sequential operations', async ({ page }) => {
  await page.goto('https://example.com');
  await page.fill('#username', 'testuser');
  await page.fill('#password', 'password123');
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL(/dashboard/);
});
```

---

### 5. What is a Playwright fixture?

**Theory:**
Fixtures are a way to set up and tear down test environments. They're like helpers that provide ready-to-use objects to your tests. The `{ page }` you see in every test is actually a fixture.

**Why fixtures are useful:**
- **Automatic setup/cleanup**: No need to manually create and close browsers
- **Reusability**: Define once, use in all tests
- **Isolation**: Each test gets fresh fixtures
- **Dependency injection**: Tests declare what they need

```javascript
// Using built-in fixtures
test('using page fixture', async ({ page }) => {
  // 'page' is automatically created before test
  await page.goto('https://example.com');
  // 'page' is automatically cleaned up after test
});

test('using multiple fixtures', async ({ page, context, browser }) => {
  // page - single page fixture
  // context - browser context fixture
  // browser - browser instance fixture
});

// Creating custom fixtures
const { test: base } = require('@playwright/test');

const test = base.extend({
  // Custom fixture for logged-in page
  authenticatedPage: async ({ page }, use) => {
    // Setup
    await page.goto('https://example.com/login');
    await page.fill('#username', 'testuser');
    await page.fill('#password', 'pass123');
    await page.click('button[type="submit"]');
    await page.waitForURL('**/dashboard');
    
    // Provide to test
    await use(page);
    
    // Teardown (runs after test)
    await page.goto('https://example.com/logout');
  }
});

// Use custom fixture
test('access protected page', async ({ authenticatedPage }) => {
  await authenticatedPage.goto('https://example.com/profile');
  await expect(authenticatedPage.locator('h1')).toHaveText('My Profile');
});
```

---

### 6. Difference between `npm` and `npx`?

**Theory:**
Both are Node.js tools, but they serve different purposes:

**npm (Node Package Manager):**
- Installs and manages packages
- Adds packages to node_modules folder
- Used for installing dependencies

**npx (Node Package eXecute):**
- Executes packages without permanent installation
- Can run packages that aren't installed globally
- Used for running CLI tools

**Real-world analogy:**
- **npm**: Like buying a tool and keeping it in your toolbox
- **npx**: Like borrowing a tool when you need it

```bash
# npm - Installation
npm install playwright          # installs locally
npm install -g playwright       # installs globally

# npx - Execution
npx playwright test            # runs without global install
npx playwright codegen         # runs code generator

# Why npx is useful:
# 1. No global pollution
npx create-react-app my-app    # don't need create-react-app installed

# 2. Always uses latest version
npx @playwright/test --version

# 3. Runs local node_modules binaries
npx playwright test
# Instead of: ./node_modules/.bin/playwright test
```

---

### 7. What are the benefits of cross-browser testing?

**Theory:**
Cross-browser testing means running the same tests across different browsers to ensure consistent behavior. Different browsers have different rendering engines and JavaScript implementations.

**Key benefits:**

1. **User experience consistency**: 70% Chrome, 15% Safari, 10% Firefox, 5% Edge users - all should have same experience

2. **CSS rendering differences**: 
   - Flexbox behaves slightly different in Safari
   - Grid layouts might have spacing issues in Firefox
   - Font rendering varies across browsers

3. **JavaScript compatibility**:
   - Modern JS features might not work in older browser versions
   - APIs like WebRTC behave differently

4. **Business impact**:
   - Don't lose customers due to browser-specific bugs
   - Identify browser-specific issues before production
   - Ensure accessibility across platforms

5. **Real examples I've seen**:
   - Date picker worked in Chrome but broke in Safari
   - Modal dialog scrolling issue only in Firefox
   - Payment form validation different in Edge

```javascript
// Same test runs in all browsers automatically
test('checkout flow', async ({ page }) => {
  await page.goto('https://store.com');
  await page.click('text=Add to Cart');
  await page.click('text=Checkout');
  await page.fill('#card-number', '4242424242424242');
  await page.click('text=Pay Now');
  await expect(page.locator('.success')).toBeVisible();
});
// This test will catch if payment works in Chrome but fails in Safari
```

---

### 8. How do you perform cross-browser testing in Playwright?

**Theory:**
Playwright makes cross-browser testing simple through its project configuration. You define different browser projects, and Playwright runs your tests in all of them.

```javascript
// playwright.config.js
const { devices } = require('@playwright/test');

module.exports = {
  projects: [
    // Desktop browsers
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] }
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] }
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] }
    },
    
    // Mobile browsers
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] }
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] }
    },
    
    // Tablets
    {
      name: 'iPad',
      use: { ...devices['iPad Pro'] }
    }
  ]
};
```

```bash
# Run in all configured browsers
npx playwright test

# Run in specific browser
npx playwright test --project=chromium
npx playwright test --project=firefox
npx playwright test --project=webkit

# Run in multiple specific browsers
npx playwright test --project=chromium --project=firefox
```

```javascript
// Browser-specific test logic
test('feature test', async ({ page, browserName }) => {
  await page.goto('https://example.com');
  
  if (browserName === 'webkit') {
    // Safari-specific handling
    await page.click('button', { force: true });
  } else {
    await page.click('button');
  }
});
```

---

### 9. How do you execute tests in a local browser?

**Theory:**
By default, Playwright runs in headless mode (no visible browser window). For debugging or watching tests, you can run in headed mode where you actually see the browser.

```bash
# Headed mode - see the browser
npx playwright test --headed

# Headed with slow motion (easier to watch)
npx playwright test --headed --slow-mo=1000

# UI mode (interactive test runner)
npx playwright test --ui

# Debug mode (step through tests)
npx playwright test --debug
```

```javascript
// In playwright.config.js
use: {
  headless: false,  // always run headed
  slowMo: 500,      // slow down by 500ms per action
  viewport: { width: 1280, height: 720 }
}

// Or programmatically in test
const { chromium } = require('playwright');

test('manual browser control', async () => {
  const browser = await chromium.launch({
    headless: false,
    slowMo: 1000
  });
  
  const page = await browser.newPage();
  await page.goto('https://example.com');
  await browser.close();
});
```

---

## Execution & Test Control

### 1. How do you open the browser with and without fixture?

**Theory:**
Fixtures provide automatic browser management (recommended), but you can also manually control browser lifecycle for advanced scenarios.

**With Fixture (Recommended):**
- Automatic setup and cleanup
- Consistent across tests
- Less code
- Parallel execution safe

**Without Fixture (Manual):**
- Full control over browser lifecycle
- Useful for complex scenarios
- Need to handle cleanup manually
- Can cause memory leaks if not careful

```javascript
// Method 1: With fixture (recommended)
test('using fixture', async ({ page }) => {
  // page is ready to use
  await page.goto('https://example.com');
  await page.click('button');
  // automatic cleanup after test
});

// Method 2: Without fixture (manual control)
const { chromium } = require('playwright');

test('manual browser control', async () => {
  // Launch browser
  const browser = await chromium.launch({ headless: false });
  
  // Create context
  const context = await browser.newContext({
    viewport: { width: 1920, height: 1080 },
    userAgent: 'Custom User Agent'
  });
  
  // Create page
  const page = await context.newPage();
  
  // Do testing
  await page.goto('https://example.com');
  await page.click('button');
  
  // Manual cleanup (important!)
  await browser.close();
});

// Method 3: Mix of both
test('custom setup with fixture', async ({ browser }) => {
  // Use browser fixture but create custom context
  const context = await browser.newContext({
    permissions: ['geolocation'],
    geolocation: { latitude: 59.95, longitude: 30.31667 }
  });
  
  const page = await context.newPage();
  await page.goto('https://maps.google.com');
  
  await context.close();
});
```

---

### 2. How do you run failed test cases?

**Theory:**
Playwright tracks test failures and provides several ways to re-run only the tests that failed. This is crucial for debugging and CI/CD pipelines where you want to quickly verify fixes.

```bash
# Run only failed tests from last run
npx playwright test --last-failed

# This creates .last-run.json file that tracks failures
```

```javascript
// Configure automatic retries in playwright.config.js
module.exports = {
  // Retry failed tests automatically
  retries: 2,  // will retry 2 times if test fails
  
  // Different retry count for CI
  retries: process.env.CI ? 2 : 0
};

// Test will run up to 3 times total (1 original + 2 retries)
test('flaky test', async ({ page }) => {
  await page.goto('https://example.com');
  // If this fails, Playwright automatically retries
  await expect(page.locator('.dynamic-content')).toBeVisible();
});
```

```javascript
// Conditional retry for specific tests
test('important test', async ({ page }) => {
  test.info().annotations.push({ type: 'issue', description: 'flaky-test-123' });
  
  await page.goto('https://example.com');
});

// Manual retry logic
test('custom retry logic', async ({ page }) => {
  let attempts = 0;
  const maxAttempts = 3;
  
  while (attempts < maxAttempts) {
    try {
      await page.goto('https://example.com');
      await expect(page.locator('.element')).toBeVisible({ timeout: 5000 });
      break; // success
    } catch (error) {
      attempts++;
      if (attempts === maxAttempts) throw error;
      await page.reload();
    }
  }
});
```

---

### 3. How do you run only a specific test from the test suite?

**Theory:**
When debugging or developing, you often want to run just one test instead of the entire suite. Playwright provides multiple ways to focus on specific tests.

```javascript
// Method 1: test.only() - only this test runs
test.only('this will run', async ({ page }) => {
  await page.goto('https://example.com');
  await page.click('button');
});

test('this will skip', async ({ page }) => {
  // This test won't run
});

test('this will also skip', async ({ page }) => {
  // This test won't run either
});

// Method 2: describe.only() - only tests in this suite run
test.describe.only('Login Tests', () => {
  test('valid login', async ({ page }) => {
    // This runs
  });
  
  test('invalid login', async ({ page }) => {
    // This runs
  });
});

test.describe('Signup Tests', () => {
  test('signup', async ({ page }) => {
    // This doesn't run
  });
});
```

```bash
# Method 3: Command line with grep
npx playwright test -g "login"          # runs tests matching "login"
npx playwright test -g "should login"   # runs tests with exact phrase

# Method 4: Run specific file
npx playwright test tests/login.spec.js

# Method 5: Run test at specific line
npx playwright test login.spec.js:42    # runs test at line 42

# Method 6: Run by test name pattern
npx playwright test --grep "login" --grep-invert "signup"
# Runs tests matching "login" but not "signup"
```

---

### 4. How do you skip a test based on browsers?

**Theory:**
Some features work differently across browsers, or certain tests are only relevant for specific browsers. Playwright lets you conditionally skip tests based on browser type.

```javascript
// Method 1: Skip specific browser
test('video feature', async ({ page, browserName }) => {
  test.skip(browserName === 'webkit', 'Video API not supported in Safari');
  
  await page.goto('https://example.com/video');
  // test continues only in Chromium and Firefox
});

// Method 2: Skip multiple browsers
test('payment test', async ({ page, browserName }) => {
  test.skip(
    browserName === 'webkit' || browserName === 'firefox',
    'Payment integration works only in Chrome'
  );
  
  await page.goto('https://example.com/checkout');
});

// Method 3: Run only in specific browser
test('chrome-only feature', async ({ page, browserName }) => {
  test.skip(browserName !== 'chromium', 'Chrome-specific test');
  
  await page.goto('https://example.com');
});

// Method 4: Annotate for reporting
test('flaky in firefox', async ({ page, browserName }) => {
  test.fixme(browserName === 'firefox', 'Known issue #1234');
  
  await page.goto('https://example.com');
});

// Method 5: Conditional test suite
test.describe('Chrome Extensions', () => {
  test.skip(({ browserName }) => browserName !== 'chromium');
  
  test('extension test 1', async ({ page }) => {
    // Only runs in Chromium
  });
  
  test('extension test 2', async ({ page }) => {
    // Only runs in Chromium
  });
});

// Method 6: Platform-based skip
test('windows only', async ({ page }) => {
  test.skip(process.platform !== 'win32', 'Windows-specific test');
  
  await page.goto('https://example.com');
});
```

---

### 5. How do you pass and use locators in Playwright?

**Theory:**
Locators are Playwright's way of finding elements on a page. They're smart - they auto-wait and auto-retry. Playwright recommends user-facing locators (role, text, label) over CSS/XPath.

**Locator hierarchy (best to worst):**
1. Role-based (accessible, resilient)
2. Label-based (user-friendly)
3. Text-based (intuitive)
4. Test ID (stable, requires setup)
5. CSS/XPath (fragile, avoid if possible)

```javascript
test('all locator types', async ({ page }) => {
  await page.goto('https://example.com');
  
  // 1. Role-based (BEST - accessible)
  await page.getByRole('button', { name: 'Submit' }).click();
  await page.getByRole('textbox', { name: 'Email' }).fill('test@test.com');
  await page.getByRole('checkbox', { name: 'I agree' }).check();
  await page.getByRole('link', { name: 'Learn More' }).click();
  
  // 2. Label-based (GOOD - user perspective)
  await page.getByLabel('Username').fill('john');
  await page.getByLabel('Password').fill('secret');
  
  // 3. Placeholder
  await page.getByPlaceholder('Enter your email').fill('test@test.com');
  
  // 4. Text-based (GOOD for unique text)
  await page.getByText('Welcome back!').click();
  await page.getByText(/welcome/i).click(); // regex
  
  // 5. Test ID (STABLE - requires data-testid)
  await page.getByTestId('submit-button').click();
  // HTML: <button data-testid="submit-button">Submit</button>
  
  // 6. CSS selector (when needed)
  await page.locator('.submit-btn').click();
  await page.locator('#login-form').waitFor();
  await page.locator('button[type="submit"]').click();
  
  // 7. XPath (last resort)
  await page.locator('//button[@type="submit"]').click();
});

// Combining and chaining locators
test('complex locators', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Find button within specific section
  await page.locator('.checkout-section').getByRole('button', { name: 'Pay' }).click();
  
  // Find nth element
  await page.locator('button').nth(2).click(); // 3rd button
  await page.locator('button').first().click();
  await page.locator('button').last().click();
  
  // Filter locators
  await page.getByRole('button').filter({ hasText: 'Submit' }).click();
  await page.locator('div').filter({ has: page.locator('button') }).count();
});

// Passing locators as parameters (reusable)
async function clickButton(page, locator) {
  await page.locator(locator).click();
}

test('reusable locators', async ({ page }) => {
  await page.goto('https://example.com');
  
  const submitButton = page.getByRole('button', { name: 'Submit' });
  const emailField = page.getByLabel('Email');
  
  await emailField.fill('test@test.com');
  await submitButton.click();
});
```

---

### 6. How do you open a new tab or window?

**Theory:**
Modern web apps often open links in new tabs. Playwright handles this through the `context.waitForEvent('page')` pattern, which listens for new pages being created.

```javascript
test('open new tab - Method 1', async ({ context, page }) => {
  await page.goto('https://example.com');
  
  // Listen for new page before clicking
  const [newPage] = await Promise.all([
    context.waitForEvent('page'), // wait for new page event
    page.click('a[target="_blank"]') // action that opens new tab
  ]);
  
  // Wait for new page to load
  await newPage.waitForLoadState();
  console.log('New tab title:', await newPage.title());
  
  // Interact with new tab
  await newPage.click('button');
  
  // Both tabs are now accessible
  await page.click('#something'); // original tab
  await newPage.click('#other'); // new tab
});

// Method 2: Multiple new tabs
test('handle multiple new tabs', async ({ context, page }) => {
  await page.goto('https://example.com');
  
  // Open first new tab
  const [newPage1] = await Promise.all([
    context.waitForEvent('page'),
    page.click('a.link1')
  ]);
  
  // Open second new tab
  const [newPage2] = await Promise.all([
    context.waitForEvent('page'),
    page.click('a.link2')
  ]);
  
  // Now you have 3 pages
  console.log('Original:', await page.title());
  console.log('Tab 1:', await newPage1.title());
  console.log('Tab 2:', await newPage2.title());
});

// Method 3: Programmatically open new tab
test('create new tab manually', async ({ context }) => {
  const page1 = await context.newPage();
  await page1.goto('https://example.com');
  
  const page2 = await context.newPage();
  await page2.goto('https://google.com');
  
  // Work with both pages
  await page1.click('button');
  await page2.fill('input[name="q"]', 'search term');
});

// Method 4: Handle popup windows
test('handle popup', async ({ context, page }) => {
  await page.goto('https://example.com');
  
  const [popup] = await Promise.all([
    context.waitForEvent('page'),
    page.click('button#open-popup')
  ]);
  
  await popup.waitForLoadState();
  await expect(popup).toHaveTitle(/Popup/);
  await popup.close();
});
```

---

### 7. How do you handle multiple tabs or windows?

**Theory:**
When dealing with multiple tabs, you need to keep track of page objects and switch between them as needed. Each tab is a separate Page object.

```javascript
test('comprehensive multi-tab handling', async ({ context, page }) => {
  await page.goto('https://example.com');
  
  // Open multiple tabs
  const [page2] = await Promise.all([
    context.waitForEvent('page'),
    page.click('a[href="/about"]')
  ]);
  
  const [page3] = await Promise.all([
    context.waitForEvent('page'),
    page.click('a[href="/contact"]')
  ]);
  
  // Get all open pages
  const allPages = context.pages();
  console.log(`Total tabs open: ${allPages.length}`); // 3
  
  // Interact with specific tabs
  await page.click('#home-button');      // original tab
  await page2.click('#about-button');    // second tab
  await page3.fill('#name', 'John');     // third tab
  
  // Switch between tabs by title
  for (const p of allPages) {
    const title = await p.title();
    if (title.includes('Contact')) {
      await p.click('button[type="submit"]');
    }
  }
  
  // Close specific tab
  await page2.close();
  
  // Close all except original
  for (const p of context.pages()) {
    if (p !== page) {
      await p.close();
    }
  }
});

// Real-world example: OAuth flow
test('oauth in popup', async ({ context, page }) => {
  await page.goto('https://myapp.com/login');
  await page.click('button:text("Login with Google")');
  
  // OAuth opens in popup
  const [oauthPage] = await Promise.all([
    context.waitForEvent('page'),
    page.click('button:text("Login with Google")')
  ]);
  
  // Handle OAuth in popup
  await oauthPage.fill('#email', 'user@gmail.com');
  await oauthPage.fill('#password', 'password');
  await oauthPage.click('#login-button');
  
  // Wait for popup to close (OAuth redirects back)
  await oauthPage.waitForEvent('close');
  
  // Back to original page, now logged in
  await expect(page.locator('.user-profile')).toBeVisible();
});
```

---

### 8. How do you handle alerts in Playwright?

**Theory:**
JavaScript alerts, confirms, and prompts are dialog boxes that block user interaction. Playwright handles these through dialog events. You must set up the listener BEFORE triggering the action that shows the dialog.

```javascript
// Basic alert handling
test('handle simple alert', async ({ page }) => {
  // Set up dialog handler BEFORE triggering alert
  page.on('dialog', async dialog => {
    console.log('Alert message:', dialog.message());
    await dialog.accept(); // clicks OK
  });
  
  await page.goto('https://example.com');
  await page.click('button#show-alert');
  // Alert is automatically handled
});

// Handle confirm dialog
test('handle confirm', async ({ page }) => {
  page.on('dialog', async dialog => {
    console.log('Type:', dialog.type()); // 'confirm'
    console.log('Message:', dialog.message());
    
    if (dialog.message().includes('Are you sure')) {
      await dialog.accept(); // clicks OK
    } else {
      await dialog.dismiss(); // clicks Cancel
    }
  });
  
  await page.goto('https://example.com');
  await page.click('button#delete-item');
});

// Handle prompt dialog
test('handle prompt', async ({ page }) => {
  page.on('dialog', async dialog => {
    console.log('Type:', dialog.type()); // 'prompt'
    await dialog.accept('My Input Text'); // enters text and clicks OK
  });
  
  await page.goto('https://example.com');
  await page.click('button#ask-name');
});

// Handle multiple dialogs
test('multiple dialogs', async ({ page }) => {
  let dialogCount = 0;
  
  page.on('dialog', async dialog => {
    dialogCount++;
    console.log(`Dialog ${dialogCount}: ${dialog.message()}`);
    await dialog.accept();
  });
  
  await page.goto('https://example.com');
  await page.click('button#show-multiple-alerts');
});

// One-time dialog handler
test('handle dialog once', async ({ page }) => {
  page.once('dialog', async dialog => {
    await dialog.accept(); // handles only the first dialog
  });
  
  await page.goto('https://example.com');
  await page.click('button#show-alert');
});

// Remove dialog handler
test('remove dialog handler', async ({ page }) => {
  const handler = async dialog => {
    await dialog.accept();
  };
  
  page.on('dialog', handler);
  await page.click('button#alert1');
  
  // Remove handler
  page.off('dialog', handler);
  // Next dialog won't be handled
});
```

---

### 9. How do you maximize the browser in Playwright?

**Theory:**
Unlike traditional automation tools, Playwright focuses on viewport size rather than window maximization. This is because consistent viewport size ensures reproducible tests. However, you can maximize if needed.

```javascript
// Method 1: Set viewport to null (uses available screen size)
// In playwright.config.js
use: {
  viewport: null,
  launchOptions: {
    args: ['--start-maximized']
  }
}

// Method 2: Set specific viewport size
use: {
  viewport: { width: 1920, height: 1080 }
}

// Method 3: Programmatically
test('maximize browser', async ({ browser }) => {
  const context = await browser.newContext({
    viewport: null,
    screen: { width: 1920, height: 1080 }
  });
  
  const page = await context.newPage();
  await page.goto('https://example.com');
});

// Method 4: Set viewport for specific test
test('full screen test', async ({ page }) => {
  await page.setViewportSize({ width: 1920, height: 1080 });
  await page.goto('https://example.com');
});

// Method 5: Get current viewport
test('check viewport', async ({ page }) => {
  const viewport = page.viewportSize();
  console.log(`Width: ${viewport.width}, Height: ${viewport.height}`);
});
```

**Why viewport matters more than maximization:**
- Consistent screenshots across different machines
- Responsive design testing at specific breakpoints
- Reproducible test results
- Better for CI/CD environments

---

### 10. How do you run tests in parallel?

**Theory:**
Playwright runs tests in parallel by default using multiple worker processes. This dramatically reduces execution time. Each worker gets its own browser instance, ensuring test isolation.

```javascript
// playwright.config.js
module.exports = {
  // Number of parallel workers
  workers: 4, // runs 4 tests simultaneously
  
  // Dynamic workers based on CPU
  workers: process.env.CI ? 2 : undefined, // 2 in CI, auto locally
  
  // Fully parallel mode
  fullyParallel: true, // even tests in same file run parallel
};

// Sequential execution for specific file
test.describe.configure({ mode: 'serial' });

test.describe('Login Flow', () => {
  test.describe.configure({ mode: 'serial' });
  
  test('step 1: open page', async ({ page }) => {
    await page.goto('https://example.com');
  });
  
  test('step 2: login', async ({ page }) => {
    // runs after step 1
    await page.fill('#username', 'user');
  });
});

// Parallel in describe
test.describe.configure({ mode: 'parallel' });

test.describe('Independent Tests', () => {
  test('test 1', async ({ page }) => {
    // runs in parallel with test 2
  });
  
  test('test 2', async ({ page }) => {
    // runs in parallel with test 1
  });
});
```

```bash
# Command line control
npx playwright test --workers=1     # sequential (1 worker)
npx playwright test --workers=4     # 4 parallel workers
npx playwright test --workers=50%   # 50% of CPU cores

# Fully parallel
npx playwright test --fully-parallel
```

**Real-world example:**
```javascript
// 100 tests with 1 worker: ~50 minutes
// 100 tests with 4 workers: ~15 minutes
// 100 tests with 10 workers: ~8 minutes

test.describe('E-commerce Tests', () => {
  test('search product', async ({ page }) => {});
  test('add to cart', async ({ page }) => {});
  test('checkout', async ({ page }) => {});
  test('payment', async ({ page }) => {});
  // All run simultaneously in different browser instances
});
```

---

## Assertions, Reports & Utilities

### 1. Difference between `fill()` and `type()`?

**Theory:**
Both methods enter text into input fields, but they work differently under the hood. Understanding when to use each is important for test speed and reliability.

**fill():**
- Clears existing content and sets value instantly
- Doesn't trigger individual keystroke events
- Much faster
- Use for most form filling scenarios

**type():**
- Simulates actual keyboard typing character by character
- Triggers keydown, keypress, keyup events for each character
- Slower but more realistic
- Use when testing keyboard event handlers

```javascript
test('fill vs type comparison', async ({ page }) => {
  await page.goto('https://example.com');
  
  // fill() - FAST (recommended for most cases)
  await page.locator('#email').fill('test@example.com');
  // Instantly sets the value
  
  // type() - SLOW (use when needed)
  await page.locator('#search').type('playwright testing', { delay: 100 });
  // Types each character with 100ms delay
});

// When to use type()
test('autocomplete search', async ({ page }) => {
  await page.goto('https://example.com/search');
  
  // Use type() because autocomplete needs keystroke events
  await page.locator('#search').type('play');
  
  // Wait for autocomplete suggestions
  await expect(page.locator('.autocomplete-item')).toHaveCount(5);
  
  // Click suggestion
  await page.click('.autocomplete-item:first-child');
});

// When to use fill()
test('login form', async ({ page }) => {
  await page.goto('https://example.com/login');
  
  // Use fill() for simple form filling (much faster)
  await page.fill('#username', 'testuser');
  await page.fill('#password', 'password123');
  await page.click('button[type="submit"]');
});

// Real-world comparison
test('performance comparison', async ({ page }) => {
  await page.goto('https://example.com');
  
  // fill() - completes in ~50ms
  const startFill = Date.now();
  await page.fill('#input1', 'This is a long text string');
  console.log(`fill() took: ${Date.now() - startFill}ms`);
  
  // type() with 50ms delay - completes in ~1250ms (25 chars * 50ms)
  const startType = Date.now();
  await page.type('#input2', 'This is a long text string', { delay: 50 });
  console.log(`type() took: ${Date.now() - startType}ms`);
});
```

---

### 2. How do you perform hard and soft assertions?

**Theory:**
Assertions verify expected outcomes. Hard assertions stop test execution on failure, while soft assertions collect all failures and report them at the end.

**Hard Assertions:**
- Test stops immediately on failure
- Good for critical checks
- Default behavior

**Soft Assertions:**
- Test continues even on failure
- Collects all failures
- Reports all issues at end
- Good for UI validation where you want to check multiple elements

```javascript
// Hard Assertions (default)
test('hard assertions', async ({ page }) => {
  await page.goto('https://example.com');
  
  // If this fails, test stops here
  await expect(page).toHaveTitle('Example Domain');
  
  // This won't execute if above fails
  await expect(page.locator('h1')).toBeVisible();
  
  // This won't execute either
  await expect(page.locator('p')).toContainText('domain');
});

// Soft Assertions
test('soft assertions', async ({ page }) => {
  await page.goto('https://example.com');
  
  // All these will execute even if some fail
  await expect.soft(page).toHaveTitle('Wrong Title'); // FAIL but continues
  await expect.soft(page.locator('h1')).toBeVisible(); // PASS
  await expect.soft(page.locator('p')).toContainText('wrong text'); // FAIL but continues
  await expect.soft(page.locator('.footer')).toBeVisible(); // PASS
  
  // Test completes and reports all failures at end
});

// Real-world example: UI validation
test('dashboard validation', async ({ page }) => {
  await page.goto('https://example.com/dashboard');
  
  // Check multiple UI elements
  await expect.soft(page.locator('.header')).toBeVisible();
  await expect.soft(page.locator('.sidebar')).toBeVisible();
  await expect.soft(page.locator('.main-content')).toBeVisible();
  await expect.soft(page.locator('.footer')).toBeVisible();
  await expect.soft(page.locator('.user-profile')).toContainText('John Doe');
  await expect.soft(page.locator('.notification-count')).toHaveText('5');
  
  // Even if some fail, you'll see all UI issues in one test run
});

// Mixed approach
test('mixed assertions', async ({ page }) => {
  await page.goto('https://example.com/checkout');
  
  // Critical check - must pass
  await expect(page).toHaveURL(/checkout/);
  
  // Soft checks for UI elements
  await expect.soft(page.locator('.cart-items')).toBeVisible();
  await expect.soft(page.locator('.total-price')).toContainText(');
  await expect.soft(page.locator('.payment-methods')).toBeVisible();
  
  // Another critical check
  await expect(page.locator('button.checkout-btn')).toBeEnabled();
});
```

---

### 3. How do you open the HTML report?

**Theory:**
Playwright's HTML report is an interactive web interface that shows test results, screenshots, videos, and traces. It's automatically generated after test runs.

```bash
# After running tests, reports are in playwright-report folder
npx playwright test

# Open the HTML report
npx playwright show-report

# Open report from specific location
npx playwright show-report ./custom-report-folder

# Report opens in browser at http://localhost:9323
```

```javascript
// Configure in playwright.config.js
module.exports = {
  reporter: [
    ['html', {
      outputFolder: 'my-report',
      open: 'never', // 'always', 'never', 'on-failure'
      host: 'localhost',
      port: 9223
    }]
  ]
};

// Multiple reporters
reporter: [
  ['html'],                    // HTML report
  ['json', { outputFile: 'results.json' }],
  ['junit', { outputFile: 'junit.xml' }],
  ['list']                     // Console output
];
```

**What's in the HTML report:**
- Test execution summary
- Pass/fail counts
- Execution time
- Screenshots of failures
- Video recordings
- Trace files
- Error messages and stack traces
- Ability to filter by browser, status, file

---

### 4. How do you generate authentication cookies?

**Theory:**
Authentication state (cookies, localStorage, sessionStorage) can be saved and reused across tests. This saves time by avoiding repeated login operations.

```javascript
// Step 1: Perform login and save state
test('save authentication state', async ({ page }) => {
  await page.goto('https://example.com/login');
  await page.fill('#username', 'testuser');
  await page.fill('#password', 'Password123!');
  await page.click('button[type="submit"]');
  
  // Wait for login to complete
  await page.waitForURL('**/dashboard');
  
  // Save authentication state (cookies + localStorage)
  await page.context().storageState({ path: 'auth.json' });
});

// Step 2: Reuse authentication in other tests
// playwright.config.js
use: {
  storageState: 'auth.json' // all tests use this auth state
}

// Or per test
test('access protected page', async ({ browser }) => {
  const context = await browser.newContext({
    storageState: 'auth.json'
  });
  
  const page = await context.newPage();
  await page.goto('https://example.com/profile');
  // Already logged in!
});

// Advanced: Global setup for authentication
// global-setup.js
async function globalSetup() {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  
  await page.goto('https://example.com/login');
  await page.fill('#username', process.env.TEST_USERNAME);
  await page.fill('#password', process.env.TEST_PASSWORD);
  await page.click('button[type="submit"]');
  await page.waitForURL('**/dashboard');
  
  await page.context().storageState({ path: 'auth.json' });
  await browser.close();
}

module.exports = globalSetup;

// playwright.config.js
globalSetup: require.resolve('./global-setup'),
use: {
  storageState: 'auth.json'
}
```

**Real-world pattern:**
```javascript
// Create different auth states for different user roles
test.describe('Admin Tests', () => {
  test.use({ storageState: 'admin-auth.json' });
  
  test('admin dashboard', async ({ page }) => {
    await page.goto('/admin');
  });
});

test.describe('Regular User Tests', () => {
  test.use({ storageState: 'user-auth.json' });
  
  test('user profile', async ({ page }) => {
    await page.goto('/profile');
  });
});
```

---

### 5. What are ARIA options in Playwright?

**Theory:**
ARIA (Accessible Rich Internet Applications) attributes make web content accessible. Playwright's ARIA locators help you find elements the way screen readers and assistive technologies do. This makes tests more resilient and ensures accessibility.

**Benefits:**
- More resilient than CSS selectors
- Tests accessibility automatically
- User-centric (finds elements how users do)
- Less brittle than id/class selectors

```javascript
test('ARIA locators', async ({ page }) => {
  await page.goto('https://example.com');
  
  // getByRole - Most powerful and recommended
  await page.getByRole('button', { name: 'Submit' }).click();
  await page.getByRole('textbox', { name: 'Email' }).fill('test@test.com');
  await page.getByRole('checkbox', { name: 'Remember me' }).check();
  await page.getByRole('link', { name: 'Learn More' }).click();
  await page.getByRole('heading', { name: 'Welcome', level: 1 }).textContent();
  await page.getByRole('listitem').count();
  
  // Common ARIA roles
  await page.getByRole('navigation').click();
  await page.getByRole('main').isVisible();
  await page.getByRole('banner').textContent(); // header
  await page.getByRole('contentinfo').isVisible(); // footer
  await page.getByRole('search').fill('query');
  await page.getByRole('dialog').waitFor(); // modal
  await page.getByRole('alert').textContent();
  await page.getByRole('menu').click();
  await page.getByRole('menuitem', { name: 'Settings' }).click();
  await page.getByRole('tab', { name: 'Profile' }).click();
  await page.getByRole('tabpanel').isVisible();
  
  // With additional options
  await page.getByRole('button', { 
    name: 'Submit',
    exact: true // exact match
  }).click();
  
  await page.getByRole('button', {
    name: /submit/i // regex match
  }).click();
  
  await page.getByRole('textbox', {
    name: 'Email',
    disabled: false // only enabled textboxes
  }).fill('test@test.com');
});

// Real-world accessible form
test('accessible login form', async ({ page }) => {
  await page.goto('https://example.com/login');
  
  // Find by label (aria-label or <label> element)
  await page.getByLabel('Email Address').fill('user@example.com');
  await page.getByLabel('Password').fill('SecurePass123');
  await page.getByLabel('Keep me logged in').check();
  
  // Find button by accessible name
  await page.getByRole('button', { name: 'Log In' }).click();
  
  // Verify success message
  await expect(page.getByRole('alert')).toContainText('Login successful');
});

// Testing navigation
test('accessible navigation', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Find navigation by role
  const nav = page.getByRole('navigation');
  
  // Find links within navigation
  await nav.getByRole('link', { name: 'Home' }).click();
  await nav.getByRole('link', { name: 'About' }).click();
  await nav.getByRole('link', { name: 'Contact' }).click();
});
```

---

### 6. Difference between `innerHTML`, `innerText`, and `textContent`?

**Theory:**
These three properties retrieve text content from elements, but they work differently regarding HTML tags, visibility, and whitespace.

**innerHTML:**
- Returns HTML markup with tags
- Includes all HTML elements
- Use when you need the actual HTML structure

**innerText:**
- Returns visible text only
- Respects CSS styling (hidden elements excluded)
- Respects line breaks as rendered
- Closer to what user sees

**textContent:**
- Returns all text including hidden elements
- Ignores CSS styling
- Includes script and style elements
- Faster than innerText

```javascript
test('innerHTML vs innerText vs textContent', async ({ page }) => {
  await page.setContent(`
    <div id="test">
      <span>Hello</span>
      <span style="display: none;">Hidden</span>
      <script>console.log('script')</script>
      World
    </div>
  `);
  
  const element = page.locator('#test');
  
  // innerHTML - includes HTML tags
  const html = await element.innerHTML();
  console.log(html);
  // Output: "<span>Hello</span><span style="display: none;">Hidden</span><script>...</script> World"
  
  // innerText - only visible text
  const inner = await element.innerText();
  console.log(inner);
  // Output: "Hello World" (Hidden is not included)
  
  // textContent - all text including hidden
  const text = await element.textContent();
  console.log(text);
  // Output: "HelloHidden console.log('script') World"
});

// Real-world examples
test('practical usage', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Use textContent for simple text extraction (fastest)
  const title = await page.locator('h1').textContent();
  console.log(title); // "Welcome to Example"
  
  // Use innerText when you need visible text only
  const visibleText = await page.locator('.content').innerText();
  console.log(visibleText); // Only visible text
  
  // Use innerHTML when you need HTML structure
  const htmlContent = await page.locator('.article').innerHTML();
  console.log(htmlContent); // "<p>Text</p><img src='...'>"
});

// Performance comparison
test('performance', async ({ page }) => {
  await page.goto('https://example.com');
  
  // textContent is fastest
  const start1 = Date.now();
  await page.locator('body').textContent();
  console.log(`textContent: ${Date.now() - start1}ms`);
  
  // innerText is slower (calculates visibility)
  const start2 = Date.now();
  await page.locator('body').innerText();
  console.log(`innerText: ${Date.now() - start2}ms`);
  
  // innerHTML is in between
  const start3 = Date.now();
  await page.locator('body').innerHTML();
  console.log(`innerHTML: ${Date.now() - start3}ms`);
});
```

---

### 7. What exceptions have you faced in Playwright?

**Theory:**
Understanding common exceptions helps in debugging and writing robust tests. Here are the most common errors and how to handle them.

```javascript
// 1. TimeoutError - Most common
test('timeout error', async ({ page }) => {
  try {
    await page.goto('https://example.com');
    // Element not found within default 30s timeout
    await page.locator('.non-existent-element').click();
  } catch (error) {
    console.log(error.name); // "TimeoutError"
  }
  
  // Solutions:
  // a) Increase timeout
  await page.locator('.slow-element').click({ timeout: 60000 });
  
  // b) Wait for element first
  await page.waitForSelector('.dynamic-element');
  await page.click('.dynamic-element');
  
  // c) Check if element exists
  const exists = await page.locator('.element').count() > 0;
  if (exists) {
    await page.click('.element');
  }
});

// 2. Strict mode violation - Multiple elements match
test('strict mode error', async ({ page }) => {
  await page.goto('https://example.com');
  
  // If multiple buttons exist, this throws error
  // Error: strict mode violation: locator('button') resolved to 5 elements
  // await page.click('button'); // FAILS
  
  // Solutions:
  await page.locator('button').first().click(); // click first
  await page.locator('button').nth(2).click(); // click 3rd
  await page.locator('button').last().click(); // click last
  await page.locator('button:has-text("Submit")').click(); // be specific
});

// 3. Target closed - Browser/page closed unexpectedly
test('target closed error', async ({ page }) => {
  try {
    await page.goto('https://example.com');
    await page.close();
    await page.click('button'); // Error: Target closed
  } catch (error) {
    console.log(error.message); // "Target page, context or browser has been closed"
  }
  
  // Solution: Ensure proper test structure
  // Don't close page manually when using fixtures
});

// 4. Navigation timeout
test('navigation timeout', async ({ page }) => {
  try {
    // Page takes too long to load
    await page.goto('https://very-slow-site.com', { timeout: 5000 });
  } catch (error) {
    console.log('Navigation timeout');
  }
  
  // Solutions:
  await page.goto('https://slow-site.com', { 
    timeout: 60000,
    waitUntil: 'domcontentloaded' // don't wait for all resources
  });
});

// 5. Element not visible/not enabled
test('element state errors', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Element exists but not visible
  // await page.click('.hidden-button'); // Fails
  
  // Solutions:
  await page.locator('.hidden-button').click({ force: true }); // force click
  
  await page.waitForSelector('.dynamic-button', { state: 'visible' });
  await page.click('.dynamic-button');
  
  // Check if visible first
  const isVisible = await page.locator('.button').isVisible();
  if (isVisible) {
    await page.click('.button');
  }
});

// 6. Element is outside viewport
test('outside viewport error', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Element exists but outside viewport
  // await page.click('.footer-button'); // Might fail
  
  // Solutions:
  await page.locator('.footer-button').scrollIntoViewIfNeeded();
  await page.click('.footer-button');
  
  // Or force click
  await page.click('.footer-button', { force: true });
});

// 7. Network errors
test('network errors', async ({ page }) => {
  try {
    await page.goto('https://nonexistent-domain-12345.com');
  } catch (error) {
    console.log('DNS/Network error');
  }
  
  // Handle with waitForLoadState
  await page.goto('https://example.com');
  await page.waitForLoadState('networkidle');
});

// Error handling best practice
test('robust error handling', async ({ page }) => {
  await page.goto('https://example.com');
  
  try {
    await page.locator('.element').click({ timeout: 5000 });
  } catch (error) {
    if (error.name === 'TimeoutError') {
      console.log('Element not found, trying alternative');
      await page.locator('.alternative-element').click();
    } else {
      throw error; // re-throw unexpected errors
    }
  }
});
```

---

### 8. How do you take screenshots?

**Theory:**
Screenshots are crucial for debugging test failures and visual regression testing. Playwright provides flexible screenshot options for full pages, specific elements, or custom regions.

```javascript
// Basic screenshots
test('basic screenshots', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Full page screenshot
  await page.screenshot({ path: 'screenshot.png' });
  
  // Full page including scrollable content
  await page.screenshot({ 
    path: 'fullpage.png',
    fullPage: true 
  });
  
  // Element screenshot
  await page.locator('.header').screenshot({ 
    path: 'header.png' 
  });
  
  // Screenshot as buffer (no file)
  const buffer = await page.screenshot();
  console.log(buffer.length);
});

// Advanced screenshot options
test('advanced screenshots', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Custom quality (for JPEG)
  await page.screenshot({
    path: 'screenshot.jpeg',
    type: 'jpeg',
    quality: 80 // 0-100
  });
  
  // Clip specific region
  await page.screenshot({
    path: 'region.png',
    clip: {
      x: 0,
      y: 0,
      width: 500,
      height: 500
    }
  });
  
  // Hide elements before screenshot
  await page.screenshot({
    path: 'masked.png',
    mask: [page.locator('.sensitive-data')]
  });
  
  // Wait for animations to complete
  await page.screenshot({
    path: 'stable.png',
    animations: 'disabled' // or 'allow'
  });
});

// Screenshot on failure (automatic)
// playwright.config.js
use: {
  screenshot: 'only-on-failure', // 'off', 'on', 'only-on-failure'
  video: 'retain-on-failure'
}

// Programmatic screenshot on failure
test('with failure screenshot', async ({ page }, testInfo) => {
  try {
    await page.goto('https://example.com');
    await page.click('.non-existent');
  } catch (error) {
    await testInfo.attach('failure-screenshot', {
      body: await page.screenshot(),
      contentType: 'image/png'
    });
    throw error;
  }
});
```

---

### 9. How do you attach screenshots to reports?

**Theory:**
Attaching screenshots to reports helps with debugging and provides visual evidence of test execution. Playwright allows attaching various types of content to test reports.

```javascript
// Method 1: Automatic attachment on failure
// playwright.config.js
use: {
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
  trace: 'retain-on-failure'
}

// Method 2: Manual attachment
test('attach screenshot manually', async ({ page }, testInfo) => {
  await page.goto('https://example.com');
  
  // Take screenshot and attach
  const screenshot = await page.screenshot();
  await testInfo.attach('homepage', {
    body: screenshot,
    contentType: 'image/png'
  });
  
  await page.click('button');
  
  // Attach another screenshot
  await testInfo.attach('after-click', {
    body: await page.screenshot(),
    contentType: 'image/png'
  });
});

// Method 3: Attach at specific steps
test('multi-step with screenshots', async ({ page }, testInfo) => {
  await page.goto('https://example.com/login');
  await testInfo.attach('1-login-page', {
    body: await page.screenshot(),
    contentType: 'image/png'
  });
  
  await page.fill('#username', 'user');
  await page.fill('#password', 'pass');
  await testInfo.attach('2-filled-form', {
    body: await page.screenshot(),
    contentType: 'image/png'
  });
  
  await page.click('button[type="submit"]');
  await page.waitForURL('**/dashboard');
  await testInfo.attach('3-dashboard', {
    body: await page.screenshot(),
    contentType: 'image/png'
  });
});

// Method 4: Attach element screenshots
test('attach element screenshot', async ({ page }, testInfo) => {
  await page.goto('https://example.com');
  
  const header = page.locator('.header');
  await testInfo.attach('header-element', {
    body: await header.screenshot(),
    contentType: 'image/png'
  });
});

// Method 5: Attach other content types
test('attach various content', async ({ page }, testInfo) => {
  await page.goto('https://example.com');
  
  // Attach text
  await testInfo.attach('page-title', {
    body: await page.title(),
    contentType: 'text/plain'
  });
  
  // Attach JSON
  await testInfo.attach('cookies', {
    body: JSON.stringify(await page.context().cookies()),
    contentType: 'application/json'
  });
  
  // Attach HTML
  await testInfo.attach('page-content', {
    body: await page.content(),
    contentType: 'text/html'
  });
});

// Method 6: Conditional attachment
test('attach on failure only', async ({ page }, testInfo) => {
  await page.goto('https://example.com');
  
  try {
    await expect(page.locator('h1')).toHaveText('Expected');
  } catch (error) {
    // Only attach if assertion fails
    await testInfo.attach('failure-state', {
      body: await page.screenshot({ fullPage: true }),
      contentType: 'image/png'
    });
    throw error;
  }
});
```

---

### 10. How do you save screenshots to a specific path?

**Theory:**
Organizing screenshots in specific directories helps with test management, especially when dealing with multiple test suites or environments.

```javascript
// Method 1: Direct path specification
test('save to specific path', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Absolute path
  await page.screenshot({ path: '/Users/username/screenshots/test.png' });
  
  // Relative path
  await page.screenshot({ path: './test-results/screenshots/homepage.png' });
  
  // Create nested directories (automatically created)
  await page.screenshot({ 
    path: './screenshots/2024/january/test1.png' 
  });
});

// Method 2: Dynamic paths based on test info
test('dynamic screenshot path', async ({ page }, testInfo) => {
  await page.goto('https://example.com');
  
  // Create path from test name and project
  const fileName = `${testInfo.project.name}-${testInfo.title.replace(/\s+/g, '-')}.png`;
  const path = `./screenshots/${fileName}`;
  
  await page.screenshot({ path });
});

// Method 3: Timestamp-based paths
test('timestamped screenshots', async ({ page }) => {
  await page.goto('https://example.com');
  
  const timestamp = new Date().toISOString().replace(/:/g, '-');
  await page.screenshot({ 
    path: `./screenshots/test-${timestamp}.png` 
  });
});

// Method 4: Configuration-based paths
// playwright.config.js
use: {
  screenshot: {
    mode: 'only-on-failure',
    fullPage: true
  }
}

// Screenshots automatically saved to test-results folder

// Method 5: Organized by test suite
test.describe('Login Tests', () => {
  test('valid login', async ({ page }) => {
    await page.goto('https://example.com/login');
    await page.screenshot({ 
      path: './screenshots/login/valid-login.png' 
    });
  });
  
  test('invalid login', async ({ page }) => {
    await page.goto('https://example.com/login');
    await page.screenshot({ 
      path: './screenshots/login/invalid-login.png' 
    });
  });
});

// Method 6: Create directories programmatically
const fs = require('fs');
const path = require('path');

test('create directory if not exists', async ({ page }) => {
  await page.goto('https://example.com');
  
  const screenshotDir = './screenshots/custom-folder';
  if (!fs.existsSync(screenshotDir)) {
    fs.mkdirSync(screenshotDir, { recursive: true });
  }
  
  await page.screenshot({ 
    path: path.join(screenshotDir, 'test.png') 
  });
});

// Method 7: Environment-based paths
test('environment-based path', async ({ page }) => {
  await page.goto('https://example.com');
  
  const env = process.env.NODE_ENV || 'development';
  const screenshotPath = `./screenshots/${env}/test.png`;
  
  await page.screenshot({ path: screenshotPath });
});
```

---

## Browser & Context Handling

### 1. Difference between new page and new context?

**Theory:**
Understanding the relationship between Browser, Context, and Page is crucial for test isolation and performance optimization.

**Browser:** 
- The actual browser instance (Chrome, Firefox, Safari)
- Can have multiple contexts
- Heavy resource

**BrowserContext:**
- Isolated incognito-like session
- Has its own cookies, cache, localStorage
- Can have multiple pages
- Lightweight

**Page:**
- Individual tab/window
- Belongs to one context

**Real-world analogy:**
- Browser = The building
- Context = Separate apartments (isolated)
- Page = Rooms in an apartment

```javascript
// New Page - shares same context (cookies, cache)
test('new page - shared context', async ({ page, context }) => {
  await page.goto('https://example.com/login');
  await page.fill('#username', 'user');
  await page.click('button[type="submit"]');
  
  // Create new page in SAME context
  const page2 = await context.newPage();
  await page2.goto('https://example.com/profile');
  // page2 has the same cookies/session as page
  // Already logged in!
});

// New Context - completely isolated
test('new context - isolated', async ({ browser }) => {
  // First context with login
  const context1 = await browser.newContext();
  const page1 = await context1.newPage();
  await page1.goto('https://example.com/login');
  await page1.fill('#username', 'user1');
  await page1.click('button[type="submit"]');
  
  // Second context - completely separate
  const context2 = await browser.newContext();
  const page2 = await context2.newPage();
  await page2.goto('https://example.com/profile');
  // page2 NOT logged in - different context
  // Will redirect to login page
  
  await context1.close();
  await context2.close();
});

// Use cases for new page
test('new page use case - multiple tabs same user', async ({ context }) => {
  const page1 = await context.newPage();
  await page1.goto('https://example.com/product1');
  
  const page2 = await context.newPage();
  await page2.goto('https://example.com/product2');
  
  const page3 = await context.newPage();
  await page3.goto('https://example.com/cart');
  
  // All pages share same shopping cart
  const cartCount = await page3.locator('.cart-count').textContent();
});

// Use cases for new context
test('new context use case - multiple users', async ({ browser }) => {
  // User 1
  const user1Context = await browser.newContext();
  const user1Page = await user1Context.newPage();
  await user1Page.goto('https://example.com/login');
  await user1Page.fill('#username', 'user1');
  
  // User 2
  const user2Context = await browser.newContext();
  const user2Page = await user2Context.newPage();
  await user2Page.goto('https://example.com/login');
  await user2Page.fill('#username', 'user2');
  
  // Completely isolated - different users, different sessions
  
  await user1Context.close();
  await user2Context.close();
});

// Performance consideration
test('performance comparison', async ({ browser }) => {
  // Creating new page is FAST
  const context = await browser.newContext();
  const start1 = Date.now();
  const page1 = await context.newPage();
  console.log(`New page: ${Date.now() - start1}ms`); // ~10ms
  
  // Creating new context is SLOWER but still fast
  const start2 = Date.now();
  const context2 = await browser.newContext();
  console.log(`New context: ${Date.now() - start2}ms`); // ~50ms
});
```

---

### 2. How do you handle multiple elements using locators?

**Theory:**
When dealing with lists, tables, or repeated elements, Playwright provides powerful methods to handle multiple elements efficiently.

```javascript
test('handle multiple elements', async ({ page }) => {
  await page.goto('https://example.com/products');
  
  // Method 1: Count elements
  const productCount = await page.locator('.product').count();
  console.log(`Total products: ${productCount}`);
  
  // Method 2: Loop through elements
  const products = page.locator('.product');
  const count = await products.count();
  
  for (let i = 0; i < count; i++) {
    const product = products.nth(i);
    const name = await product.locator('.product-name').textContent();
    const price = await product.locator('.product-price').textContent();
    console.log(`${name}: ${price}`);
  }
  
  // Method 3: Get all text contents
  const productNames = await page.locator('.product-name').allTextContents();
  console.log(productNames); // ['Product 1', 'Product 2', 'Product 3']
  
  // Method 4: Get all inner texts
  const productPrices = await page.locator('.product-price').allInnerTexts();
  console.log(productPrices); // ['$10', '$20', '$30']
  
  // Method 5: Click all elements
  const buttons = page.locator('.add-to-cart');
  const buttonCount = await buttons.count();
  
  for (let i = 0; i < buttonCount; i++) {
    await buttons.nth(i).click();
  }
  
  // Method 6: Filter elements
  const expensiveProducts = page.locator('.product')
    .filter({ hasText: '$100' });
  await expensiveProducts.first().click();
  
  // Method 7: Find specific element in list
  const searchResult = page.locator('.product')
    .filter({ hasText: 'iPhone' });
  await searchResult.click();
});

// Real-world example: Table handling
test('handle table rows', async ({ page }) => {
  await page.goto('https://example.com/users');
  
  const rows = page.locator('table tbody tr');
  const rowCount = await rows.count();
  
  // Find row with specific user
  for (let i = 0; i < rowCount; i++) {
    const row = rows.nth(i);
    const username = await row.locator('td').nth(0).textContent();
    
    if (username === 'john.doe') {
      // Click edit button in this row
      await row.locator('button:has-text("Edit")').click();
      break;
    }
  }
});

// Real-world example: Checkboxes
test('handle multiple checkboxes', async ({ page }) => {
  await page.goto('https://example.com/settings');
  
  // Check all checkboxes
  const checkboxes = page.locator('input[type="checkbox"]');
  const count = await checkboxes.count();
  
  for (let i = 0; i < count; i++) {
    await checkboxes.nth(i).check();
  }
  
  // Or use specific checkbox
  await page.locator('input[value="notifications"]').check();
});

// Real-world example: Dropdown options
test('handle dropdown options', async ({ page }) => {
  await page.goto('https://example.com/form');
  
  // Get all options
  const options = page.locator('select#country option');
  const allOptions = await options.allTextContents();
  console.log('Available countries:', allOptions);
  
  // Select specific option
  await page.selectOption('select#country', 'USA');
});
```

---

### 3. How do you use explicit waits in Playwright?

**Theory:**
Playwright has auto-waiting built in, but sometimes you need explicit control over waiting conditions. Explicit waits ensure elements are in the correct state before interactions.

```javascript
// Built-in auto-waiting
test('auto waiting', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Playwright automatically waits for element to be:
  // - Attached to DOM
  // - Visible
  // - Stable (not animating)
  // - Enabled
  // - Receiving events
  await page.click('button'); // Auto-waits up to 30s
});

// Explicit waits
test('explicit waits', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Wait for selector to be visible
  await page.waitForSelector('.dynamic-content', { 
    state: 'visible',
    timeout: 10000 
  });
  
  // Wait for element to be hidden
  await page.waitForSelector('.loading-spinner', { 
    state: 'hidden' 
  });
  
  // Wait for element to be attached (doesn't need to be visible)
  await page.waitForSelector('.element', { 
    state: 'attached' 
  });
  
  // Wait for element to be detached (removed from DOM)
  await page.waitForSelector('.modal', { 
    state: 'detached' 
  });
});

// Wait for navigation
test('wait for navigation', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Wait for URL change
  await page.waitForURL('**/dashboard');
  await page.waitForURL(/.*dashboard/);
  
  // Wait for specific URL
  await page.waitForURL('https://example.com/profile');
  
  // Wait for load state
  await page.waitForLoadState('networkidle'); // all network connections done
  await page.waitForLoadState('domcontentloaded'); // DOM loaded
  await page.waitForLoadState('load'); // page fully loaded
});

// Wait for function
test('wait for custom condition', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Wait for custom JavaScript condition
  await page.waitForFunction(() => {
    return document.querySelectorAll('.item').length > 5;
  });
  
  // Wait with polling
  await page.waitForFunction(() => {
    return document.querySelector('.status').textContent === 'Ready';
  }, { 
    polling: 1000, // check every 1 second
    timeout: 30000 
  });
  
  // Pass arguments to function
  await page.waitForFunction(
    (minCount) => document.querySelectorAll('.item').length >= minCount,
    5 // minCount argument
  );
});

// Wait for timeout (avoid when possible)
test('wait for timeout', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Hard wait (BAD PRACTICE - use only if no alternative)
  await page.waitForTimeout(3000); // waits 3 seconds
  
  // Better alternatives:
  await page.waitForSelector('.element');
  await page.waitForLoadState('networkidle');
});

// Wait for event
test('wait for event', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Wait for console message
  const messagePromise = page.waitForEvent('console');
  await page.click('button');
  const message = await messagePromise;
  console.log(message.text());
  
  // Wait for response
  const responsePromise = page.waitForResponse('**/api/data');
  await page.click('button#load-data');
  const response = await responsePromise;
  console.log(await response.json());
  
  // Wait for request
  const requestPromise = page.waitForRequest('**/api/submit');
  await page.click('button#submit');
  const request = await requestPromise;
  console.log(request.postData());
});

// Real-world example: AJAX content
test('wait for AJAX content', async ({ page }) => {
  await page.goto('https://example.com/search');
  
  await page.fill('#search', 'playwright');
  
  // Wait for API response
  await page.waitForResponse(response =>
    response.url().includes('/api/search') && response.status() === 200
  );
  
  // Wait for results to appear
  await page.waitForSelector('.search-results .item');
  
  // Verify results
  const resultsCount = await page.locator('.search-results .item').count();
  expect(resultsCount).toBeGreaterThan(0);
});
```

---

### 4. How do you verify CSS properties (like color)?

**Theory:**
Verifying CSS properties ensures your styling is correct. This is important for visual testing and ensuring design consistency across browsers.

```javascript
test('verify CSS properties', async ({ page }) => {
  await page.goto('https://example.com');
  
  const element = page.locator('.header');
  
  // Method 1: Using evaluate
  const backgroundColor = await element.evaluate(el => {
    return window.getComputedStyle(el).backgroundColor;
  });
  expect(backgroundColor).toBe('rgb(255, 0, 0)'); // red
  
  // Method 2: Get multiple CSS properties
  const styles = await element.evaluate(el => {
    const computed = window.getComputedStyle(el);
    return {
      color: computed.color,
      fontSize: computed.fontSize,
      fontWeight: computed.fontWeight,
      backgroundColor: computed.backgroundColor,
      display: computed.display,
      padding: computed.padding
    };
  });
  
  expect(styles.color).toBe('rgb(0, 0, 0)');
  expect(styles.fontSize).toBe('16px');
  expect(styles.fontWeight).toBe('700'); // bold
});

// Verify specific CSS properties
test('detailed CSS verification', async ({ page }) => {
  await page.goto('https://example.com');
  
  const button = page.locator('button.primary');
  
  // Color (returns as rgb)
  const color = await button.evaluate(el =>
    window.getComputedStyle(el).color
  );
  expect(color).toBe('rgb(255, 255, 255)'); // white
  
  // Background color
  const bgColor = await button.evaluate(el =>
    window.getComputedStyle(el).backgroundColor
  );
  expect(bgColor).toBe('rgb(0, 123, 255)'); // blue
  
  // Font size
  const fontSize = await button.evaluate(el =>
    window.getComputedStyle(el).fontSize
  );
  expect(fontSize).toBe('14px');
  
  // Border
  const border = await button.evaluate(el =>
    window.getComputedStyle(el).border
  );
  console.log(border);
  
  // Display property
  const display = await button.evaluate(el =>
    window.getComputedStyle(el).display
  );
  expect(display).toBe('inline-block');
});

// Helper function for CSS verification
async function getCSSProperty(locator, property) {
  return await locator.evaluate(
    (el, prop) => window.getComputedStyle(el)[prop],
    property
  );
}

test('using helper function', async ({ page }) => {
  await page.goto('https://example.com');
  
  const header = page.locator('h1');
  
  const color = await getCSSProperty(header, 'color');
  const fontFamily = await getCSSProperty(header, 'fontFamily');
  const textAlign = await getCSSProperty(header, 'textAlign');
  
  expect(color).toBe('rgb(33, 37, 41)');
  expect(textAlign).toBe('center');
});

// Verify hover states
test('verify hover CSS', async ({ page }) => {
  await page.goto('https://example.com');
  
  const button = page.locator('button');
  
  // Get normal state
  const normalColor = await button.evaluate(el =>
    window.getComputedStyle(el).backgroundColor
  );
  
  // Hover
  await button.hover();
  
  // Get hover state
  const hoverColor = await button.evaluate(el =>
    window.getComputedStyle(el).backgroundColor
  );
  
  expect(normalColor).not.toBe(hoverColor);
});

// Real-world example: Theme verification
test('verify dark mode theme', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Enable dark mode
  await page.click('button#dark-mode-toggle');
  
  // Verify theme colors
  const bodyBg = await page.evaluate(() =>
    window.getComputedStyle(document.body).backgroundColor
  );
  expect(bodyBg).toBe('rgb(33, 37, 41)'); // dark background
  
  const textColor = await page.locator('p').first().evaluate(el =>
    window.getComputedStyle(el).color
  );
  expect(textColor).toBe('rgb(248, 249, 250)'); // light text
});
```

---

### 5. What is WebKit in Playwright?

**Theory:**
WebKit is the browser engine that powers Safari (Apple's browser). Testing in WebKit ensures your application works for Safari users, which is crucial for iOS and macOS platforms.

**Browser Engines:**
- **Chromium**: Chrome, Edge, Brave, Opera (Blink engine)
- **Firefox**: Firefox (Gecko engine)
- **WebKit**: Safari, iOS browsers (WebKit engine)

**Why WebKit matters:**
- Safari has ~19% global market share
- iOS requires all browsers to use WebKit
- WebKit renders and executes JavaScript differently
- Some CSS/JS features work differently

```javascript
// Running tests in WebKit
test('webkit test', async ({ page, browserName }) => {
  await page.goto('https://example.com');
  
  console.log(`Running in: ${browserName}`); // "webkit"
  
  // WebKit-specific behavior
  if (browserName === 'webkit') {
    // Handle Safari-specific quirks
    await page.click('button', { force: true });
  } else {
    await page.click('button');
  }
});

// Configure WebKit in playwright.config.js
module.exports = {
  projects: [
    {
      name: 'webkit',
      use: { 
        browserName: 'webkit',
        // Safari-specific viewport
        viewport: { width: 1280, height: 720 }
      }
    }
  ]
};

// Run only in WebKit
// npx playwright test --project=webkit

// WebKit-specific test
test('safari date picker', async ({ page, browserName }) => {
  test.skip(browserName !== 'webkit', 'Safari-specific test');
  
  await page.goto('https://example.com/form');
  await page.fill('input[type="date"]', '2024-12-25');
  // Date input behaves differently in Safari
});

// Common WebKit differences to watch for
test('webkit compatibility checks', async ({ page, browserName }) => {
  await page.goto('https://example.com');
  
  if (browserName === 'webkit') {
    // 1. Video autoplay restrictions
    await page.click('.play-button'); // Manual click needed
    
    // 2. Geolocation permissions
    // WebKit handles permissions differently
    
    // 3. Font rendering
    // Fonts may look different in WebKit
    
    // 4. Flexbox behavior
    // Some flexbox properties behave differently
  }
});
```

---

### 6. Difference between Selenium and Playwright?

**Theory:**
While both are automation frameworks, Playwright is newer and designed specifically for modern web applications with significant architectural improvements.

**Key Differences:**

| Feature | Selenium | Playwright |
|---------|----------|------------|
| Architecture | WebDriver protocol | Direct CDP/DevTools |
| Auto-waiting | Manual waits needed | Built-in auto-waiting |
| Speed | Slower | Faster (2-3x) |
| Browser support | All major browsers | Chromium, Firefox, WebKit |
| Multiple tabs | Complex | Easy |
| Network interception | Requires proxy | Built-in |
| Screenshots | Basic | Advanced |
| Trace viewer | No | Yes |
| Parallel execution | Complex setup | Built-in |
| API design | Older patterns | Modern async/await |
| Mobile testing | Appium needed | Built-in device emulation |

```javascript
// Selenium (Java example)
WebDriver driver = new ChromeDriver();
driver.get("https://example.com");
WebDriverWait wait = new WebDriverWait(driver, 10);
wait.until(ExpectedConditions.elementToBeClickable(By.id("button")));
driver.findElement(By.id("button")).click();
Thread.sleep(2000); // Explicit waits everywhere
driver.quit();

// Playwright (JavaScript) - cleaner, modern
test('playwright example', async ({ page }) => {
  await page.goto('https://example.com');
  await page.click('#button'); // Auto-waits, no explicit waits needed
  // No cleanup needed, fixtures handle it
});

// Auto-waiting comparison
// Selenium - manual waits
wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("element")));
wait.until(ExpectedConditions.elementToBeClickable(By.id("button")));

// Playwright - auto-waits
await page.click('#button'); // Automatically waits for element to be ready

// Network interception
// Selenium - needs proxy
// Complex setup with BrowserMob Proxy

// Playwright - built-in
await page.route('**/api/**', route => route.fulfill({
  status: 200,
  body: '{"data": "mocked"}'
}));

// Multiple tabs
// Selenium - complex
String mainWindow = driver.getWindowHandle();
for (String handle : driver.getWindowHandles()) {
  driver.switchTo().window(handle);
}

// Playwright - simple
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.click('a[target="_blank"]')
]);
```

**When to use Selenium:**
- Need IE support
- Existing large Selenium codebase
- Team already knows Selenium

**When to use Playwright:**
- Modern web apps (SPAs, React, Angular, Vue)
- Need faster execution
- Want better debugging tools
- Need network mocking
- Mobile testing required

---

### 7. What are browser contexts and why are they used?

**Theory:**
Browser contexts are isolated browser sessions within a single browser instance. They're like incognito windows - each context has its own cookies, localStorage, and cache, providing complete isolation.

**Why use contexts:**
- **Test isolation**: No state leakage between tests
- **Parallel testing**: Multiple isolated sessions
- **Multi-user scenarios**: Test different users simultaneously
- **Performance**: Lighter than launching new browsers
- **Session management**: Different auth states

```javascript
// Single browser, multiple contexts
test('multiple user sessions', async ({ browser }) => {
  // Admin user context
  const adminContext = await browser.newContext({
    storageState: 'admin-auth.json'
  });
  const adminPage = await adminContext.newPage();
  await adminPage.goto('https://example.com/admin');
  
  // Regular user context
  const userContext = await browser.newContext({
    storageState: 'user-auth.json'
  });
  const userPage = await userContext.newPage();
  await userPage.goto('https://example.com/dashboard');
  
  // Both sessions are completely isolated
  await adminContext.close();
  await userContext.close();
});

// Context with custom configuration
test('custom context', async ({ browser }) => {
  const context = await browser.newContext({
    viewport: { width: 1280, height: 720 },
    userAgent: 'Custom User Agent',
    locale: 'en-US',
    timez
