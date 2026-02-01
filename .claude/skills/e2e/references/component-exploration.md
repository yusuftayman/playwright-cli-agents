# Component Exploration Reference

This reference guide explains how to explore and discover UI components using playwright-cli for test generation.

## Exploration Workflow

```
1. NAVIGATE → Open target URL
2. SNAPSHOT → Capture accessibility tree
3. IDENTIFY → Find interactive elements with refs
4. INTERACT → Test element interactions
5. DOCUMENT → Record element selectors for Page Objects
```

---

## CLI Browser Commands for Exploration

### Navigation

```bash
playwright-cli open https://example.com
playwright-cli go-back
playwright-cli go-forward
playwright-cli reload
```

### Snapshots & Screenshots

```bash
playwright-cli snapshot
playwright-cli screenshot
playwright-cli screenshot --full-page
playwright-cli screenshot e12   # Screenshot specific element
```

### Interactions

```bash
playwright-cli click e12
playwright-cli fill e15 "test@example.com"
playwright-cli hover e20
playwright-cli select e25 "US"
playwright-cli press Enter
```

### Form Filling

```bash
playwright-cli fill e1 "test@example.com"
playwright-cli check e2
playwright-cli uncheck e3
playwright-cli select e4 "Option A"
```

### Debugging

```bash
playwright-cli console
playwright-cli network
playwright-cli eval "document.title"
```

### Tab Management

```bash
playwright-cli tab-list
playwright-cli tab-new https://example.com/other
playwright-cli tab-close 1
playwright-cli tab-select 0
```

---

## Element Discovery from Snapshots

When you run `playwright-cli snapshot`, you receive an accessibility tree with element references.

### Snapshot Output Example

```
- button "Create Strategy" [ref: e12]
- textbox "Search" [ref: e13]
- grid [ref: e14]
- button "Filters" [ref: e15]
- combobox "Select option" [ref: e16]
- checkbox "Remember me" [ref: e17]
```

### Key Element Types to Identify

| Element Type | What to Look For | Test Actions |
|--------------|------------------|--------------|
| **Buttons** | `button`, `link`, clickable elements | Click, verify state change |
| **Inputs** | `textbox`, `searchbox`, `spinbutton` | Fill, clear, validate |
| **Dropdowns** | `combobox`, `listbox` | Select options, verify selection |
| **Checkboxes** | `checkbox`, `switch` | Toggle, verify state |
| **Navigation** | `tab`, `menuitem`, `link` | Click, verify navigation |
| **Modals** | `dialog`, elements with modal roles | Open, close, interact |
| **Tables/Grids** | `grid`, `row`, `cell` | Row count, cell values |

---

## Locator Strategy Priority

When creating Page Objects, use these locator strategies in order of preference:

### 1. ID-based (Most Stable)
```typescript
page.locator('#create-strategy')
page.locator('#search')
```

### 2. Data Attributes
```typescript
page.locator('[data-attr-value="active"]')
page.locator('[data-test-id="submit-button"]')
```

### 3. Role-based with Accessible Name
```typescript
page.getByRole('button', { name: 'Submit' })
page.getByRole('textbox', { name: 'Email' })
page.getByRole('tab', { name: 'Settings' })
```

### 4. Class-based (For Components)
```typescript
page.locator('.ag-root')
page.locator('.in-dropdown-box__option')
```

### 5. Text Content
```typescript
page.locator('button', { hasText: 'Submit' })
page.locator('a', { hasText: 'Back to List' })
```

### 6. Compound Locators
```typescript
page.locator('.in-dropdown-box__option').filter({ hasText: 'Option Text' })
page.locator('ue-ui-container').filter({ hasText: /^Button Text$/ }).locator('#undefined')
```

---

## Interaction Testing Flow

Before generating Page Objects, always test interactions:

```bash
# 1. Open page
playwright-cli open https://example.com

# 2. Take snapshot
playwright-cli snapshot

# 3. Identify element refs from snapshot output

# 4. Test interaction
playwright-cli click e12

# 5. Verify state change
playwright-cli snapshot

# 6. Document working selectors for Page Object
```

---

## Common UI Patterns

### Modal/Dialog Pattern
```
1. Click trigger button → Modal opens
2. Take snapshot → Identify modal elements
3. Interact with modal content
4. Close modal → Verify modal hidden
```

### Form Pattern
```
1. Navigate to form page
2. Snapshot → Identify all form fields
3. Fill each field
4. Submit form
5. Verify success/error states
```

### Navigation Pattern
```
1. Identify navigation elements
2. Click navigation item
3. Verify URL change
4. Verify page content loaded
```

### Grid/Table Pattern
```
1. Wait for grid to load
2. Snapshot → Identify row/cell selectors
3. Test row click, cell values
4. Test pagination if exists
5. Test sorting/filtering
```
