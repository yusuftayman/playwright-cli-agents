# API Mocking Reference

Mock API responses in E2E tests using the `mockApi` utility for deterministic, fast, and reliable tests.

## Why Mock APIs?

| Benefit | Description |
|---------|-------------|
| **Deterministic** | Same data every run, no flaky tests |
| **Fast** | No network latency |
| **Edge Cases** | Easily test empty states, errors, limits |
| **No Backend** | Tests run without real API |

---

## Infrastructure

### Mock Utility

**Location:** `__tests__/ui/mockServer.ts`

```typescript
import { mockApi } from '../mockServer';
```

### Mock Files

**Location:** `__mocks__/api/`

```
__mocks__/api/
├── {endpoint}/
│   ├── empty.json
│   ├── populated.json
│   └── error.json
└── {simple-endpoint}.json
```

### Endpoint Constants

**Location:** `__tests__/ui/enums/constants.ts`

```typescript
import { API_ENDPOINTS } from '../enums/constants';
```

---

## Usage

### Basic Pattern

```typescript
import { test, expect } from '@playwright/test';
import { API_ENDPOINTS } from '../enums/constants';
import { mockApi } from '../mockServer';

test('displays empty state', async ({ page }) => {
  // Mock BEFORE navigation
  await mockApi(page, `**/${API_ENDPOINTS.LIST_DATA}**`, 'list-data/empty.json');

  await page.goto('/your-page');
  await expect(page.locator('[data-testid="empty-state"]')).toBeVisible();
});
```

### Multiple Mocks

```typescript
test('page with dependencies', async ({ page }) => {
  await mockApi(page, `**/${API_ENDPOINTS.LIST_DATA}**`, 'list-data/populated.json');
  await mockApi(page, `**/${API_ENDPOINTS.SETTINGS}**`, 'settings/enabled.json');
  await mockApi(page, `**/${API_ENDPOINTS.USER_INFO}**`, 'user/admin.json');

  await page.goto('/dashboard');
});
```

### State Testing

```typescript
test.describe('List States', () => {

  test('empty state @visual', async ({ page }) => {
    await mockApi(page, `**/${API_ENDPOINTS.LIST_DATA}**`, 'list-data/empty.json');
    const listPage = new ListPage(page);
    await listPage.goto('/list');
    await expect(listPage.emptyState).toHaveScreenshot('list-empty.png');
  });

  test('populated state @visual', async ({ page }) => {
    await mockApi(page, `**/${API_ENDPOINTS.LIST_DATA}**`, 'list-data/populated.json');
    const listPage = new ListPage(page);
    await listPage.goto('/list');
    await expect(listPage.gridRoot).toHaveScreenshot('list-populated.png');
  });

});
```

### Feature Flags

```typescript
test('feature disabled', async ({ page }) => {
  await mockApi(page, `**/${API_ENDPOINTS.FEATURE}**`, 'feature/disabled.json');
  await page.goto('/page');
  await expect(page.locator('[data-testid="setup-prompt"]')).toBeVisible();
});

test('feature enabled', async ({ page }) => {
  await mockApi(page, `**/${API_ENDPOINTS.FEATURE}**`, 'feature/enabled.json');
  await page.goto('/page');
  await expect(page.locator('[data-testid="feature-panel"]')).toBeVisible();
});
```

---

## Mock File Naming

| State | File Name |
|-------|-----------|
| Empty/zero | `empty.json` |
| With data | `populated.json` |
| Maximum | `max.json` |
| Enabled | `enabled.json` |
| Disabled | `disabled.json` |
| Role-based | `{role}.json` |

---

## Best Practices

### 1. Mock BEFORE Navigation

```typescript
// Correct
await mockApi(page, `**/${API_ENDPOINTS.DATA}**`, 'data/empty.json');
await page.goto('/page');

// Wrong - too late
await page.goto('/page');
await mockApi(page, ...);
```

### 2. Mock All Dependencies

```typescript
// Mock every API the page calls
await mockApi(page, `**/${API_ENDPOINTS.LIST}**`, 'list/populated.json');
await mockApi(page, `**/${API_ENDPOINTS.SETTINGS}**`, 'settings/default.json');
await page.goto('/page');
```

### 3. Use Endpoint Constants

```typescript
// Use constants
await mockApi(page, `**/${API_ENDPOINTS.LIST_DATA}**`, 'list-data/empty.json');

// Avoid hardcoded strings
await mockApi(page, '**/get-list-data**', 'list-data/empty.json');
```

### 4. Mocks in Test File, Not Page Object

```typescript
// Correct - mock in test
test('list view', async ({ page }) => {
  await mockApi(page, `**/${API_ENDPOINTS.LIST}**`, 'list/populated.json');
  const listPage = new ListPage(page);
  await listPage.goto();
});
```

---

## Pattern Matching

```typescript
// Any URL containing endpoint
`**/${API_ENDPOINTS.LIST_DATA}**`

// Specific path
`**/api/v1/${API_ENDPOINTS.LIST_DATA}**`

// With query params
`**/${API_ENDPOINTS.LIST_DATA}?**`
```

---

## Debugging

Check network activity with playwright-cli:

```bash
playwright-cli network
```
