# Integration Test Examples

This file contains examples and templates for writing integration tests with visual regression testing using the `@visual` tag pattern in test titles.

---

## Test File Structure

```typescript
import { test, expect } from '@playwright/test';
import { [PageName]Page } from './pages/[page-name]-page';

test.describe('[Feature Name]', () => {

  test('should [describe expected behavior]', async ({ page }) => {
    const pageObject = new [PageName]Page(page);
    await pageObject.goto('URL');

    // Arrange
    await pageObject.search('test query');

    // Act
    await pageObject.clickSubmit();

    // Assert
    await expect(page).toHaveURL(/expected-url/);
  });

  test('should display error for invalid input', async ({ page }) => {
    const pageObject = new [PageName]Page(page);
    await pageObject.goto('URL');

    // Act
    await pageObject.search('invalid');
    await pageObject.clickSubmit();

    // Assert
    const errorText = await pageObject.getErrorMessageText();
    expect(errorText).toContain('expected error');
  });

});
```

---

## Integration Tests with Visual Regression (@visual)

Visual tests are integrated with regular integration tests using the `@visual` tag in the test title and `toHaveScreenshot()` assertion:

```typescript
test.describe('[Feature Name]', () => {

  test('should display page correctly @visual', async ({ page }) => {
    const pageObject = new [PageName]Page(page);
    await pageObject.goto('URL');

    // Wait for page to stabilize
    await pageObject.waitForLoadingToComplete();

    // Visual snapshot - component-level, not full-page
    await expect(pageObject.contentArea).toHaveScreenshot('page-initial-state.png');
  });

  test('should display modal correctly @visual', async ({ page }) => {
    const pageObject = new [PageName]Page(page);
    await pageObject.goto('URL');
    await pageObject.clickExampleButton();
    await pageObject.waitForModal();

    // Visual snapshot of modal - component-level
    await expect(pageObject.modalTitle.locator('..')).toHaveScreenshot('modal-open.png');
  });

});
```

**Note:** The `@visual` tag allows you to run only visual tests with: `npx playwright test --grep @visual`

---

## Complete Integration Test Example with Page Objects

```typescript
import { test, expect } from '@playwright/test';
import { FeaturePage } from './pages/feature-page';
import { TEST_DATA } from '../constants/test-data';

/**
 * Feature Block Integration Tests
 *
 * All interaction logic is in the FeaturePage class (no separate helpers)
 */

test.describe('Feature Block Settings', () => {

  test('should add block to template and open settings @visual', async ({ page }) => {
    const featurePage = new FeaturePage(page);
    await featurePage.goto();
    await featurePage.setupFeatureBlock();
    await featurePage.navigateToTab('Settings');
    await expect(featurePage.contentArea).toHaveScreenshot();
  });

});

test.describe('Feature Configuration', () => {

  test('should configure link @visual', async ({ page }) => {
    const featurePage = new FeaturePage(page);
    await featurePage.goto();
    await featurePage.setupAndNavigateToSettings();
    await featurePage.setLinkValue(TEST_DATA.URLS.EXAMPLE);
    await expect(featurePage.contentArea).toHaveScreenshot();
  });

  test('should configure email @visual', async ({ page }) => {
    const featurePage = new FeaturePage(page);
    await featurePage.goto();
    await featurePage.setupAndNavigateToSettings();
    await featurePage.configureLinkType('MAIL');
    await featurePage.setEmailConfiguration(
      TEST_DATA.CONTACT.EMAIL,
      TEST_DATA.EMAIL.SUBJECT,
      TEST_DATA.EMAIL.BODY
    );
    await expect(featurePage.contentArea).toHaveScreenshot();
  });

});

test.describe('Feature Alignment', () => {

  test('should set alignment to left @visual', async ({ page }) => {
    const featurePage = new FeaturePage(page);
    await featurePage.goto();
    await featurePage.setupAndNavigateToSettings();
    await featurePage.setAlignment(0);
    await expect(featurePage.contentArea).toHaveScreenshot();
  });

  test('should set alignment to center @visual', async ({ page }) => {
    const featurePage = new FeaturePage(page);
    await featurePage.goto();
    await featurePage.setupAndNavigateToSettings();
    await featurePage.setAlignment(1);
    await expect(featurePage.contentArea).toHaveScreenshot();
  });

  test('should set alignment to right @visual', async ({ page }) => {
    const featurePage = new FeaturePage(page);
    await featurePage.goto();
    await featurePage.setupAndNavigateToSettings();
    await featurePage.setAlignment(2);
    await expect(featurePage.contentArea).toHaveScreenshot();
  });

});

test.describe('Feature Styles', () => {

  test('should open styles tab @visual', async ({ page }) => {
    const featurePage = new FeaturePage(page);
    await featurePage.goto();
    await featurePage.setupAndNavigateToStyles();
    await expect(featurePage.contentArea).toHaveScreenshot();
  });

  test('should set background color @visual', async ({ page }) => {
    const featurePage = new FeaturePage(page);
    await featurePage.goto();
    await featurePage.setupAndNavigateToStyles();
    await featurePage.setBackgroundColor();
    await expect(featurePage.contentArea).toHaveScreenshot();
  });

  test('should set font family @visual', async ({ page }) => {
    const featurePage = new FeaturePage(page);
    await featurePage.goto();
    await featurePage.setupAndNavigateToStyles();
    await featurePage.setFontFamily();
    await expect(featurePage.contentArea).toHaveScreenshot();
  });

});
```

---

## Tests with API Mocking

Use `mockApi` utility for deterministic test data. See [references/api-mocking.md](../references/api-mocking.md) for full documentation.

```typescript
import { test, expect } from '@playwright/test';
import { FeaturePage } from './pages/feature-page';
import { API_ENDPOINTS } from '../enums/constants';
import { mockApi } from '../mockServer';

test.describe('Feature with Mocked API', () => {

  test('empty state @visual', async ({ page }) => {
    // Setup mocks BEFORE navigation
    await mockApi(page, `**/${API_ENDPOINTS.LIST_DATA}**`, 'list-data/empty.json');

    const featurePage = new FeaturePage(page);
    await featurePage.goto();

    await expect(featurePage.emptyState).toBeVisible();
    await expect(featurePage.contentArea).toHaveScreenshot();
  });

  test('populated state @visual', async ({ page }) => {
    await mockApi(page, `**/${API_ENDPOINTS.LIST_DATA}**`, 'list-data/populated.json');

    const featurePage = new FeaturePage(page);
    await featurePage.goto();

    await expect(featurePage.dataGrid).toBeVisible();
    await expect(featurePage.contentArea).toHaveScreenshot();
  });

  test('multiple API dependencies', async ({ page }) => {
    await mockApi(page, `**/${API_ENDPOINTS.LIST_DATA}**`, 'list-data/populated.json');
    await mockApi(page, `**/${API_ENDPOINTS.SETTINGS}**`, 'settings/enabled.json');
    await mockApi(page, `**/${API_ENDPOINTS.USER_INFO}**`, 'user/admin.json');

    const featurePage = new FeaturePage(page);
    await featurePage.goto();
    // All API calls are mocked
  });

});
```

---

## Edge Case Tests

```typescript
test.describe('[Feature Name] - Edge Cases', () => {

  test('should handle empty state', async ({ page }) => {
    const pageObject = new [PageName]Page(page);
    await pageObject.goto('URL');

    // Search for non-existent item
    await pageObject.search('nonexistent_item_xyz');
    await pageObject.waitForGridUpdate();

    // Assert empty state is shown
    const rowCount = await pageObject.getGridRowCount();
    expect(rowCount).toBe(0);
  });

  test('should handle loading state', async ({ page }) => {
    const pageObject = new [PageName]Page(page);
    await pageObject.goto('URL');

    // Trigger action that causes loading
    await pageObject.clickSubmit();

    // Verify loading completes
    await pageObject.waitForLoadingToComplete();
    await expect(pageObject.successToast).toBeVisible();
  });

});
```

---

## Visual Testing Best Practices

### ⭐ ALWAYS Use Component-Level Screenshots (Required)

**Never use full-page screenshots.** Component-level screenshots are more stable and focused:

```typescript
// ✅ CORRECT - Component-level screenshot (use Page Object locators)
test('should display header in preview mode @visual', async ({ page }) => {
  const guidoPage = new GuidoPage(page);
  await guidoPage.goto();
  await guidoPage.togglePreview();

  // Screenshot the specific component, not the whole page
  await expect(guidoPage.header).toHaveScreenshot('preview-mode-header.png');
});

test('should display global styles panel @visual', async ({ page }) => {
  const guidoPage = new GuidoPage(page);
  await guidoPage.goto();

  // Screenshot the settings panel only
  await expect(guidoPage.globalStylesPanel).toHaveScreenshot('global-styles-panel.png');
});

// ❌ WRONG - Full-page screenshots are fragile and should be avoided
test('bad example @visual', async ({ page }) => {
  await expect(page).toHaveScreenshot('page.png'); // DON'T DO THIS
});
```

### Add Component Locators to Page Objects

Define component locators in your Page Object constructor:

```typescript
export class GuidoPage {
  readonly header: Locator;
  readonly globalStylesPanel: Locator;
  readonly leftSidebar: Locator;
  readonly editorCanvas: Locator;

  constructor(page: Page) {
    this.header = page.locator('.guido__header, header').first();
    this.globalStylesPanel = page.getByRole('tabpanel', { name: 'General Styles' });
    this.leftSidebar = page.locator('.left-sidebar, [class*="sidebar"]').first();
    this.editorCanvas = page.locator('.editor-canvas, #stripoEditorContainer').first();
  }
}
```

### Masking Dynamic Content

When dynamic content (like timers) is inside your component, mask it:

```typescript
test('component with dynamic content @visual', async ({ page }) => {
  const guidoPage = new GuidoPage(page);
  await guidoPage.goto();

  // Mask any dynamic elements within the component
  await expect(guidoPage.editorCanvas).toHaveScreenshot('canvas.png', {
    mask: [
      page.locator('[class*="timer"], [class*="countdown"]'),
    ]
  });
});
```

### Strict Pixel Comparison (Default)

Use strict comparison in `playwright.config.ts` - no threshold:

```typescript
expect: {
  toHaveScreenshot: {
    maxDiffPixels: 0, // Strict - any pixel difference fails
  },
}
```

Individual tests can override only when absolutely necessary:

```typescript
// Only use when the component has unavoidable minor variations
await expect(guidoPage.somePanel).toHaveScreenshot('panel.png', {
  maxDiffPixelRatio: 0.01, // 1% threshold - use sparingly
});
```

### Running Only Visual Tests

```bash
# Run only tests tagged with @visual
npx playwright test --grep @visual

# Run all tests EXCEPT visual tests
npx playwright test --grep-invert @visual
```

---

## Demo/Integration Test Pattern

For tests that combine multiple interactions with visual verification:

```typescript
import { test, expect } from '@playwright/test';
import { FeaturePage } from './pages/feature-page';

test.describe('Feature Demo', () => {

  test('should create complete demo with styles and preview @visual', async ({ page }) => {
    const featurePage = new FeaturePage(page);
    await featurePage.goto();
    await featurePage.setupDemoFeature();
    await featurePage.previewChanges();
    await expect(featurePage.contentArea).toHaveScreenshot();
  });

  test('should interact with element in preview @visual', async ({ page }) => {
    const featurePage = new FeaturePage(page);
    await featurePage.goto();
    await featurePage.setupDemoFeature();
    await featurePage.previewChanges();
    await expect(featurePage.contentArea).toHaveScreenshot();

    // Interact with element inside iframe
    await featurePage.clickPreviewElement('Click Me');
    await expect(featurePage.contentArea).toHaveScreenshot();
  });

});
```

---

## Test Constants Pattern

Create constants files for reusable test data:

```typescript
// constants/test-data.ts
export const TEST_DATA = {
  URLS: {
    EXAMPLE: 'https://example.com',
    GOOGLE: 'https://www.google.com/',
    FTP_SERVER: 'ftp://ftp.example.com/files',
  },
  CONTACT: {
    EMAIL: 'test@example.com',
    PHONE: '+1234567890',
  },
  TEXT: {
    SHORT: 'Test',
    LONG: 'This is a longer test string for testing',
  },
  VALUES: {
    HEIGHT_PX: '50',
    REPEAT_COUNT: 3,
  },
  CONTROLS: {
    PADDING: {
      INCREASE_TOP: 'Increase Top Padding by',
      DECREASE_BOTTOM: 'Decrease Bottom Padding by',
    },
  },
};
```

---

## File Structure

```
__tests__/
├── e2e/
│   ├── pages/                # Page Object classes (ALL locators & logic here)
│   │   ├── home-page.ts
│   │   ├── login-page.ts
│   │   └── dashboard-page.ts
│   ├── home.spec.ts          # Integration tests (may include @visual tagged tests)
│   ├── login.spec.ts         # Integration tests (may include @visual tagged tests)
│   └── dashboard.spec.ts     # Integration tests (may include @visual tagged tests)
└── constants/
    └── test-data.ts          # Only test data (URLs, emails, text values)
```

**Important Notes:**
- **All locators in Page Objects** - No separate selectors file
- **No helpers folder** - All interaction methods belong in Page Object classes
- **No fixtures folder** - Use standard Playwright `{ page }` destructuring
- **No visual folder** - Visual tests use `@visual` tag in test title for filtering
