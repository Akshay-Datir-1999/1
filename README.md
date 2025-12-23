};
```

---

### 8. Why are browser contexts important?

**Theory:**
Browser contexts are the foundation of Playwright's test isolation strategy. They provide clean, isolated environments without the overhead of launching new browsers.

**Importance of Browser Contexts:**

1. **Test Isolation**
   - Each context has its own cookies, localStorage, sessionStorage
   - No data leakage between tests
   - Prevents flaky tests

2. **Performance**
   - Faster than launching new browsers (~50ms vs 500ms)
   - Lightweight resource usage
   - Enables efficient parallel testing

3. **Multi-user Testing**
   - Simulate multiple users simultaneously
   - Different authentication states
   - Different permissions/roles

4. **Clean State**
   - Fresh environment for each test
   - No cleanup needed
   - Predictable test behavior

```javascript
// Problem without contexts: State pollution
test('test 1', async ({ page }) => {
  await page.goto('https://example.com');
  await page.evaluate(() => {
    localStorage.setItem('flag', 'true');
  });
});

test('test 2', async ({ page }) => {
  await page.goto('https://example.com');
  const flag = await page.evaluate(() => localStorage.getItem('flag'));
  // 'flag' might still be 'true' from test 1! (flaky)
});

// Solution with contexts: Complete isolation
test('isolated test 1', async ({ browser }) => {
  const context1 = await browser.newContext();
  const page1 = await context1.newPage();
  
  await page1.goto('https://example.com');
  await page1.evaluate(() => localStorage.setItem('flag', 'true'));
  
  await context1.close();
});

test('isolated test 2', async ({ browser }) => {
  const context2 = await browser.newContext();
  const page2 = await context2.newPage();
  
  await page2.goto('https://example.com');
  const flag = await page2.evaluate(() => localStorage.getItem('flag'));
  // 'flag' is always null - completely isolated!
  
  await context2.close();
});

// Real-world: Multi-user scenario
test('admin and user simultaneously', async ({ browser }) => {
  // Admin context
  const adminContext = await browser.newContext({
    storageState: 'admin-auth.json'
  });
  const adminPage = await adminContext.newPage();
  await adminPage.goto('https://example.com/admin');
  await adminPage.click('button.create-user');
  
  // Regular user context (different session)
  const userContext = await browser.newContext({
    storageState: 'user-auth.json'
  });
  const userPage = await userContext.newPage();
  await userPage.goto('https://example.com/profile');
  
  // Both users operate independently
  // No interference between sessions
  
  await adminContext.close();
  await userContext.close();
});

// Performance benefit
test('context vs browser performance', async () => {
  // Launching browser is slow
  const start1 = Date.now();
  const browser1 = await chromium.launch();
  console.log(`Browser launch: ${Date.now() - start1}ms`); // ~500ms
  await browser1.close();
  
  // Creating context is fast
  const browser2 = await chromium.launch();
  const start2 = Date.now();
  const context = await browser2.newContext();
  console.log(`Context creation: ${Date.now() - start2}ms`); // ~50ms
  await browser2.close();
});

// Testing different user permissions
test('role-based testing', async ({ browser }) => {
  const roles = ['admin', 'editor', 'viewer'];
  
  for (const role of roles) {
    const context = await browser.newContext({
      storageState: `${role}-auth.json`
    });
    
    const page = await context.newPage();
    await page.goto('https://example.com/dashboard');
    
    // Verify role-specific UI
    if (role === 'admin') {
      await expect(page.locator('.admin-panel')).toBeVisible();
    } else {
      await expect(page.locator('.admin-panel')).not.toBeVisible();
    }
    
    await context.close();
  }
});

// Context with specific configuration
test('configured context', async ({ browser }) => {
  const context = await browser.newContext({
    // Geolocation
    geolocation: { latitude: 40.7128, longitude: -74.0060 },
    permissions: ['geolocation'],
    
    // Device emulation
    viewport: { width: 375, height: 667 },
    userAgent: 'Mobile Safari',
    
    // Network conditions
    offline: false,
    
    // Timezone
    timezoneId: 'America/New_York',
    locale: 'en-US',
    
    // Color scheme
    colorScheme: 'dark'
  });
  
  const page = await context.newPage();
  await page.goto('https://example.com');
  
  await context.close();
});
```

---

### 9. Explain auto-waiting and its benefits

**Theory:**
Auto-waiting is Playwright's intelligent waiting mechanism that automatically waits for elements to be ready before interacting with them. This eliminates most explicit waits and makes tests more reliable.

**What Auto-waiting Checks:**

1. Element is **attached** to DOM
2. Element is **visible** (not display:none or hidden)
3. Element is **stable** (not animating)
4. Element **receives events** (not covered by another element)
5. Element is **enabled** (for inputs/buttons)

```javascript
// Without auto-waiting (Selenium style)
// BAD - Lots of explicit waits needed
await page.goto('https://example.com');
await page.waitForSelector('button', { visible: true });
await page.waitForTimeout(1000); // Hope animation is done
const isEnabled = await page.isEnabled('button');
if (isEnabled) {
  await page.click('button');
}

// With auto-waiting (Playwright style)
// GOOD - Just perform the action
await page.goto('https://example.com');
await page.click('button'); // Automatically waits for all conditions!

// Auto-waiting benefits demonstration
test('auto-waiting demo', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Scenario 1: Element loads dynamically
  await page.click('.load-data-button');
  await page.click('.data-item'); // Waits for item to appear
  
  // Scenario 2: Element is animating
  await page.click('.menu-trigger');
  await page.click('.menu-item'); // Waits for animation to complete
  
  // Scenario 3: Element is covered by modal
  await page.click('.open-modal');
  await page.fill('.modal input', 'text'); // Waits for modal to be ready
  
  // Scenario 4: Button is disabled then enabled
  await page.click('.submit'); // Waits for button to be enabled
  
  // All of these "just work" - no explicit waits needed!
});

// What auto-waiting prevents
test('problems auto-waiting solves', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Problem 1: Element not yet in DOM
  // Old way: await page.waitForSelector('.element')
  // Playwright way: Auto-waits
  await page.click('.element');
  
  // Problem 2: Element hidden by CSS
  // Old way: await page.waitForSelector('.element', { state: 'visible' })
  // Playwright way: Auto-waits
  await page.click('.element');
  
  // Problem 3: Element still animating
  // Old way: await page.waitForTimeout(500)
  // Playwright way: Auto-waits for stability
  await page.click('.animated-button');
  
  // Problem 4: Element covered by overlay
  // Old way: Complex logic to wait for overlay to disappear
  // Playwright way: Auto-waits for element to receive events
  await page.click('.button-behind-overlay');
  
  // Problem 5: Input disabled
  // Old way: await page.waitForFunction(() => !document.querySelector('input').disabled)
  // Playwright way: Auto-waits for enabled state
  await page.fill('input', 'text');
});

// Configure auto-waiting timeouts
test('custom timeouts', async ({ page }) => {
  // Default timeout is 30 seconds
  
  // Set default timeout for all actions
  page.setDefaultTimeout(60000); // 60 seconds
  
  // Set timeout for specific action
  await page.click('button', { timeout: 5000 }); // 5 seconds
  
  // Set timeout in config
  // playwright.config.js
  // use: { actionTimeout: 10000 }
});

// Bypass auto-waiting when needed
test('force actions', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Force click even if element not ready (use carefully!)
  await page.click('.element', { force: true });
  
  // Force fill even if input disabled
  await page.fill('input', 'text', { force: true });
  
  // Use when:
  // - Testing error states
  // - Intentionally clicking disabled elements
  // - Known edge cases
});

// Real-world benefits
test('AJAX content loading', async ({ page }) => {
  await page.goto('https://example.com/search');
  
  // Type search query
  await page.fill('#search', 'playwright');
  
  // Click search button
  await page.click('button#search');
  
  // Click first result - auto-waits for AJAX response!
  await page.click('.search-results .result:first-child');
  
  // No explicit waits needed for:
  // - Search results to load
  // - Loading spinner to disappear
  // - Results to render
  // - Results to be clickable
});

// Comparison with explicit waits
test('explicit vs auto-waiting', async ({ page }) => {
  await page.goto('https://example.com');
  
  // ❌ OLD WAY - Lots of explicit waits
  await page.waitForLoadState('networkidle');
  await page.waitForSelector('button', { state: 'visible' });
  await page.waitForTimeout(1000);
  const button = await page.$('button');
  const isEnabled = await button.isEnabled();
  if (isEnabled) {
    await button.click();
  }
  
  // ✅ NEW WAY - Just do it
  await page.click('button');
  
  // Playwright handles all the waiting automatically!
});

// When auto-waiting might not be enough
test('additional waits needed', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Case 1: Wait for specific condition
  await page.waitForFunction(() => {
    return document.querySelectorAll('.item').length > 5;
  });
  
  // Case 2: Wait for API response
  await page.waitForResponse('**/api/data');
  
  // Case 3: Wait for URL change
  await page.waitForURL('**/dashboard');
  
  // Auto-waiting handles element states,
  // but you still need explicit waits for:
  // - Custom conditions
  // - Network events
  // - Navigation events
});
```

---

### 10. How do you handle authentication and CSRF tokens?

**Theory:**
Modern web applications use various authentication mechanisms including session tokens, JWT, OAuth, and CSRF tokens. Playwright provides multiple ways to handle these securely and efficiently.

```javascript
// Method 1: Session-based authentication
test('session auth', async ({ page }) => {
  await page.goto('https://example.com/login');
  
  // Login
  await page.fill('#username', 'testuser');
  await page.fill('#password', 'password123');
  await page.click('button[type="submit"]');
  
  await page.waitForURL('**/dashboard');
  
  // Save session state
  await page.context().storageState({ path: 'auth.json' });
});

// Reuse session
test.use({ storageState: 'auth.json' });

test('protected page', async ({ page }) => {
  await page.goto('https://example.com/profile');
  // Already authenticated
});

// Method 2: JWT Token authentication
test('JWT auth', async ({ page, request }) => {
  // Get JWT token via API
  const response = await request.post('https://example.com/api/login', {
    data: {
      username: 'testuser',
      password: 'password123'
    }
  });
  
  const { token } = await response.json();
  
  // Set token in localStorage
  await page.goto('https://example.com');
  await page.evaluate((token) => {
    localStorage.setItem('authToken', token);
  }, token);
  
  // Or set as header
  await page.goto('https://example.com', {
    extraHTTPHeaders: {
      'Authorization': `Bearer ${token}`
    }
  });
});

// Method 3: CSRF Token handling
test('CSRF token', async ({ page }) => {
  await page.goto('https://example.com/form');
  
  // Extract CSRF token from page
  const csrfToken = await page.locator('input[name="_csrf"]').inputValue();
  
  // Or from meta tag
  const csrfTokenMeta = await page.evaluate(() => {
    return document.querySelector('meta[name="csrf-token"]').content;
  });
  
  // Submit form with CSRF token
  await page.fill('#input', 'data');
  await page.evaluate((token) => {
    document.querySelector('input[name="_csrf"]').value = token;
  }, csrfToken);
  
  await page.click('button[type="submit"]');
});

// Method 4: Cookie-based authentication
test('cookie auth', async ({ context, page }) => {
  // Set authentication cookie
  await context.addCookies([
    {
      name: 'session_id',
      value: 'abc123xyz789',
      domain: 'example.com',
      path: '/',
      httpOnly: true,
      secure: true,
      sameSite: 'Lax'
    }
  ]);
  
  await page.goto('https://example.com/dashboard');
  // Authenticated via cookie
});

// Method 5: OAuth authentication
test('OAuth flow', async ({ page, context }) => {
  await page.goto('https://example.com/login');
  
  // Click OAuth login button
  const [oauthPage] = await Promise.all([
    context.waitForEvent('page'),
    page.click('button:text("Login with Google")')
  ]);
  
  // Handle OAuth provider page
  await oauthPage.fill('#email', 'test@gmail.com');
  await oauthPage.fill('#password', 'password');
  await oauthPage.click('#login-button');
  
  // Wait for redirect back to app
  await page.waitForURL('**/dashboard');
  
  // Save OAuth session
  await page.context().storageState({ path: 'oauth-auth.json' });
});

// Method 6: API-based authentication setup
// auth-setup.js
const { chromium } = require('playwright');

async function getAuthToken() {
  const browser = await chromium.launch();
  const context = await browser.newContext();
  const page = await context.newPage();
  
  // Login via API
  const response = await page.request.post('https://api.example.com/login', {
    data: {
      username: process.env.USERNAME,
      password: process.env.PASSWORD
    }
  });
  
  const { token, refreshToken } = await response.json();
  
  // Save tokens
  await context.addCookies([
    {
      name: 'auth_token',
      value: token,
      domain: 'example.com',
      path: '/'
    }
  ]);
  
  await context.storageState({ path: 'api-auth.json' });
  await browser.close();
  
  return { token, refreshToken };
}

module.exports = getAuthToken;

// Method 7: Handle CSRF in API calls
test('CSRF in API request', async ({ page, request }) => {
  await page.goto('https://example.com/form');
  
  // Get CSRF token
  const csrfToken = await page.evaluate(() => {
    return document.querySelector('meta[name="csrf-token"]').content;
  });
  
  // Make API request with CSRF token
  const response = await request.post('https://example.com/api/submit', {
    headers: {
      'X-CSRF-Token': csrfToken
    },
    data: {
      field: 'value'
    }
  });
  
  expect(response.status()).toBe(200);
});

// Method 8: Basic HTTP authentication
test('HTTP basic auth', async ({ browser }) => {
  const context = await browser.newContext({
    httpCredentials: {
      username: 'admin',
      password: 'admin123'
    }
  });
  
  const page = await context.newPage();
  await page.goto('https://example.com/admin');
  
  await context.close();
});

// Real-world: Complete authentication flow
test.describe('Complete auth flow', () => {
  let authToken;
  let csrfToken;
  
  test.beforeAll(async ({ request }) => {
    // Step 1: Get auth token via API
    const response = await request.post('https://api.example.com/auth/login', {
      data: {
        email: 'test@example.com',
        password: 'SecurePass123!'
      }
    });
    
    const data = await response.json();
    authToken = data.token;
  });
  
  test('authenticated request with CSRF', async ({ page }) => {
    // Step 2: Set auth token
    await page.goto('https://example.com');
    await page.evaluate((token) => {
      localStorage.setItem('token', token);
    }, authToken);
    
    // Step 3: Get CSRF token
    await page.goto('https://example.com/form');
    csrfToken = await page.locator('input[name="_csrf"]').inputValue();
    
    // Step 4: Submit with both tokens
    await page.fill('#data', 'test data');
    await page.evaluate((csrf) => {
      document.querySelector('input[name="_csrf"]').value = csrf;
    }, csrfToken);
    
    await page.click('button[type="submit"]');
    
    await expect(page.locator('.success-message')).toBeVisible();
  });
});

// Method 9: Token refresh handling
test('token refresh', async ({ page, request }) => {
  let accessToken = 'initial-token';
  let refreshToken = 'refresh-token';
  
  // Intercept API calls
  await page.route('**/api/**', async (route) => {
    const headers = route.request().headers();
    headers['Authorization'] = `Bearer ${accessToken}`;
    
    const response = await route.fetch({ headers });
    
    // If token expired, refresh it
    if (response.status() === 401) {
      const refreshResponse = await request.post('https://api.example.com/refresh', {
        data: { refreshToken }
      });
      
      const data = await refreshResponse.json();
      accessToken = data.accessToken;
      
      // Retry with new token
      headers['Authorization'] = `Bearer ${accessToken}`;
      await route.continue({ headers });
    } else {
      await route.fulfill({ response });
    }
  });
  
  await page.goto('https://example.com');
});
```

---

## Stability, Debugging & Mobile

### 1. How does Playwright handle dynamic elements?

**Theory:**
Dynamic elements are those that load asynchronously, animate, or change based on user interaction. Playwright's auto-waiting and smart selectors handle most dynamic scenarios automatically.

```javascript
// Scenario 1: Elements that load after page load
test('AJAX loaded elements', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Click button that loads data
  await page.click('#load-data');
  
  // Playwright automatically waits for element to appear
  await page.click('.dynamic-item'); // No explicit wait needed!
});

// Scenario 2: Elements with changing attributes
test('dynamic attributes', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Element might have changing classes or IDs
  // Use stable locators that don't depend on dynamic attributes
  
  // ❌ BAD - relies on dynamic class
  // await page.click('.item-123-active');
  
  // ✅ GOOD - uses stable attributes
  await page.click('[data-testid="item-button"]');
  await page.getByRole('button', { name: 'Submit' }).click();
  await page.getByLabel('Email').fill('test@test.com');
});

// Scenario 3: Infinite scroll / Lazy loading
test('infinite scroll', async ({ page }) => {
  await page.goto('https://example.com/feed');
  
  // Scroll and load more items
  for (let i = 0; i < 5; i++) {
    // Scroll to bottom
    await page.evaluate(() => {
      window.scrollTo(0, document.body.scrollHeight);
    });
    
    // Wait for new items to load
    await page.waitForFunction(
      (expectedCount) => {
        return document.querySelectorAll('.feed-item').length >= expectedCount;
      },
      (i + 1) * 10
    );
  }
  
  const itemCount = await page.locator('.feed-item').count();
  expect(itemCount).toBeGreaterThanOrEqual(50);
});

// Scenario 4: Elements that appear/disappear
test('toggling elements', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Open dropdown
  await page.click('.dropdown-trigger');
  
  // Wait for dropdown to be visible
  await expect(page.locator('.dropdown-menu')).toBeVisible();
  
  // Click item
  await page.click('.dropdown-menu .item');
  
  // Wait for dropdown to hide
  await expect(page.locator('.dropdown-menu')).toBeHidden();
});

// Scenario 5: Dynamic table rows
test('dynamic table', async ({ page }) => {
  await page.goto('https://example.com/users');
  
  // Add new row
  await page.click('button:text("Add User")');
  await page.fill('#name', 'John Doe');
  await page.click('button:text("Save")');
  
  // Wait for new row to appear
  await expect(page.locator('table tr:has-text("John Doe")')).toBeVisible();
  
  // Count rows
  const rowCount = await page.locator('table tbody tr').count();
  expect(rowCount).toBeGreaterThan(0);
});

// Scenario 6: Animated elements
test('animated elements', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Click to show modal with animation
  await page.click('.show-modal');
  
  // Playwright waits for animation to complete
  await page.fill('.modal input', 'text'); // Auto-waits for stability
  
  // Or explicitly wait for animations
  await page.waitForFunction(() => {
    const modal = document.querySelector('.modal');
    return modal && !modal.getAnimations().length;
  });
});

// Scenario 7: Conditional rendering
test('conditional elements', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Element might or might not exist
  const count = await page.locator('.optional-element').count();
  
  if (count > 0) {
    await page.click('.optional-element');
  } else {
    console.log('Element not present, skipping');
  }
  
  // Or use conditional visibility
  const isVisible = await page.locator('.element').isVisible();
  if (isVisible) {
    await page.click('.element');
  }
});

// Scenario 8: Real-time updates (WebSocket/SSE)
test('real-time data', async ({ page }) => {
  await page.goto('https://example.com/dashboard');
  
  // Wait for specific text to appear (from WebSocket)
  await expect(page.locator('.status')).toContainText('Connected', {
    timeout: 10000
  });
  
  // Wait for counter to reach value
  await page.waitForFunction(() => {
    const counter = document.querySelector('.live-counter');
    return counter && parseInt(counter.textContent) >= 100;
  });
});

// Scenario 9: Multiple dynamic elements
test('handle multiple dynamic elements', async ({ page }) => {
  await page.goto('https://example.com/search');
  
  await page.fill('#search', 'playwright');
  await page.click('button#search');
  
  // Wait for all results to load
  await page.waitForSelector('.search-results .item');
  
  // Get all dynamic items
  const items = page.locator('.search-results .item');
  const count = await items.count();
  
  // Interact with each
  for (let i = 0; i < count; i++) {
    const item = items.nth(i);
    const text = await item.textContent();
    console.log(`Item ${i}: ${text}`);
  }
});

// Best practices for dynamic elements
test('best practices', async ({ page }) => {
  await page.goto('https://example.com');
  
  // 1. Use data-testid for dynamic elements
  await page.click('[data-testid="dynamic-button"]');
  
  // 2. Use role-based locators (most resilient)
  await page.getByRole('button', { name: /submit/i }).click();
  
  // 3. Wait for specific conditions
  await page.waitForFunction(() => {
    return document.querySelector('.loader').style.display === 'none';
  });
  
  // 4. Use flexible text matching
  await page.getByText(/loading|loaded|complete/i).waitFor();
  
  // 5. Chain locators for specificity
  await page
    .locator('.results-container')
    .locator('.item')
    .filter({ hasText: 'specific' })
    .click();
});
```

---

### 2. What is Playwright Trace Viewer?

**Theory:**
Trace Viewer is Playwright's most powerful debugging tool. It records everything that happens during test execution - screenshots, DOM snapshots, network activity, console logs - and lets you replay and inspect the test step by step.

```javascript
// Enable tracing in playwright.config.js
module.exports = {
  use: {
    trace: 'on-first-retry', // 'on', 'off', 'retain-on-failure', 'on-first-retry'
  }
};

// Or programmatically
test('with tracing', async ({ page, context }) => {
  // Start tracing
  await context.tracing.start({
    screenshots: true,
    snapshots: true,
    sources: true
  });
  
  // Run your test
  await page.goto('https://example.com');
  await page.click('button');
  await page.fill('input', 'text');
  
  // Stop and save trace
  await context.tracing.stop({ path: 'trace.zip' });
});

// View trace
// npx playwright show-trace trace.zip

// What Trace Viewer shows:
// 1. Timeline of all actions
// 2. Screenshot at each step
// 3. DOM snapshot (can inspect elements)
// 4. Network requests/responses
// 5. Console logs
// 6. Source code
// 7. Action metadata (duration, selector used)
```

**Trace Viewer Features:**

```javascript
// Enable different trace options
await context.tracing.start({
  // Capture screenshots
  screenshots: true,
  
  // Capture DOM snapshots
  snapshots: true,
  
  // Include source code
  sources: true,
  
  // Record each action
  title: 'My Test Trace',
  
  // Capture on specific events
  snapshots: {
    mode: 'on', // 'on', 'off', 'on-failure'
  }
});

// Trace only specific test sections
test('selective tracing', async ({ page, context }) => {
  await page.goto('https://example.com');
  
  // Start tracing before critical section
  await context.tracing.start({ screenshots: true, snapshots: true });
  
  // Critical operations
  await page.click('button#important');
  await page.fill('#critical-input', 'data');
  await page.click('button#submit');
  
  // Stop tracing
  await context.tracing.stop({ path: 'critical-section-trace.zip' });
  
  // Continue test without tracing
  await page.goto('https://example.com/other');
});

// Trace with custom naming
test('named traces', async ({ page, context }, testInfo) => {
  await context.tracing.start({ screenshots: true, snapshots: true });
  
  await page.goto('https://example.com');
  await page.click('button');
  
  // Save with test-specific name
  const traceName = `trace-${testInfo.title.replace(/\s+/g, '-')}.zip`;
  await context.tracing.stop({ path: traceName });
});

// Real-world debugging workflow
test('debug failed test', async ({ page, context }) => {
  await context.tracing.start({ screenshots: true, snapshots: true });
  
  try {
    await page.goto('https://example.com');
    await page.click('.non-existent-element'); // This fails
  } catch (error) {
    // Save trace on failure
    await context.tracing.stop({ path: 'failure-trace.zip' });
    throw error;
  }
  
  await context.tracing.stop();
});

// How to use Trace Viewer:
/*
1. Run test with tracing enabled
2. Open trace file: npx playwright show-trace trace.zip
3. Trace Viewer opens in browser showing:
   - Timeline of all actions
   - Screenshots at each step
   - DOM snapshot (clickable, inspectable)
   - Network activity
   - Console logs
   - Action details (selector, duration)
4. Navigate through timeline
5. Click on any action to see details
6. Inspect DOM at that point in time
7. View network requests made
8. See console output
9. Hover over elements to see selectors
*/

// CI/CD integration
// In playwright.config.js
use: {
  trace: process.env.CI ? 'retain-on-failure' : 'on-first-retry',
  // Keeps traces only for failures in CI
}

// Trace Viewer vs Other Tools:
/*
Playwright Inspector:
- Step-by-step debugging while test runs
- Good for writing tests
- Live interaction

Trace Viewer:
- Post-mortem analysis
- See entire test execution
- Good for debugging failures
- Works with CI/CD
*/
```

---

### 3. How do you implement logging and reporting?

**Theory:**
Comprehensive logging helps debug issues and understand test execution. Playwright provides multiple ways to capture logs, errors, and test metadata.

```javascript
// Method 1: Console logging from browser
test('browser console logs', async ({ page }) => {
  // Listen to browser console
  page.on('console', msg => {
    console.log(`Browser ${msg.type()}: ${msg.text()}`);
  });
  
  // Listen to page errors
  page.on('pageerror', error => {
    console.log(`Page error: ${error.message}`);
  });
  
  // Listen to request failures
  page.  await Promise.all([
    page.waitForNavigation({ timeout: 60000 }),
    page.click('a.link')
  ]);
});

// Real-world example: Wait for AJAX
test('wait for AJAX complete', async ({ page }) => {
  await page.goto('https://example.com/search');
  
  await page.fill('#search', 'playwright');
  await page.click('button#search-btn');
  
  // Wait for API response
  await page.waitForResponse(response =>
    response.url().includes('/api/search') && response.status() === 200
  );
  
  // Wait for results to appear
  await page.waitForSelector('.search-results .item');
  
  // Wait for specific count
  await page.waitForFunction(() =>
    document.querySelectorAll('.search-results .item').length > 0
  );
});
```

---

## Framework, Architecture & Best Practices

### 1. How do you skip test cases from execution?

**Theory:**
Skipping tests is useful when tests are not ready, broken, or not applicable to certain environments or browsers.

```javascript
// Method 1: test.skip()
test.skip('not ready yet', async ({ page }) => {
  // This test won't run
});

// Method 2: Conditional skip
test('browser-specific', async ({ page, browserName }) => {
  test.skip(browserName === 'webkit', 'Not supported in Safari');
  
  await page.goto('https://example.com');
  // Test runs in Chrome and Firefox only
});

// Method 3: Skip entire describe block
test.describe.skip('Feature not ready', () => {
  test('test 1', async ({ page }) => {});
  test('test 2', async ({ page }) => {});
  // All tests in this block are skipped
});

// Method 4: Skip based on condition
test('platform specific', async ({ page }) => {
  test.skip(process.platform !== 'darwin', 'macOS only test');
  
  await page.goto('https://example.com');
});

// Method 5: Skip in specific project
test('mobile only', async ({ page }) => {
  test.skip(({ viewport }) => viewport.width > 768, 'Mobile test only');
  
  await page.goto('https://example.com');
});

// Method 6: test.fixme() - Known broken test
test.fixme('known bug #123', async ({ page }) => {
  // Marked as "fixme" in reports
  await page.goto('https://example.com');
});

// Method 7: Skip based on environment
test('production only', async ({ page }) => {
  test.skip(!process.env.PROD_URL, 'Not production environment');
  
  await page.goto(process.env.PROD_URL);
});

// Method 8: Annotations (for reporting)
test('annotated test', async ({ page }) => {
  test.info().annotations.push({ 
    type: 'issue', 
    description: 'Bug JIRA-123' 
  });
  test.skip(true, 'Waiting for bug fix');
});

// Method 9: Skip based on slow tests
test('slow test', async ({ page }) => {
  test.slow(); // Triples the timeout
  test.skip(process.env.CI && !process.env.RUN_SLOW_TESTS);
  
  await page.goto('https://example.com');
  // Long-running operations
});

// Real-world example
test.describe('Payment Tests', () => {
  test.skip(({ browserName }) => browserName === 'webkit', 
    'Payment provider not supported in Safari');
  
  test('credit card payment', async ({ page }) => {
    await page.goto('https://example.com/checkout');
  });
  
  test('paypal payment', async ({ page }) => {
    test.skip(process.env.ENV !== 'staging', 'Staging only');
    await page.goto('https://example.com/checkout');
  });
});
```

---

### 2. Explain the architecture of Playwright

**Theory:**
Playwright's architecture is fundamentally different from Selenium. It uses direct browser communication via DevTools Protocol, making it faster and more reliable.

**Architecture Components:**

```
┌─────────────────────────────────────────┐
│         Playwright Test Runner          │
│  (Manages test execution, reporting)    │
└─────────────┬───────────────────────────┘
              │
┌─────────────▼───────────────────────────┐
│       Playwright Node.js Library        │
│   (API for browser automation)          │
└─────────────┬───────────────────────────┘
              │
      ┌───────┴───────┬──────────┐
      │               │          │
┌─────▼─────┐  ┌──────▼────┐  ┌─▼────────┐
│  Chromium │  │  Firefox  │  │  WebKit  │
│   (CDP)   │  │   (CDP)   │  │   (WI)   │
└───────────┘  └───────────┘  └──────────┘
```

**Key Architecture Features:**

1. **Direct Protocol Communication**
   - Uses Chrome DevTools Protocol (CDP) for Chromium
   - Uses patched Firefox for CDP support
   - Uses WebKit Inspector protocol

2. **Browser Context Isolation**
   - Multiple isolated browser sessions
   - No shared cookies/storage
   - Lightweight and fast

3. **Auto-waiting Mechanism**
   - Built into the core
   - Waits for actionability
   - No explicit waits needed

```javascript
// Architecture in code
const { chromium, firefox, webkit } = require('playwright');

test('architecture example', async () => {
  // Browser Process
  const browser = await chromium.launch();
  
  // Browser Context (isolated session)
  const context = await browser.newContext({
    viewport: { width: 1920, height: 1080 },
    // Each context has own cookies, localStorage, etc.
  });
  
  // Page (tab/window)
  const page = await context.newPage();
  
  // All communication goes through CDP
  await page.goto('https://example.com');
  
  // Hierarchy:
  // Browser → Context → Page → Frame
  
  await browser.close();
});

// Multiple contexts in one browser
test('multi-context architecture', async ({ browser }) => {
  // One browser process
  const browser = await chromium.launch();
  
  // Multiple isolated contexts
  const context1 = await browser.newContext(); // User 1
  const context2 = await browser.newContext(); // User 2
  
  const page1 = await context1.newPage();
  const page2 = await context2.newPage();
  
  // Completely isolated sessions
  await page1.goto('https://example.com');
  await page2.goto('https://example.com');
  
  await browser.close();
});
```

**Playwright vs Selenium Architecture:**

```
Selenium:
Test → WebDriver → Browser Driver → Browser
(JSON Wire Protocol, slow, indirect)

Playwright:
Test → Browser → DevTools Protocol
(Direct communication, fast, reliable)
```

---

### 3. Explain Browser, BrowserContext, and Page

**Theory:**
Understanding the hierarchy of Browser → Context → Page is fundamental to writing effective Playwright tests.

**Browser:**
- The actual browser application (Chrome, Firefox, Safari)
- Heavy resource - usually one per test run
- Can contain multiple contexts

**BrowserContext:**
- Isolated incognito-like session
- Has own cookies, localStorage, cache
- Lightweight - create many per browser
- Perfect for parallel testing

**Page:**
- Individual tab or window
- Belongs to one context
- Where actual testing happens

```javascript
// Relationship diagram
test('hierarchy explanation', async () => {
  // 1 Browser
  const browser = await chromium.launch();
  
  //   ├── Context 1 (User A's session)
  const contextA = await browser.newContext();
  //   │     ├── Page 1 (Tab 1)
  const pageA1 = await contextA.newPage();
  //   │     └── Page 2 (Tab 2)
  const pageA2 = await contextA.newPage();
  
  //   └── Context 2 (User B's session)
  const contextB = await browser.newContext();
  //         ├── Page 1 (Tab 1)
  const pageB1 = await contextB.newPage();
  //         └── Page 2 (Tab 2)
  const pageB2 = await contextB.newPage();
  
  await browser.close(); // Closes everything
});

// Browser methods
test('browser methods', async () => {
  const browser = await chromium.launch({
    headless: false,
    slowMo: 1000
  });
  
  console.log(browser.version()); // Browser version
  console.log(browser.isConnected()); // Connection status
  
  const contexts = browser.contexts(); // All contexts
  await browser.close();
});

// BrowserContext methods and configuration
test('context methods', async ({ browser }) => {
  const context = await browser.newContext({
    // Viewport
    viewport: { width: 1920, height: 1080 },
    
    // User agent
    userAgent: 'Custom UA',
    
    // Locale and timezone
    locale: 'en-US',
    timezoneId: 'America/New_York',
    
    // Permissions
    permissions: ['geolocation'],
    geolocation: { latitude: 40.7128, longitude: -74.0060 },
    
    // Color scheme
    colorScheme: 'dark',
    
    // HTTP credentials
    httpCredentials: {
      username: 'user',
      password: 'pass'
    },
    
    // Storage state
    storageState: 'auth.json',
    
    // Extra HTTP headers
    extraHTTPHeaders: {
      'Authorization': 'Bearer token'
    },
    
    // Offline mode
    offline: false,
    
    // Downloads
    acceptDownloads: true
  });
  
  // Context methods
  await context.addCookies([
    { name: 'session', value: '123', domain: 'example.com', path: '/' }
  ]);
  
  const cookies = await context.cookies();
  await context.clearCookies();
  
  await context.storageState({ path: 'state.json' });
  
  const pages = context.pages(); // All pages in context
  
  await context.close();
});

// Page methods
test('page methods', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Navigation
  await page.goBack();
  await page.goForward();
  await page.reload();
  
  // Information
  const title = await page.title();
  const url = page.url();
  const content = await page.content();
  
  // Viewport
  await page.setViewportSize({ width: 800, height: 600 });
  const viewport = page.viewportSize();
  
  // Screenshots
  await page.screenshot({ path: 'screenshot.png' });
  
  // PDF (Chromium only)
  await page.pdf({ path: 'page.pdf' });
  
  // Evaluate JavaScript
  const result = await page.evaluate(() => {
    return document.title;
  });
  
  // Events
  page.on('console', msg => console.log(msg.text()));
  page.on('dialog', dialog => dialog.accept());
  page.on('load', () => console.log('Page loaded'));
  
  // Close page
  await page.close();
});

// Real-world example: Multi-user scenario
test('multiple users shopping', async ({ browser }) => {
  // User 1: Add items to cart
  const user1Context = await browser.newContext();
  const user1Page = await user1Context.newPage();
  await user1Page.goto('https://shop.com');
  await user1Page.click('.product1 .add-to-cart');
  
  // User 2: Different session, empty cart
  const user2Context = await browser.newContext();
  const user2Page = await user2Context.newPage();
  await user2Page.goto('https://shop.com');
  
  // Verify isolation
  const user1Cart = await user1Page.locator('.cart-count').textContent();
  const user2Cart = await user2Page.locator('.cart-count').textContent();
  
  expect(user1Cart).toBe('1');
  expect(user2Cart).toBe('0');
  
  await user1Context.close();
  await user2Context.close();
});
```

---

### 4. How do you run tests in UI and non-UI mode?

**Theory:**
Playwright can run in headed mode (visible browser) for debugging and headless mode (no UI) for CI/CD. Each mode has its use cases.

```javascript
// Method 1: Command line
// Headless mode (default, no UI)
// npx playwright test

// Headed mode (visible browser)
// npx playwright test --headed

// UI mode (interactive test runner)
// npx playwright test --ui

// Method 2: Configuration file
// playwright.config.js
module.exports = {
  use: {
    headless: true, // or false for headed mode
    
    // Slow down for headed mode
    slowMo: process.env.HEADED ? 500 : 0
  }
};

// Method 3: Programmatic control
test('headed test', async () => {
  const browser = await chromium.launch({
    headless: false, // Show browser
    slowMo: 1000 // Slow down by 1 second
  });
  
  const page = await browser.newPage();
  await page.goto('https://example.com');
  
  await browser.close();
});

// Method 4: Environment-based
// playwright.config.js
use: {
  headless: process.env.CI ? true : false,
  // Headless in CI, headed locally
}

// Run with environment variable
// HEADED=true npx playwright test

// Method 5: Debug mode (always headed)
// npx playwright test --debug
// Opens Playwright Inspector

// UI Mode features
// npx playwright test --ui
// Provides:
// - Visual test execution
// - Step-through debugging
// - Time-travel
// - Watch mode
// - Trace viewer

// Real-world usage
test('debugging test', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Pause and open inspector (headed mode only)
  await page.pause();
  
  await page.click('button');
});

// Different modes for different scenarios
const config = {
  // Development: headed, slow
  dev: {
    headless: false,
    slowMo: 1000
  },
  
  // Testing: headless, fast
  test: {
    headless: true,
    slowMo: 0
  },
  
  // CI: headless, fast, retries
  ci: {
    headless: true,
    slowMo: 0,
    retries: 2
  }
};

// Use based on environment
use: config[process.env.ENV || 'test']
```

---

### 5. How do you add tags or annotations to tests?

**Theory:**
Tags and annotations help organize, filter, and categorize tests. They're useful for running specific test subsets and adding metadata for reporting.

```javascript
// Method 1: test.describe with tag
test.describe('@smoke', () => {
  test('login test', async ({ page }) => {
    await page.goto('https://example.com/login');
  });
  
  test('logout test', async ({ page }) => {
    await page.goto('https://example.com/logout');
  });
});

// Run tagged tests
// npx playwright test --grep @smoke

// Method 2: Test title tags
test('@critical User can login', async ({ page }) => {
  await page.goto('https://example.com/login');
});

test('@regression Dashboard loads', async ({ page }) => {
  await page.goto('https://example.com/dashboard');
});

// Run specific tag
// npx playwright test --grep @critical

// Method 3: Annotations for metadata
test('payment test', async ({ page }, testInfo) => {
  // Add annotation
  testInfo.annotations.push({
    type: 'issue',
    description: 'JIRA-123'
  });
  
  testInfo.annotations.push({
    type: 'priority',
    description: 'high'
  });
  
  await page.goto('https://example.com/checkout');
});

// Method 4: Multiple tags
test('@smoke @critical @fast Login works', async ({ page }) => {
  await page.goto('https://example.com/login');
});

// Run tests with multiple tags
// npx playwright test --grep "@smoke.*@critical"

// Method 5: Exclude tags
// Run all except @slow tests
// npx playwright test --grep-invert @slow

// Method 6: Custom annotations in config
// playwright.config.js
projects: [
  {
    name: 'smoke',
    grep: /@smoke/
  },
  {
    name: 'regression',
    grep: /@regression/
  }
]

// Run project: npx playwright test --project=smoke

// Method 7: Describe-level annotations
test.describe('Payment Tests', () => {
  test.beforeEach(async ({ }, testInfo) => {
    testInfo.annotations.push({ 
      type: 'feature', 
      description: 'payments' 
    });
  });
  
  test('credit card', async ({ page }) => {});
  test('paypal', async ({ page }) => {});
});

// Real-world example: Comprehensive tagging
test.describe('E-commerce Tests', () => {
  test('@smoke @fast @p0 Homepage loads', async ({ page }) => {
    await page.goto('https://shop.com');
    await expect(page).toHaveTitle(/Shop/);
  });
  
  test('@regression @slow @p1 Full checkout flow', async ({ page }, testInfo) => {
    testInfo.annotations.push({ type: 'jira', description: 'SHOP-456' });
    
    await page.goto('https://shop.com');
    // Full checkout process
  });
  
  test('@api @fast @p0 Product API', async ({ request }) => {
    const response = await request.get('https://api.shop.com/products');
    expect(response.status()).toBe(200);
  });
});

// Run combinations
// npx playwright test --grep "@smoke.*@fast"
// npx playwright test --grep "@p0|@p1"
// npx playwright test --grep "@regression" --grep-invert "@slow"
```

---

### 6. How do you run tests from different directories?

**Theory:**
Organizing tests in directories helps maintain large test suites. Playwright provides flexible ways to run tests from specific directories.

```javascript
// Directory structure
/*
tests/
  ├── auth/
  │   ├── login.spec.js
  │   └── signup.spec.js
  ├── checkout/
  │   ├── cart.spec.js
  │   └── payment.spec.js
  └── admin/
      ├── dashboard.spec.js
      └── users.spec.js
*/

// Method 1: Run specific directory
// npx playwright test tests/auth/
// npx playwright test tests/checkout/

// Method 2: Run multiple directories
// npx playwright test tests/auth/ tests/checkout/

// Method 3: Configure in playwright.config.js
module.exports = {
  testDir: './tests',
  
  projects: [
    {
      name: 'auth-tests',
      testMatch: '**/auth/**/*.spec.js'
    },
    {
      name: 'checkout-tests',
      testMatch: '**/checkout/**/*.spec.js'
    },
    {
      name: 'admin-tests',
      testMatch: '**/admin/**/*.spec.js'
    }
  ]
};

// Run specific project
// npx playwright test --project=auth-tests

// Method 4: Pattern matching
// npx playwright test **/auth/*.spec.js
// npx playwright test **/*payment*.spec.js

// Method 5: Glob patterns in config
testMatch: [
  '**/tests/**/*.spec.js',
  '**/e2e/**/*.test.js'
]

// Or exclude patterns
testIgnore: [
  '**/tests/draft/**',
  '**/tests/wip/**'
]

// Method 6: Environment-based directory selection
// playwright.config.js
testDir: process.env.TEST_TYPE === 'smoke' 
  ? './tests/smoke' 
  : './tests/regression'

// Run: TEST_TYPE=smoke npx playwright test

// Real-world example: Organized test structure
/*
e2e/
  ├── critical/          # P0 tests
  │   ├── login.spec.js
  │   └── checkout.spec.js
  ├── regression/        # Full regression
  │   ├── search.spec.js
  │   └── filters.spec.js
  └── integration/       # API tests
      └── api.spec.js
*/

// playwright.config.js
projects: [
  {
    name: 'critical',
    testDir: './e2e/critical',
    retries: 2
  },
  {
    name: 'regression',
    testDir: './e2e/regression',
    retries: 1
  },
  {
    name: 'integration',
    testDir: './e2e/integration',
    use: { baseURL: 'https://api.example.com' }
  }
]

// Run critical tests only
// npx playwright test --project=critical

// Run all tests from root
// npx playwright test ./e2e/
```

---

### 7. How do you maintain session or login state?

**Theory:**
Maintaining login state across tests saves time and speeds up test execution. Instead of logging in for each test, you authenticate once and reuse the session.

```javascript
// Method 1: Global setup with storage state
// global-setup.js
const { chromium } = require('@playwright/test');

async function globalSetup() {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  
  // Perform login
  await page.goto('https://example.com/login');
  await page.fill('#username', process.env.USERNAME);
  await page.fill('#password', process.env.PASSWORD);
  await page.click('button[type="submit"]');
  
  // Wait for successful login
  await page.waitForURL('**/dashboard');
  
  // Save authentication state
  await page.context().storageState({ path: 'auth.json' });
  
  await browser.close();
}

module.exports = globalSetup;

// playwright.config.js
module.exports = {
  globalSetup: require.resolve('./global-setup'),
  
  use: {
    // Use saved auth state
    storageState: 'auth.json'
  }
};

// Now all tests are pre-authenticated
test('access dashboard', async ({ page }) => {
  await page.goto('https://example.com/dashboard');
  // Already logged in!
});

// Method 2: Custom fixture for authenticated page
// fixtures.js
const { test: base } = require('@playwright/test');

const test = base.extend({
  authenticatedPage: async ({ browser }, use) => {
    const context = await browser.newContext({
      storageState: 'auth.json'
    });
    const page = await context.newPage();
    await use(page);
    await context.close();
  }
});

module.exports = { test };

// Use in tests
const { test } = require('./fixtures');

test('with auth fixture', async ({ authenticatedPage }) => {
  await authenticatedPage.goto('https://example.com/profile');
});

// Method 3: Multiple user roles
// admin-auth.json, user-auth.json, guest-auth.json

test.describe('Admin Tests', () => {
  test.use({ storageState: 'admin-auth.json' });
  
  test('admin can access dashboard', async ({ page }) => {
    await page.goto('https://example.com/admin');
  });
});

test.describe('User Tests', () => {
  test.use({ storageState: 'user-auth.json' });
  
  test('user can edit profile', async ({ page }) => {
    await page.goto('https://example.com/profile');
  });
});

// Method 4: Session management with beforeEach
let sharedContext;

test.beforeAll(async ({ browser }) => {
  // Create shared context with auth
  sharedContext = await browser.newContext({
    storageState: 'auth.json'
  });
});

test.afterAll(async () => {
  await sharedContext.close();
});

test('test 1', async () => {
  const page = await sharedContext.newPage();
  await page.goto('https://example.com');
  await page.close();
});

test('test 2', async () => {
  const page = await sharedContext.newPage();
  await page.goto('https://example.com/other');
  await page.close();
});

// Method 5: API-based authentication (fastest)
test('API auth', async ({ page, request }) => {
  // Get token via API
  const response = await request.post('https://example.com/api/login', {
    data: {
      username: 'user',
      password: 'pass'
    }
  });
  
  const { token } = await response.json();
  
  // Set token in page context
  await page.goto('https://example.com', {
    extraHTTPHeaders: {
      'Authorization': `Bearer ${token}`
    }
  });
});

// Method 6: Cookie-based session
test('cookie session', async ({ context, page }) => {
  // Set cookies directly
  await context.addCookies([
    {
      name: 'session_id',
      value: 'abc123',
      domain: 'example.com',
      path: '/'
    }
  ]);
  
  await page.goto('https://example.com/dashboard');
  // Logged in via cookie
});

// Real-world example: Multi-step auth setup
// auth.setup.js
const { test: setup, expect } = require('@playwright/test');

setup('authenticate', async ({ page }) => {
  // Navigate to login
  await page.goto('https://example.com/login');
  
  // Fill credentials
  await page.fill('#email', 'test@example.com');
  await page.fill('#password', 'SecurePass123!');
  
  // Submit form
  await page.click('button[type="submit"]');
  
  // Wait for redirect
  await page.waitForURL('**/dashboard');
  
  // Verify login success
  await expect(page.locator('.user-menu')).toBeVisible();
  
  // Save storage state
  await page.context().storageState({ 
    path: './playwright/.auth/user.json' 
  });
});

// Use in config
// playwright.config.js
module.exports = {
  projects: [
    { name: 'setup', testMatch: /.*\.setup\.js/ },
    {
      name: 'logged-in-tests',
      use: { 
        storageState: './playwright/.auth/user.json' 
      },
      dependencies: ['setup']
    }
  ]
};
```

---

I'll continue with the remaining sections. Would you like me to complete all remaining questions (sections 8-10 of Framework/Architecture, and all of Stability/Debugging/Mobile and Framework Design sections)?    await expect(page).toHaveScreenshot('firefox-homepage.png');
  });
});

// Real-world workflow
// 1. First run generates baseline images
// 2. Subsequent runs compare against baseline
// 3. If differences found, test fails and shows diff
// 4. Review diff images in test-results folder
// 5. Update baseline if changes are intentional

// Update baseline screenshots
// npx playwright test --update-snapshots
```

---

### 4. How do you integrate Playwright with CI/CD?

**Theory:**
CI/CD integration ensures tests run automatically on code changes. Playwright works seamlessly with all major CI/CD platforms.

```yaml
# GitHub Actions (.github/workflows/playwright.yml)
name: Playwright Tests

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - uses: actions/setup-node@v3
      with:
        node-version: 18
    
    - name: Install dependencies
      run: npm ci
    
    - name: Install Playwright Browsers
      run: npx playwright install --with-deps
    
    - name: Run Playwright tests
      run: npx playwright test
    
    - uses: actions/upload-artifact@v3
      if: always()
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 30
```

```yaml
# GitLab CI (.gitlab-ci.yml)
image: mcr.microsoft.com/playwright:v1.40.0-focal

test:
  script:
    - npm ci
    - npx playwright test
  artifacts:
    when: always
    paths:
      - playwright-report/
    expire_in: 1 week
```

```groovy
// Jenkins (Jenkinsfile)
pipeline {
    agent {
        docker {
            image 'mcr.microsoft.com/playwright:v1.40.0-focal'
        }
    }
    
    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npx playwright test'
            }
        }
    }
    
    post {
        always {
            publishHTML([
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright Report'
            ])
        }
    }
}
```

```yaml
# CircleCI (.circleci/config.yml)
version: 2.1

jobs:
  test:
    docker:
      - image: mcr.microsoft.com/playwright:v1.40.0-focal
    
    steps:
      - checkout
      - run: npm ci
      - run: npx playwright test
      
      - store_artifacts:
          path: playwright-report
      - store_test_results:
          path: test-results

workflows:
  test:
    jobs:
      - test
```

**CI/CD Best Practices:**

```javascript
// playwright.config.js optimized for CI
module.exports = {
  // Use CI-optimized settings
  workers: process.env.CI ? 2 : undefined,
  retries: process.env.CI ? 2 : 0,
  
  use: {
    // Reduce timeouts for CI
    actionTimeout: 10000,
    navigationTimeout: 30000,
    
    // Always capture on failure
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'retain-on-failure',
    
    // Headless in CI
    headless: true
  },
  
  // CI-friendly reporters
  reporter: [
    ['html'],
    ['github'], // GitHub Actions integration
    ['junit', { outputFile: 'results.xml' }]
  ]
};

// Environment-specific configuration
test.use({
  baseURL: process.env.BASE_URL || 'http://localhost:3000'
});
```

---

### 5. What are the benefits of Playwright in CI/CD?

**Theory:**
Playwright is designed with CI/CD in mind, offering features that make it ideal for automated testing pipelines.

**Key Benefits:**

1. **Fast Execution**
   - Parallel test execution by default
   - 2-3x faster than Selenium
   - Tests run concurrently across browsers

2. **Docker Support**
   - Official Docker images available
   - Pre-installed browsers
   - Consistent environment

3. **Reliable Tests**
   - Auto-waiting eliminates flakiness
   - Smart retry mechanisms
   - Network mocking for stable tests

4. **Comprehensive Debugging**
   - Trace viewer for failed tests
   - Screenshots and videos on failure
   - Detailed error messages

5. **Cross-browser Testing**
   - Test all browsers in one pipeline
   - Consistent API across browsers
   - Mobile browser emulation

```javascript
// Example: CI-optimized test suite
test.describe('CI Test Suite', () => {
  // Retry flaky tests in CI
  test.describe.configure({ 
    retries: process.env.CI ? 2 : 0 
  });
  
  test('critical user flow', async ({ page }) => {
    // Mock external dependencies
    await page.route('**/api/external/**', route => {
      route.fulfill({
        status: 200,
        body: JSON.stringify({ data: 'mocked' })
      });
    });
    
    await page.goto('/');
    await page.click('button#start');
    
    // Fast assertions
    await expect(page).toHaveURL(/dashboard/);
  });
});

// Performance optimization for CI
test('performance test', async ({ page }) => {
  // Skip heavy resources in CI
  if (process.env.CI) {
    await page.route('**/*.{png,jpg,jpeg}', route => route.abort());
  }
  
  await page.goto('/');
  await page.click('button');
});

// Conditional tests based on environment
test('production smoke test', async ({ page }) => {
  test.skip(!process.env.PROD_URL, 'Skipping - not production');
  
  await page.goto(process.env.PROD_URL);
  await expect(page.locator('h1')).toBeVisible();
});
```

**CI/CD Pipeline Benefits:**

```yaml
# Example: Comprehensive CI pipeline
stages:
  - lint
  - test
  - deploy

lint:
  script:
    - npm run lint

test-unit:
  script:
    - npm run test:unit

test-e2e-chrome:
  script:
    - npx playwright test --project=chromium
  parallel: 4  # Run 4 parallel jobs

test-e2e-firefox:
  script:
    - npx playwright test --project=firefox

test-e2e-webkit:
  script:
    - npx playwright test --project=webkit

deploy-staging:
  when: on_success
  script:
    - deploy to staging

deploy-production:
  when: manual
  script:
    - deploy to production
```

---

### 6. What are common Playwright actions?

**Theory:**
Actions are the core interactions in Playwright tests - clicking, typing, selecting, and navigating. Understanding all available actions helps write comprehensive tests.

```javascript
test('common actions', async ({ page }) => {
  await page.goto('https://example.com');
  
  // 1. Click actions
  await page.click('button'); // regular click
  await page.dblclick('button'); // double click
  await page.click('button', { force: true }); // bypass actionability checks
  await page.click('button', { button: 'right' }); // right click
  await page.click('button', { clickCount: 3 }); // triple click
  await page.click('button', { position: { x: 10, y: 10 } }); // click at position
  
  // 2. Input actions
  await page.fill('#input', 'text'); // fast input
  await page.type('#input', 'text', { delay: 100 }); // slow typing
  await page.press('#input', 'Enter'); // keyboard press
  await page.clear('#input'); // clear input (Playwright 1.27+)
  
  // 3. Keyboard actions
  await page.keyboard.press('Enter');
  await page.keyboard.type('Hello World');
  await page.keyboard.down('Shift');
  await page.keyboard.up('Shift');
  await page.keyboard.insertText('Paste text');
  
  // 4. Mouse actions
  await page.mouse.move(100, 200);
  await page.mouse.down();
  await page.mouse.up();
  await page.mouse.click(100, 200);
  await page.mouse.dblclick(100, 200);
  await page.mouse.wheel(0, 100); // scroll
  
  // 5. Hover
  await page.hover('button');
  await page.locator('button').hover();
  
  // 6. Focus
  await page.focus('#input');
  await page.locator('#input').focus();
  
  // 7. Select dropdown
  await page.selectOption('select#country', 'USA');
  await page.selectOption('select#country', { label: 'United States' });
  await page.selectOption('select#country', { value: 'us' });
  await page.selectOption('select#country', { index: 0 });
  
  // 8. Checkbox/Radio
  await page.check('input[type="checkbox"]');
  await page.uncheck('input[type="checkbox"]');
  await page.setChecked('input[type="checkbox"]', true);
  
  // 9. File upload
  await page.setInputFiles('input[type="file"]', 'path/to/file.pdf');
  await page.setInputFiles('input[type="file"]', [
    'file1.pdf',
    'file2.pdf'
  ]); // multiple files
  
  // 10. Drag and drop
  await page.dragAndDrop('#source', '#target');
  
  // 11. Scroll
  await page.evaluate(() => window.scrollBy(0, 1000));
  await page.locator('.footer').scrollIntoViewIfNeeded();
  
  // 12. Screenshot
  await page.screenshot({ path: 'screenshot.png' });
  await page.locator('.element').screenshot({ path: 'element.png' });
  
  // 13. Navigate
  await page.goto('https://example.com');
  await page.goBack();
  await page.goForward();
  await page.reload();
  
  // 14. Wait actions
  await page.waitForTimeout(1000); // hard wait (avoid)
  await page.waitForSelector('.element');
  await page.waitForURL('**/dashboard');
  await page.waitForLoadState('networkidle');
  
  // 15. Evaluate JavaScript
  await page.evaluate(() => console.log('Hello'));
  const title = await page.evaluate(() => document.title);
  await page.evaluate((text) => {
    document.querySelector('h1').textContent = text;
  }, 'New Title');
});

// Real-world complex actions
test('complex interaction', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Multi-step form interaction
  await page.fill('#firstName', 'John');
  await page.fill('#lastName', 'Doe');
  await page.selectOption('#country', 'USA');
  await page.check('#terms');
  await page.setInputFiles('#resume', 'resume.pdf');
  await page.click('button[type="submit"]');
  
  // Drag and drop with custom steps
  const source = page.locator('#draggable');
  const target = page.locator('#droppable');
  
  await source.hover();
  await page.mouse.down();
  await target.hover();
  await page.mouse.up();
  
  // Keyboard shortcuts
  await page.keyboard.press('Control+A'); // select all
  await page.keyboard.press('Control+C'); // copy
  await page.keyboard.press('Control+V'); // paste
});
```

---

### 7. How do you handle file upload and download?

**Theory:**
File handling is common in web applications. Playwright provides straightforward methods for both uploading files and intercepting downloads.

```javascript
// File Upload
test('file upload - single file', async ({ page }) => {
  await page.goto('https://example.com/upload');
  
  // Method 1: Upload from path
  await page.setInputFiles('input[type="file"]', 'path/to/file.pdf');
  
  // Method 2: Upload buffer
  await page.setInputFiles('input[type="file"]', {
    name: 'test.txt',
    mimeType: 'text/plain',
    buffer: Buffer.from('File content here')
  });
  
  await page.click('button#upload');
  await expect(page.locator('.success')).toBeVisible();
});

// Multiple files
test('file upload - multiple files', async ({ page }) => {
  await page.goto('https://example.com/upload');
  
  await page.setInputFiles('input[type="file"]', [
    'file1.pdf',
    'file2.pdf',
    'file3.pdf'
  ]);
  
  await page.click('button#upload');
});

// Remove files
test('remove uploaded files', async ({ page }) => {
  await page.goto('https://example.com/upload');
  
  // Upload files
  await page.setInputFiles('input[type="file"]', 'file.pdf');
  
  // Remove files (empty array)
  await page.setInputFiles('input[type="file"]', []);
});

// File Download
test('file download - basic', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Start waiting for download before clicking
  const downloadPromise = page.waitForEvent('download');
  await page.click('a#download-link');
  
  // Wait for download to complete
  const download = await downloadPromise;
  
  // Get download info
  console.log('Filename:', download.suggestedFilename());
  console.log('URL:', download.url());
  
  // Save to specific path
  await download.saveAs('./downloads/' + download.suggestedFilename());
  
  // Or get as buffer
  const buffer = await download.createReadStream();
});

// Download with custom filename
test('download with custom name', async ({ page }) => {
  await page.goto('https://example.com');
  
  const downloadPromise = page.waitForEvent('download');
  await page.click('button#export');
  const download = await downloadPromise;
  
  await download.saveAs('./downloads/my-custom-name.pdf');
});

// Verify downloaded file
const fs = require('fs');
const path = require('path');

test('verify downloaded file', async ({ page }) => {
  await page.goto('https://example.com');
  
  const downloadPromise = page.waitForEvent('download');
  await page.click('a#download');
  const download = await downloadPromise;
  
  const filePath = path.join('./downloads', download.suggestedFilename());
  await download.saveAs(filePath);
  
  // Verify file exists
  expect(fs.existsSync(filePath)).toBeTruthy();
  
  // Verify file size
  const stats = fs.statSync(filePath);
  expect(stats.size).toBeGreaterThan(0);
  
  // Read and verify content
  const content = fs.readFileSync(filePath, 'utf-8');
  expect(content).toContain('Expected content');
});

// Download in headless mode
test('download headless', async ({ browser }) => {
  const context = await browser.newContext({
    acceptDownloads: true // Important for downloads
  });
  
  const page = await context.newPage();
  await page.goto('https://example.com');
  
  const downloadPromise = page.waitForEvent('download');
  await page.click('a#download');
  const download = await downloadPromise;
  
  await download.saveAs('./downloads/file.pdf');
  await context.close();
});

// Real-world example: Download and parse CSV
test('download and parse CSV', async ({ page }) => {
  await page.goto('https://example.com/reports');
  
  const downloadPromise = page.waitForEvent('download');
  await page.click('button:text("Export CSV")');
  const download = await downloadPromise;
  
  const filePath = './downloads/report.csv';
  await download.saveAs(filePath);
  
  // Parse CSV
  const csv = require('csv-parser');
  const results = [];
  
  fs.createReadStream(filePath)
    .pipe(csv())
    .on('data', (data) => results.push(data))
    .on('end', () => {
      expect(results.length).toBeGreaterThan(0);
      expect(results[0]).toHaveProperty('name');
    });
});

// Upload drag and drop
test('drag and drop file upload', async ({ page }) => {
  await page.goto('https://example.com/upload');
  
  // Create file input
  const fileInput = await page.evaluateHandle(() => {
    const input = document.createElement('input');
    input.type = 'file';
    input.id = 'hidden-file-input';
    document.body.appendChild(input);
    return input;
  });
  
  // Upload file
  await fileInput.asElement().setInputFiles('file.pdf');
  
  // Trigger drop event
  await page.evaluate((input) => {
    const dropZone = document.querySelector('.drop-zone');
    const dataTransfer = new DataTransfer();
    dataTransfer.items.add(input.files[0]);
    
    const event = new DragEvent('drop', { dataTransfer });
    dropZone.dispatchEvent(event);
  }, fileInput);
});
```

---

### 8. How do you compare screenshots?

**Theory:**
Screenshot comparison is used for visual regression testing. Playwright can automatically compare screenshots and highlight differences, helping catch unintended UI changes.

```javascript
// Basic screenshot comparison
test('visual regression - basic', async ({ page }) => {
  await page.goto('https://example.com');
  
  // First run: creates baseline
  // Subsequent runs: compares against baseline
  await expect(page).toHaveScreenshot('homepage.png');
  
  // If different, test fails and creates diff image
});

// Configure comparison settings
test('visual with threshold', async ({ page }) => {
  await page.goto('https://example.com');
  
  await expect(page).toHaveScreenshot('page.png', {
    maxDiffPixels: 100, // Allow 100 pixels to differ
    threshold: 0.2, // Allow 20% difference
  });
});

// Full page comparison
test('full page screenshot', async ({ page }) => {
  await page.goto('https://example.com');
  
  await expect(page).toHaveScreenshot('fullpage.png', {
    fullPage: true, // Capture entire scrollable page
    animations: 'disabled', // Disable animations for consistency
  });
});

// Element screenshot comparison
test('element comparison', async ({ page }) => {
  await page.goto('https://example.com');
  
  const header = page.locator('.header');
  await expect(header).toHaveScreenshot('header.png');
  
  const footer = page.locator('.footer');
  await expect(footer).toHaveScreenshot('footer.png');
});

// Mask dynamic content
test('mask dynamic elements', async ({ page }) => {
  await page.goto('https://example.com');
  
  await expect(page).toHaveScreenshot('page.png', {
    mask: [
      page.locator('.timestamp'), // Hide timestamp
      page.locator('.user-avatar'), // Hide avatar
      page.locator('.live-data') // Hide live updating data
    ]
  });
});

// Compare specific region
test('clip region comparison', async ({ page }) => {
  await page.goto('https://example.com');
  
  await expect(page).toHaveScreenshot('region.png', {
    clip: {
      x: 0,
      y: 0,
      width: 500,
      height: 500
    }
  });
});

// Cross-browser visual comparison
test('compare across browsers', async ({ page, browserName }) => {
  await page.goto('https://example.com');
  
  // Different baseline for each browser
  await expect(page).toHaveScreenshot(`${browserName}-homepage.png`);
});

// Responsive design comparison
const breakpoints = [
  { width: 375, height: 667, name: 'mobile' },
  { width: 768, height: 1024, name: 'tablet' },
  { width: 1920, height: 1080, name: 'desktop' }
];

for (const bp of breakpoints) {
  test(`visual test - ${bp.name}`, async ({ page }) => {
    await page.setViewportSize({ width: bp.width, height: bp.height });
    await page.goto('https://example.com');
    
    await expect(page).toHaveScreenshot(`${bp.name}.png`);
  });
}

// Update baseline screenshots
// Run this when you intentionally change UI
// npx playwright test --update-snapshots

// Real-world example: Theme comparison
test('light vs dark theme', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Light theme
  await expect(page).toHaveScreenshot('light-theme.png');
  
  // Switch to dark theme
  await page.click('#theme-toggle');
  await page.waitForTimeout(500); // wait for transition
  
  // Dark theme
  await expect(page).toHaveScreenshot('dark-theme.png');
});

// Component states comparison
test('button states', async ({ page }) => {
  await page.goto('https://example.com/components');
  
  const button = page.locator('button.primary');
  
  // Normal state
  await expect(button).toHaveScreenshot('button-normal.png');
  
  // Hover state
  await button.hover();
  await expect(button).toHaveScreenshot('button-hover.png');
  
  // Active state
  await button.click({ noWaitAfter: true });
  await expect(button).toHaveScreenshot('button-active.png');
  
  // Disabled state
  await button.evaluate(b => b.disabled = true);
  await expect(button).toHaveScreenshot('button-disabled.png');
});

// Handling animations
test('wait for animations', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Wait for animations to complete
  await page.waitForFunction(() => {
    return !document.querySelector('.animated-element').getAnimations().length;
  });
  
  await expect(page).toHaveScreenshot('no-animations.png');
});
```

---

### 9. What types of reports are available in Playwright?

**Theory:**
Playwright supports multiple report formats for different needs - interactive HTML reports for developers, JUnit/XML for CI integration, and JSON for custom processing.

```javascript
// playwright.config.js
module.exports = {
  reporter: [
    // 1. HTML Reporter (default, interactive)
    ['html', {
      outputFolder: 'playwright-report',
      open: 'never', // 'always', 'never', 'on-failure'
      host: 'localhost',
      port: 9223
    }],
    
    // 2. List Reporter (console output)
    ['list'],
    
    // 3. Dot Reporter (minimal console)
    ['dot'],
    
    // 4. Line Reporter (one line per test)
    ['line'],
    
    // 5. JSON Reporter (machine-readable)
    ['json', {
      outputFile: 'test-results.json'
    }],
    
    // 6. JUnit Reporter (CI/CD integration)
    ['junit', {
      outputFile: 'junit-results.xml'
    }],
    
    // 7. GitHub Actions Reporter (GitHub integration)
    ['github'],
    
    // 8. Blob Reporter (sharding)
    ['blob', {
      outputDir: 'blob-report'
    }]
  ]
};

// Multiple reporters simultaneously
reporter: [
  ['html'],
  ['json', { outputFile: 'results.json' }],
  ['junit', { outputFile: 'junit.xml' }],
  ['list'] // Console output
]
```

**Report Types Explained:**

```javascript
// HTML Reporter - Most detailed
// Features:
// - Interactive web interface
// - Screenshots and videos
// - Trace viewer integration
// - Filtering and search
// - Timeline view
// npx playwright show-report

// JSON Reporter - For custom processing
test('with JSON metadata', async ({ page }, testInfo) => {
  testInfo.annotations.push({
    type: 'issue',
    description: 'JIRA-123'
  });
  
  await page.goto('https://example.com');
  
  // This metadata appears in JSON report
});

// JUnit Reporter - For CI/CD
// Produces XML file compatible with Jenkins, Azure DevOps, etc.

// Custom Reporter
class MyReporter {
  onBegin(config, suite) {
    console.log(`Starting test run with ${suite.allTests().length} tests`);
  }
  
  onTestEnd(test, result) {
    console.log(`Finished ${test.title}: ${result.status}`);
  }
  
  onEnd(result) {
    console.log(`Finished test run: ${result.status}`);
  }
}

module.exports = MyReporter;

// Use custom reporter
// playwright.config.js
reporter: './my-reporter.js'
```

**Viewing Reports:**

```bash
# Open HTML report
npx playwright show-report

# Open report from specific location
npx playwright show-report ./custom-report

# HTML report automatically opens in browser
# Shows:
# - Test results summary
# - Pass/Fail counts
# - Execution time
# - Screenshots
# - Videos
# - Trace files
# - Error messages
```

---

### 10. What types of waits are supported in Playwright?

**Theory:**
Waits ensure elements and pages are ready before interactions. Playwright has auto-waiting built-in, but explicit waits provide fine-grained control for complex scenarios.

```javascript
// 1. Auto-waiting (built-in, recommended)
test('auto-waiting', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Automatically waits for element to be:
  // - Attached, Visible, Stable, Enabled, Receiving events
  await page.click('button'); // Waits up to 30s by default
  await page.fill('#input', 'text');
  await page.selectOption('select', 'value');
});

// 2. Wait for selector
test('wait for selector', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Wait for element to appear
  await page.waitForSelector('.dynamic-element');
  
  // With state options
  await page.waitForSelector('.element', { state: 'visible' });
  await page.waitForSelector('.element', { state: 'hidden' });
  await page.waitForSelector('.element', { state: 'attached' });
  await page.waitForSelector('.element', { state: 'detached' });
  
  // With timeout
  await page.waitForSelector('.element', { timeout: 5000 });
});

// 3. Wait for URL
test('wait for URL', async ({ page }) => {
  await page.goto('https://example.com');
  await page.click('a.link');
  
  // Wait for specific URL
  await page.waitForURL('https://example.com/dashboard');
  
  // Wait for URL pattern
  await page.waitForURL('**/dashboard');
  await page.waitForURL(/.*profile.*/);
  
  // With timeout
  await page.waitForURL('**/page', { timeout: 10000 });
});

// 4. Wait for load state
test('wait for load state', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Wait for DOM to be loaded
  await page.waitForLoadState('domcontentloaded');
  
  // Wait for page to fully load
  await page.waitForLoadState('load');
  
  // Wait for no network activity (600ms idle)
  await page.waitForLoadState('networkidle');
});

// 5. Wait for function (custom condition)
test('wait for function', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Wait until JavaScript condition is true
  await page.waitForFunction(() => {
    return document.querySelectorAll('.item').length > 5;
  });
  
  // With arguments
  await page.waitForFunction(
    (minCount) => document.querySelectorAll('.item').length >= minCount,
    5
  );
  
  // With polling interval
  await page.waitForFunction(
    () => document.querySelector('.status').textContent === 'Ready',
    { polling: 1000, timeout: 30000 }
  );
});

// 6. Wait for timeout (avoid when possible)
test('wait for timeout', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Hard wait - BAD PRACTICE
  await page.waitForTimeout(3000); // Waits 3 seconds
  
  // Only use when no alternative exists
});

// 7. Wait for event
test('wait for event', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Wait for navigation
  const navigationPromise = page.waitForNavigation();
  await page.click('a.link');
  await navigationPromise;
  
  // Wait for popup
  const popupPromise = page.waitForEvent('popup');
  await page.click('a[target="_blank"]');
  const popup = await popupPromise;
  
  // Wait for console message
  const consolePromise = page.waitForEvent('console');
  await page.click('button');
  const message = await consolePromise;
});

// 8. Wait for request/response
test('wait for network', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Wait for specific API call
  const responsePromise = page.waitForResponse('**/api/data');
  await page.click('button#load');
  const response = await responsePromise;
  console.log(await response.json());
  
  // Wait for request
  const requestPromise = page.waitForRequest('**/api/submit');
  await page.click('button#submit');
  const request = await requestPromise;
  
  // Wait for successful response
  await page.waitForResponse(
    response => response.url().includes('/api/') && response.status() === 200
  );
});

// 9. Locator wait
test('locator wait methods', async ({ page }) => {
  await page.goto('https://example.com');
  
  const locator = page.locator('.element');
  
  // Wait for locator
  await locator.waitFor({ state: 'visible' });
  await locator.waitFor({ state: 'hidden' });
  await locator.waitFor({ state: 'attached' });
  await locator.waitFor({ state: 'detached' });
  
  // Wait for count
  await expect(locator).toHaveCount(5, { timeout: 10000 });
});

// 10. Wait for navigation with timeout
test('navigation with custom timeout', async ({ page }) => {
  await page.goto('https://example.com');
  
  await Promise.# Playwright Interview Questions & Answers - Complete Guide

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
    timezone: 'America/New_York',
    permissions: ['geolocation', 'notifications'],
    geolocation: { latitude: 40.7128, longitude: -74.0060 },
    colorScheme: 'dark',
    extraHTTPHeaders: {
      'Authorization': 'Bearer token123'
    }
  });
  
  const page = await context.newPage();
  await page.goto('https://example.com');
  
  await context.close();
});

// Context with authentication
test('authenticated context', async ({ browser }) => {
  const context = await browser.newContext({
    httpCredentials: {
      username: 'admin',
      password: 'password123'
    }
  });
  
  const page = await context.newPage();
  await page.goto('https://example.com/protected');
});

// Real-world example: E-commerce multi-user
test('shopping cart isolation', async ({ browser }) => {
  // User 1 adds items
  const context1 = await browser.newContext();
  const page1 = await context1.newPage();
  await page1.goto('https://shop.com');
  await page1.click('.add-to-cart:nth-child(1)');
  const cart1 = await page1.locator('.cart-count').textContent();
  
  // User 2 has empty cart (isolated context)
  const context2 = await browser.newContext();
  const page2 = await context2.newPage();
  await page2.goto('https://shop.com');
  const cart2 = await page2.locator('.cart-count').textContent();
  
  expect(cart1).toBe('1');
  expect(cart2).toBe('0'); // Completely isolated
  
  await context1.close();
  await context2.close();
});

// Performance: Context vs Browser
test('performance comparison', async ({ browser }) => {
  // Creating new context (fast ~50ms)
  const start1 = Date.now();
  const context = await browser.newContext();
  console.log(`Context creation: ${Date.now() - start1}ms`);
  
  await context.close();
  
  // Creating new browser (slow ~500ms)
  const { chromium } = require('playwright');
  const start2 = Date.now();
  const newBrowser = await chromium.launch();
  console.log(`Browser launch: ${Date.now() - start2}ms`);
  
  await newBrowser.close();
});
```

---

### 8. How does Playwright handle waiting mechanisms?

**Theory:**
Playwright's intelligent waiting is one of its biggest advantages. It eliminates flaky tests by automatically waiting for elements to be actionable before performing actions.

**Auto-waiting checks:**
1. Element is attached to DOM
2. Element is visible
3. Element is stable (not animating)
4. Element receives events (not obscured)
5. Element is enabled (for inputs/buttons)

```javascript
// Auto-waiting in action
test('auto-waiting demonstration', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Playwright automatically waits for ALL these conditions:
  await page.click('button'); 
  // ✓ Button exists in DOM
  // ✓ Button is visible
  // ✓ Button is not animating
  // ✓ Button is not covered by another element
  // ✓ Button is enabled
  
  // Same for other actions
  await page.fill('#input', 'text'); 
  // Waits for input to be visible, enabled, and ready
  
  await page.selectOption('select', 'value');
  // Waits for dropdown to be ready
});

// Default timeout is 30 seconds
test('timeout configuration', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Custom timeout for specific action
  await page.click('button', { timeout: 5000 }); // 5 seconds
  
  // Set timeout for all actions
  page.setDefaultTimeout(60000); // 60 seconds
  
  // No timeout (wait forever)
  await page.click('button', { timeout: 0 });
});

// Wait for specific states
test('wait for states', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Wait for element to be visible
  await page.waitForSelector('.element', { state: 'visible' });
  
  // Wait for element to be hidden
  await page.waitForSelector('.loading', { state: 'hidden' });
  
  // Wait for element to be attached (doesn't need to be visible)
  await page.waitForSelector('.element', { state: 'attached' });
  
  // Wait for element to be detached (removed from DOM)
  await page.waitForSelector('.modal', { state: 'detached' });
});

// Wait for navigation
test('navigation waiting', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Click and wait for navigation
  await Promise.all([
    page.waitForNavigation(),
    page.click('a.link')
  ]);
  
  // Wait for specific URL
  await page.click('button');
  await page.waitForURL('**/dashboard');
  
  // Wait for URL pattern
  await page.waitForURL(/.*profile.*/);
  
  // Wait for load states
  await page.waitForLoadState('load'); // full load
  await page.waitForLoadState('domcontentloaded'); // DOM ready
  await page.waitForLoadState('networkidle'); // no network activity
});

// Wait for API responses
test('wait for API', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Wait for specific API call
  const responsePromise = page.waitForResponse('**/api/users');
  await page.click('button#load-users');
  const response = await responsePromise;
  
  console.log(await response.json());
  
  // Wait for successful response
  await page.waitForResponse(
    response => response.url().includes('/api/data') && response.status() === 200
  );
});

// Wait for custom conditions
test('custom waiting conditions', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Wait until JavaScript condition is true
  await page.waitForFunction(() => {
    return document.querySelectorAll('.item').length > 10;
  });
  
  // Wait with arguments
  await page.waitForFunction(
    (minItems) => document.querySelectorAll('.item').length >= minItems,
    10
  );
  
  // Wait for text content
  await page.waitForFunction(() => {
    const el = document.querySelector('.status');
    return el && el.textContent === 'Ready';
  });
});

// Real-world example: AJAX loading
test('handle AJAX content', async ({ page }) => {
  await page.goto('https://example.com/search');
  
  // Type in search
  await page.fill('#search', 'playwright');
  
  // Auto-waits for API response and results to appear
  await page.click('.search-results .first-result');
  
  // If you need explicit control:
  await page.fill('#search', 'playwright');
  await page.waitForResponse('**/api/search');
  await page.waitForSelector('.search-results .item');
  await page.click('.first-result');
});

// Comparison with manual waiting (bad practice)
test('manual vs auto waiting', async ({ page }) => {
  await page.goto('https://example.com');
  
  // ❌ BAD - Manual waiting (flaky)
  await page.click('#load-data');
  await page.waitForTimeout(5000); // hope 5 seconds is enough
  await page.click('.data-item');
  
  // ✅ GOOD - Auto-waiting (reliable)
  await page.click('#load-data');
  await page.click('.data-item'); // automatically waits for element
  
  // ✅ BETTER - Explicit wait for condition
  await page.click('#load-data');
  await page.waitForSelector('.data-item');
  await page.click('.data-item');
});
```

---

### 9. What debugging tools are available in Playwright?

**Theory:**
Playwright provides comprehensive debugging tools that make troubleshooting tests much easier than traditional automation frameworks.

```javascript
// 1. Playwright Inspector (most useful)
// Run with --debug flag
// npx playwright test --debug

// This opens Playwright Inspector which allows you to:
// - Step through test line by line
// - Inspect locators
// - Record new actions
// - See screenshots at each step

// 2. Headed mode
// npx playwright test --headed
test('debug in headed mode', async ({ page }) => {
  await page.goto('https://example.com');
  await page.pause(); // Pauses execution, opens inspector
  await page.click('button');
});

// 3. Slow motion
// npx playwright test --headed --slow-mo=1000
// Slows down operations by 1000ms

// In code:
const { chromium } = require('playwright');
test('slow motion', async () => {
  const browser = await chromium.launch({
    headless: false,
    slowMo: 1000
  });
  const page = await browser.newPage();
  await page.goto('https://example.com');
});

// 4. page.pause() - Interactive debugging
test('pause during test', async ({ page }) => {
  await page.goto('https://example.com');
  await page.fill('#username', 'user');
  
  await page.pause(); // Opens inspector, pauses here
  
  await page.click('button');
});

// 5. Console logging
test('console debugging', async ({ page }) => {
  // Listen to console messages from page
  page.on('console', msg => {
    console.log('Browser console:', msg.text());
  });
  
  await page.goto('https://example.com');
  
  // Evaluate and log
  const title = await page.title();
  console.log('Page title:', title);
  
  // Log element properties
  const buttonText = await page.locator('button').textContent();
  console.log('Button text:', buttonText);
});

// 6. Screenshots for debugging
test('screenshot debugging', async ({ page }, testInfo) => {
  await page.goto('https://example.com');
  
  // Take screenshot at specific points
  await page.screenshot({ path: 'before-click.png' });
  await page.click('button');
  await page.screenshot({ path: 'after-click.png' });
  
  // Attach to report
  await testInfo.attach('debug-screenshot', {
    body: await page.screenshot(),
    contentType: 'image/png'
  });
});

// 7. Trace Viewer (most powerful)
// Configure in playwright.config.js
use: {
  trace: 'on-first-retry', // or 'on', 'retain-on-failure'
}

// Or programmatically
test('with trace', async ({ page, context }) => {
  await context.tracing.start({ screenshots: true, snapshots: true });
  
  await page.goto('https://example.com');
  await page.click('button');
  
  await context.tracing.stop({ path: 'trace.zip' });
});

// View trace: npx playwright show-trace trace.zip

// 8. Verbose logging
// npx playwright test --reporter=list --workers=1

// 9. Page.evaluate for debugging
test('evaluate debugging', async ({ page }) => {
  await page.goto('https://example.com');
  
  // Check element state
  const isVisible = await page.evaluate(() => {
    const el = document.querySelector('.element');
    return el && window.getComputedStyle(el).display !== 'none';
  });
  console.log('Element visible?', isVisible);
  
  // Get all element info
  const elementInfo = await page.locator('.element').evaluate(el => ({
    tagName: el.tagName,
    id: el.id,
    className: el.className,
    visible: el.offsetParent !== null,
    text: el.textContent,
    value: el.value
  }));
  console.log('Element info:', elementInfo);
});

// 10. Network debugging
test('network debugging', async ({ page }) => {
  // Log all requests
  page.on('request', request => {
    console.log('Request:', request.url());
  });
  
  // Log all responses
  page.on('response', response => {
    console.log('Response:', response.url(), response.status());
  });
  
  // Log failed requests
  page.on('requestfailed', request => {
    console.log('Failed:', request.url(), request.failure());
  });
  
  await page.goto('https://example.com');
});

// 11. UI Mode (interactive test runner)
// npx playwright test --ui
// Provides:
// - Visual test runner
// - Watch mode
// - Time travel debugging
// - Trace viewer integration

// 12. VS Code extension
// Install "Playwright Test for VSCode"
// Provides:
// - Run tests from editor
// - Debug with breakpoints
// - Record tests
// - Pick locators
```

---

### 10. What is Codegen in Playwright?

**Theory:**
Codegen is Playwright's test generator tool that records your browser interactions and generates test code automatically. It's perfect for quickly creating test scripts or learning Playwright syntax.

```bash
# Basic codegen
npx playwright codegen https://example.com

# Codegen with specific browser
npx playwright codegen --browser=firefox https://example.com
npx playwright codegen --browser=webkit https://example.com

# Codegen with device emulation
npx playwright codegen --device="iPhone 12" https://example.com
npx playwright codegen --device="Pixel 5" https://example.com

# Codegen with custom viewport
npx playwright codegen --viewport-size=800,600 https://example.com

# Codegen with authentication
npx playwright codegen --load-storage=auth.json https://example.com

# Codegen and save to file
npx playwright codegen https://example.com -o my-test.spec.js

# Codegen with specific language
npx playwright codegen --target=python https://example.com
npx playwright codegen --target=java https://example.com
npx playwright codegen --target=csharp https://example.com
```

**How Codegen works:**
1. Opens browser and Playwright Inspector
2. Records your actions (clicks, typing, navigation)
3. Generates code in real-time
4. Shows suggested locators
5. You can copy code or save to file

**What Codegen records:**
- Navigation (goto)
- Clicks
- Text input (fill)
- Selections (select option)
- Checkbox/radio selections
- Hover actions
- Screenshots

```javascript
// Example generated code from Codegen
const { test, expect } = require('@playwright/test');

test('test', async ({ page }) => {
  // Codegen generated this automatically:
  await page.goto('https://example.com/');
  await page.getByRole('link', { name: 'Get started' }).click();
  await page.getByLabel('Email').fill('test@example.com');
  await page.getByLabel('Password').fill('password123');
  await page.getByRole('button', { name: 'Sign in' }).click();
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
});

// You can then refine and improve this generated code
```

**Codegen best practices:**
1. Use it to learn Playwright locator strategies
2. Generate initial test structure quickly
3. Always review and optimize generated code
4. Use it to find good locators for elements
5. Combine with manual coding for complex scenarios

```javascript
// Real-world workflow
// 1. Generate base test with codegen
// npx playwright codegen https://myapp.com/login

// 2. Codegen produces:
test('generated login', async ({ page }) => {
  await page.goto('https://myapp.com/login');
  await page.fill('#username', 'testuser');
  await page.fill('#password', 'password');
  await page.click('button[type="submit"]');
});

// 3. You refine it:
test('refined login', async ({ page }) => {
  await page.goto('https://myapp.com/login');
  
  // Add better structure
  const username = 'testuser';
  const password = 'password';
  
  await page.fill('#username', username);
  await page.fill('#password', password);
  await page.click('button[type="submit"]');
  
  // Add assertions that codegen didn't include
  await expect(page).toHaveURL(/dashboard/);
  await expect(page.locator('.user-profile')).toBeVisible();
});
```

---

## Advanced Automation Concepts

### 1. How do you handle authentication in Playwright?

**Theory:**
Authentication is needed for most web applications. Playwright provides efficient ways to handle login once and reuse authentication state across all tests, saving significant execution time.

```javascript
// Method 1: Global setup (best for reusing auth)
// auth.setup.js
const { test: setup } = require('@playwright/test');

setup('authenticate', async ({ page }) => {
  await page.goto('https://example.com/login');
  await page.fill('#username', 'testuser');
  await page.fill('#password', 'Password123!');
  await page.click('button[type="submit"]');
  
  // Wait for authentication
  await page.waitForURL('**/dashboard');
  
  // Save authentication state
  await page.context().storageState({ path: 'auth.json' });
});

// playwright.config.js
module.exports = {
  projects: [
    { name: 'setup', testMatch: '**/*.setup.js' },
    {
      name: 'chromium',
      use: { 
        ...devices['Desktop Chrome'],
        storageState: 'auth.json' // Reuse auth
      },
      dependencies: ['setup'] // Run setup first
    }
  ]
};

// All tests now use authenticated state
test('access protected page', async ({ page }) => {
  await page.goto('https://example.com/profile');
  // Already logged in!
});

// Method 2: beforeEach hook
test.beforeEach(async ({ page }) => {
  await page.goto('https://example.com/login');
  await page.fill('#username', 'testuser');
  await page.fill('#password', 'password');
  await page.click('button[type="submit"]');
  await page.waitForURL('**/dashboard');
});

test('test 1', async ({ page }) => {
  await page.goto('/profile');
  // Logged in from beforeEach
});

test('test 2', async ({ page }) => {
  await page.goto('/settings');
  // Logged in from beforeEach
});

// Method 3: Custom fixture for authentication
const { test: base } = require('@playwright/test');

const test = base.extend({
  authenticatedPage: async ({ browser }, use) => {
    const context = await browser.newContext({
      storageState: 'auth.json'
    });
    const page = await context.newPage();
    await use(page);
    await context.close();
  }
});

test('use authenticated page', async ({ authenticatedPage }) => {
  await authenticatedPage.goto('https://example.com/profile');
});

// Method 4: API-based authentication (fastest)
test('API authentication', async ({ page, request }) => {
  // Login via API
  const response = await request.post('https://example.com/api/login', {
    data: {
      username: 'testuser',
      password: 'password123'
    }
  });
  
  const { token } = await response.json();
  
  // Set authentication in context
  await page.goto('https://example.com', {
    extraHTTPHeaders: {
      'Authorization': `Bearer ${token}`
    }
  });
});

// Method 5: HTTP Basic Auth
test('basic auth', async ({ browser }) => {
  const context = await browser.newContext({
    httpCredentials: {
      username: 'admin',
      password: 'admin123'
    }
  });
  
  const page = await context.newPage();
  await page.goto('https://example.com/admin');
  
  await context.close();
});

// Method 6: Multiple user roles
// admin-auth.json, user-auth.json, guest-auth.json

test.describe('Admin tests', () => {
  test.use({ storageState: 'admin-auth.json' });
  
  test('admin dashboard', async ({ page }) => {
    await page.goto('/admin');
  });
});

test.describe('User tests', () => {
  test.use({ storageState: 'user-auth.json' });
  
  test('user profile', async ({ page }) => {
    await page.goto('/profile');
  });
});

// Method 7: OAuth/Social login
test('google oauth', async ({ page, context }) => {
  await page.goto('https://example.com/login');
  
  // Click OAuth button
  const [oauthPage] = await Promise.all([
    context.waitForEvent('page'),
    page.click('button:text("Login with Google")')
  ]);
  
  // Handle OAuth flow
  await oauthPage.fill('#email', 'test@gmail.com');
  await oauthPage.fill('#password', 'password');
  await oauthPage.click('#login');
  
  // Wait for redirect back
  await page.waitForURL('**/dashboard');
});
```

---

### 2. What is network interception and why is it used?

**Theory:**
Network interception allows you to monitor, modify, or mock network requests and responses. This is crucial for testing different scenarios without backend changes, testing error states, and speeding up tests.

**Use cases:**
- Mock API responses
- Test error scenarios (500, 404)
- Simulate slow networks
- Block unnecessary resources (images, ads)
- Modify request headers
- Test offline functionality

```javascript
// Basic request interception
test('intercept and mock API', async ({ page }) => {
  // Intercept all API calls matching pattern
  await page.route('**/api/users', route => {
    route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify([
        { id: 1, name: 'John Doe' },
        { id: 2, name: 'Jane Smith' }
      ])
    });
  });
  
  await page.goto('https://example.com');
  // Page loads with mocked user data
});

// Intercept and modify request
test('modify request headers', async ({ page }) => {
  await page.route('**/api/**', route => {
    const headers = route.request().headers();
    headers['Authorization'] = 'Bearer fake-token';
    
    route.continue({ headers });
  });
  
  await page.goto('https://example.com');
});

// Mock different scenarios
test('test error handling', async ({ page }) => {
  // Mock 500 error
  await page.route('**/api/submit', route => {
    route.fulfill({
      status: 500,
      body: 'Internal Server Error'
    });
  });
  
  await page.goto('https://example.com/form');
  await page.fill('#input', 'data');
  await page.click('button#submit');
  
  // Verify error message is shown
  await expect(page.locator('.error-message')).toBeVisible();
});

// Block resources (speed up tests)
test('block images and CSS', async ({ page }) => {
  await page.route('**/*.{png,jpg,jpeg,gif,svg,css}', route => {
    route.abort(); // Block these resources
  });
  
  await page.goto('https://example.com');
  // Loads faster without images/CSS
});

// Conditional mocking
test('conditional mock', async ({ page }) => {
  await page.route('**/api/data', route => {
    const url = route.request().url();
    
    if (url.includes('test-mode')) {
      route.fulfill({
        body: JSON.stringify({ mode: 'test', data: [] })
      });
    } else {
      route.continue(); // Let real request go through
    }
  });
  
  await page.goto('https://example.com?test-mode=true');
});

// Simulate slow network
test('test slow loading', async ({ page }) => {
  await page.route('**/api/data', async route => {
    // Wait 5 seconds before responding
    await new Promise(resolve => setTimeout(resolve, 5000));
    
    route.fulfill({
      body: JSON.stringify({ data: 'slow response' })
    });
  });
  
  await page.goto('https://example.com');
  // Verify loading spinner appears
  await expect(page.locator('.loading-spinner')).toBeVisible();
});

// Monitor network activity
test('log all requests', async ({ page }) => {
  // Log all requests
  page.on('request', request => {
    console.log('→', request.method(), request.url());
  });
  
  // Log all responses
  page.on('response', response => {
    console.log('←', response.status(), response.url());
  });
  
  await page.goto('https://example.com');
});

// Wait for specific API call
test('wait for API response', async ({ page }) => {
  const responsePromise = page.waitForResponse('**/api/users');
  
  await page.goto('https://example.com');
  await page.click('button#load-users');
  
  const response = await responsePromise;
  const data = await response.json();
  
  expect(data.length).toBeGreaterThan(0);
});

// Mock file upload
test('mock file upload', async ({ page }) => {
  await page.route('**/upload', route => {
    route.fulfill({
      status: 200,
      body: JSON.stringify({ 
        success: true, 
        fileId: '12345' 
      })
    });
  });
  
  await page.goto('https://example.com/upload');
  await page.setInputFiles('input[type="file"]', 'test.pdf');
  await page.click('button#upload');
  
  await expect(page.locator('.success-message')).toBeVisible();
});

// Real-world example: Test pagination without real data
test('test pagination with mock data', async ({ page }) => {
  let pageNumber = 1;
  
  await page.route('**/api/items**', route => {
    const url = new URL(route.request().url());
    const page = url.searchParams.get('page') || '1';
    
    route.fulfill({
      status: 200,
      body: JSON.stringify({
        page: parseInt(page),
        items: Array.from({ length: 10 }, (_, i) => ({
          id: i + 1,
          name: `Item ${i + 1} Page ${page}`
        })),
        totalPages: 5
      })
    });
  });
  
  await page.goto('https://example.com/items');
  
  // Test pagination
  await expect(page.locator('.item')).toHaveCount(10);
  await page.click('button:text("Next")');
  await expect(page.locator('.item').first()).toContainText('Page 2');
});
```

---

### 3. How do you perform UI regression testing?

**Theory:**
UI regression testing ensures visual elements haven't changed unexpectedly. Playwright can compare screenshots to detect visual differences, catching CSS changes, layout shifts, or UI bugs.

```javascript
// Method 1: Visual comparison (built-in)
test('visual regression', async ({ page }) => {
  await page.goto('https://example.com');
  
  // First run creates baseline screenshot
  // Subsequent runs compare against baseline
  await expect(page).toHaveScreenshot('homepage.png');
  
  // Element screenshot comparison
  await expect(page.locator('.header')).toHaveScreenshot('header.png');
});

// Method 2: Full page visual testing
test('full page visual', async ({ page }) => {
  await page.goto('https://example.com');
  
  await expect(page).toHaveScreenshot('fullpage.png', {
    fullPage: true,
    animations: 'disabled', // disable animations for consistent screenshots
    mask: [page.locator('.timestamp')], // mask dynamic content
  });
});

// Method 3: Configure threshold for differences
// playwright.config.js
expect: {
  toHaveScreenshot: {
    maxDiffPixels: 100, // allow 100 pixels difference
    threshold: 0.2 // 20% threshold
  }
}

// Method 4: Test multiple breakpoints
const viewports = [
  { width: 375, height: 667, name: 'mobile' },
  { width: 768, height: 1024, name: 'tablet' },
  { width: 1920, height: 1080, name: 'desktop' }
];

for (const viewport of viewports) {
  test(`visual test - ${viewport.name}`, async ({ page }) => {
    await page.setViewportSize({ 
      width: viewport.width, 
      height: viewport.height 
    });
    
    await page.goto('https://example.com');
    await expect(page).toHaveScreenshot(`homepage-${viewport.name}.png`);
  });
}

// Method 5: Component-level visual testing
test('button visual regression', async ({ page }) => {
  await page.goto('https://example.com/components');
  
  // Test different button states
  const button = page.locator('button.primary');
  
  // Normal state
  await expect(button).toHaveScreenshot('button-normal.png');
  
  // Hover state
  await button.hover();
  await expect(button).toHaveScreenshot('button-hover.png');
  
  // Disabled state
  await button.evaluate(btn => btn.disabled = true);
  await expect(button).toHaveScreenshot('button-disabled.png');
});

// Method 6: Mask dynamic content
test('mask dynamic elements', async ({ page }) => {
  await page.goto('https://example.com');
  
  await expect(page).toHaveScreenshot('page.png', {
    mask: [
      page.locator('.timestamp'),
      page.locator('.user-avatar'),
      page.locator('.live-counter')
    ]
  });
});

// Method 7: Ignore specific areas
test('ignore regions', async ({ page }) => {
  await page.goto('https://example.com');
  
  await expect(page).toHaveScreenshot('page.png', {
    clip: { x: 0, y: 100, width: 1280, height: 600 }, // only test this region
    mask: [page.locator('.ads')]
  });
});

// Method 8: Cross-browser visual testing
test.describe('Visual tests', () => {
  test('chrome visual', async ({ page }) => {
    test.skip(({ browserName }) => browserName !== 'chromium');
    await page.goto('https://example.com');
    await expect(page).toHaveScreenshot('chrome-homepage.png');
  });
  
  test('firefox visual', async ({ page }) => {
    test.skip(({ browserName }) => browserName !== 'firefox');
    await page.goto('https://example.com
