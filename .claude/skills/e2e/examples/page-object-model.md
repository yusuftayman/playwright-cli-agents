# Page Object Model Examples

This file contains examples and templates for creating Page Object classes following project patterns.

---

## Page Object Template

```typescript
import { Page, Locator } from '@playwright/test';

export class [PageName]Page {
  readonly page: Page;

  // ============================================================================
  // LOCATORS - Define all locators as readonly properties
  // ============================================================================

  readonly exampleButton: Locator;
  readonly searchInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;
  readonly successToast: Locator;
  readonly loadingSpinner: Locator;

  // Grid/Table Locators (if applicable)
  readonly gridRoot: Locator;
  readonly gridRows: Locator;
  readonly gridHeader: Locator;

  // Modal/Drawer Locators (if applicable)
  readonly modalTitle: Locator;
  readonly modalCloseButton: Locator;
  readonly modalConfirmButton: Locator;
  readonly modalCancelButton: Locator;

  constructor(page: Page) {
    this.page = page;

    // Initialize all locators in constructor
    this.exampleButton = page.locator('#example-button');
    this.searchInput = page.locator('#search');
    this.submitButton = page.locator('#submit');
    this.errorMessage = page.locator('.error-message');
    this.successToast = page.locator('#toast.success');
    this.loadingSpinner = page.locator('.loading-spinner');

    // Grid locators
    this.gridRoot = page.locator('.ag-root, .grid-container');
    this.gridRows = page.locator('[row-index]');
    this.gridHeader = page.locator('.ag-header, .grid-header');

    // Modal locators
    this.modalTitle = page.locator('.modal-title, .in-modal-v2-wrapper__title');
    this.modalCloseButton = page.locator('#close-button, .modal-close');
    this.modalConfirmButton = page.locator('#confirm-button, .modal-confirm');
    this.modalCancelButton = page.locator('#cancel-button, .modal-cancel');
  }

  // ============================================================================
  // NAVIGATION METHODS
  // ============================================================================

  async goto(url: string): Promise<void> {
    await this.page.goto(url);
    await this.gridRoot.waitFor({ state: 'visible' });
    await this.waitForPageLoad();
  }

  // ============================================================================
  // INTERACTION METHODS - Each method waits for element visibility
  // ============================================================================

  async clickExampleButton(): Promise<void> {
    await this.exampleButton.waitFor({ state: 'visible' });
    await this.exampleButton.click();
  }

  async search(searchText: string): Promise<void> {
    await this.searchInput.waitFor({ state: 'visible' });
    await this.searchInput.fill(searchText);
  }

  async clickSubmit(): Promise<void> {
    await this.submitButton.waitFor({ state: 'visible' });
    await this.submitButton.click();
  }

  // ============================================================================
  // WAIT METHODS
  // ============================================================================

  async waitForPageLoad(): Promise<void> {
    await this.page.waitForLoadState('networkidle');
  }

  async waitForLoadingToComplete(): Promise<void> {
    const isLoading = await this.loadingSpinner.isVisible().catch(() => false);
    if (isLoading) {
      await this.loadingSpinner.waitFor({ state: 'hidden', timeout: 30000 });
    }
  }

  async waitForGridUpdate(): Promise<void> {
    try {
      await this.page.waitForResponse(
        response => response.url().includes('/api/') && response.status() === 200,
        { timeout: 3000 }
      );
    } catch {
      // Response may not always be triggered
    }
    await this.gridRoot.waitFor({ state: 'visible' });
    await this.gridRows.first().waitFor({ state: 'visible', timeout: 5000 }).catch(() => undefined);
  }

  // ============================================================================
  // ASSERTION HELPER METHODS
  // ============================================================================

  async getErrorMessageText(): Promise<string> {
    await this.errorMessage.waitFor({ state: 'visible' });
    return (await this.errorMessage.innerText()).trim();
  }

  async isSuccessToastVisible(): Promise<boolean> {
    return await this.successToast.isVisible().catch(() => false);
  }

  async waitForSuccessToast(): Promise<void> {
    await this.successToast.waitFor({ state: 'visible' });
  }

  async getSuccessToastText(): Promise<string> {
    await this.waitForSuccessToast();
    return (await this.successToast.innerText()).trim();
  }

  // ============================================================================
  // GRID/TABLE HELPER METHODS (if applicable)
  // ============================================================================

  async getGridRowCount(): Promise<number> {
    return await this.gridRows.count();
  }

  async getCellTextByRowAndCol(rowIndex: number, colId: string): Promise<string> {
    const cell = this.gridRoot.locator(`[row-index="${rowIndex}"] [col-id="${colId}"]`);
    await cell.waitFor({ state: 'visible' });
    return (await cell.innerText()).trim();
  }

  async clickRowByIndex(rowIndex: number): Promise<void> {
    const row = this.gridRows.nth(rowIndex);
    await row.waitFor({ state: 'visible' });
    await row.click();
  }

  // ============================================================================
  // MODAL/DRAWER HELPER METHODS (if applicable)
  // ============================================================================

  async waitForModal(): Promise<void> {
    await this.modalTitle.waitFor({ state: 'visible' });
  }

  async closeModal(): Promise<void> {
    await this.modalCloseButton.waitFor({ state: 'visible' });
    await this.modalCloseButton.click();
    await this.modalTitle.waitFor({ state: 'hidden' });
  }

  async confirmModal(): Promise<void> {
    await this.modalConfirmButton.waitFor({ state: 'visible' });
    await this.modalConfirmButton.click();
  }

  async cancelModal(): Promise<void> {
    await this.modalCancelButton.waitFor({ state: 'visible' });
    await this.modalCancelButton.click();
  }

  // ============================================================================
  // DROPDOWN HELPER METHODS
  // ============================================================================

  async openDropdown(dropdownLocator: Locator): Promise<void> {
    await dropdownLocator.waitFor({ state: 'visible' });
    await dropdownLocator.click();
  }

  async selectDropdownOption(optionValue: string): Promise<void> {
    const option = this.page.locator(`[data-attr-value="${optionValue}"]`);
    await option.waitFor({ state: 'visible' });
    await option.click();
  }

  async closeDropdown(): Promise<void> {
    await this.page.keyboard.press('Escape');
  }
}
```

---

## Key Pattern Rules

1. **All locators are `readonly` class properties** - Not getters
2. **Locators are initialized in constructor** - Not inline in methods
3. **Every interaction method has `waitFor` before action** - Ensures element is ready
4. **Use `.catch(() => false)` for visibility checks** - Prevents test failures on missing elements
5. **Grid/Table patterns** - Use `[row-index]`, `[col-id]` selectors for AG Grid
6. **Dropdown patterns** - Use `[data-attr-value]` for option selection

---

## Placeholder Replacement Guide

When generating from `playwright-cli snapshot` output, replace:

| Placeholder | Example Value |
|-------------|---------------|
| `<page-name>` | `user-list` (lowercase with hyphens) |
| `<PageName>` | `UserList` (PascalCase) |
| `#<button-id>` | `#create-btn` (from snapshot) |
| `#<search-id>` | `#search` (from snapshot) |
| `.<grid-class>` | `.ag-root` (from snapshot) |

---

## Complete Page Object Example with All Methods

All interaction logic belongs in the Page Object class - no separate helpers needed:

```typescript
import { Page, Locator, expect } from '@playwright/test';

export class FeaturePage {
  readonly page: Page;

  // ============================================================================
  // LOCATORS
  // ============================================================================

  readonly settingsTab: Locator;
  readonly stylesTab: Locator;
  readonly linkInput: Locator;
  readonly linkTypeCombobox: Locator;
  readonly alignmentButtons: Locator;
  readonly backgroundColorPicker: Locator;
  readonly fontFamilyDropdown: Locator;
  readonly previewButton: Locator;
  readonly featureBlock: Locator;
  readonly dropTarget: Locator;

  constructor(page: Page) {
    this.page = page;

    this.settingsTab = page.getByRole('tab', { name: 'Settings' });
    this.stylesTab = page.getByRole('tab', { name: 'Styles' });
    this.linkInput = page.getByRole('textbox', { name: 'Link' });
    this.linkTypeCombobox = page.getByRole('combobox');
    this.alignmentButtons = page.locator('.alignment-buttons button');
    this.backgroundColorPicker = page.locator('.color-picker');
    this.fontFamilyDropdown = page.locator('.font-family-dropdown');
    this.previewButton = page.locator('#preview-button');
    this.featureBlock = page.locator('[data-block-type="feature"]');
    this.dropTarget = page.frameLocator('iframe').locator('.drop-target').first();
  }

  // ============================================================================
  // NAVIGATION
  // ============================================================================

  async goto(url: string = '/'): Promise<void> {
    await this.page.goto(url);
    await this.page.waitForLoadState('networkidle');
  }

  async navigateToTab(tabName: 'Settings' | 'Styles'): Promise<void> {
    const tab = tabName === 'Settings' ? this.settingsTab : this.stylesTab;
    await tab.waitFor({ state: 'visible' });
    await tab.click();
    await this.page.waitForLoadState('domcontentloaded');
  }

  // ============================================================================
  // SETUP METHODS
  // ============================================================================

  async setupFeatureBlock(): Promise<void> {
    await this.featureBlock.waitFor({ timeout: 30000 });
    await this.featureBlock.hover();
    await this.page.mouse.down();
    await this.dropTarget.waitFor({ state: 'visible' });
    await this.dropTarget.hover();
    await this.page.mouse.up();
    await this.dropTarget.waitFor({ state: 'attached' });
  }

  async setupAndNavigateToSettings(): Promise<void> {
    await this.setupFeatureBlock();
    await this.navigateToTab('Settings');
  }

  async setupAndNavigateToStyles(): Promise<void> {
    await this.setupFeatureBlock();
    await this.navigateToTab('Styles');
  }

  async setupDemoFeature(): Promise<void> {
    await this.setupFeatureBlock();
    await this.navigateToTab('Styles');
    await this.setBackgroundColor();
    await this.setFontFamily();
  }

  // ============================================================================
  // SETTINGS INTERACTIONS
  // ============================================================================

  async setLinkValue(value: string): Promise<void> {
    await this.linkInput.waitFor({ state: 'visible' });
    await this.linkInput.fill(value);
    await this.linkInput.blur();
  }

  async configureLinkType(linkType: string): Promise<void> {
    await this.linkTypeCombobox.waitFor({ state: 'visible' });
    await this.linkTypeCombobox.click();
    const option = this.page.locator('a').filter({ hasText: linkType });
    await option.waitFor({ state: 'visible' });
    await option.click();
  }

  async setEmailConfiguration(email: string, subject: string, body: string): Promise<void> {
    await this.page.getByRole('textbox', { name: 'Email' }).fill(email);
    await this.page.getByRole('textbox', { name: 'Subject' }).fill(subject);
    await this.page.getByRole('textbox', { name: 'Body' }).fill(body);
  }

  async setAlignment(index: number): Promise<void> {
    const button = this.alignmentButtons.nth(index);
    await button.waitFor({ state: 'visible' });
    await button.click();
  }

  // ============================================================================
  // STYLES INTERACTIONS
  // ============================================================================

  async setBackgroundColor(): Promise<void> {
    await this.backgroundColorPicker.waitFor({ state: 'visible' });
    await this.backgroundColorPicker.click();
  }

  async setFontFamily(): Promise<void> {
    await this.fontFamilyDropdown.waitFor({ state: 'visible' });
    await this.fontFamilyDropdown.click();
    const option = this.page.locator('.font-option').first();
    await option.waitFor({ state: 'visible' });
    await option.click();
  }

  // ============================================================================
  // PREVIEW INTERACTIONS
  // ============================================================================

  async previewChanges(): Promise<void> {
    await this.previewButton.waitFor({ state: 'visible' });
    await this.previewButton.click();
    await this.page.waitForLoadState('domcontentloaded');
  }

  async clickPreviewElement(text: string): Promise<void> {
    const element = this.page
      .locator('iframe')
      .first()
      .contentFrame()
      .locator('a')
      .filter({ hasText: new RegExp(`^${text}$`) });
    await element.click();
  }

  // ============================================================================
  // WAIT UTILITIES (included in Page Object, not separate helper)
  // ============================================================================

  async waitForPageLoad(): Promise<void> {
    await this.page.waitForLoadState('networkidle');
  }

  async waitForLoadingToComplete(): Promise<void> {
    const spinner = this.page.locator('.loading-spinner');
    const isLoading = await spinner.isVisible().catch(() => false);
    if (isLoading) {
      await spinner.waitFor({ state: 'hidden', timeout: 30000 });
    }
  }
}
```
