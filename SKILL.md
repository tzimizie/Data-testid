---
name: add-data-testids
description: Audit front-end UI code for missing data-testid attributes and add them automatically with consistent, stable names. Use when someone wants to add test IDs or data-testids, make a page or component testable, prepare UI for Playwright, Cypress, Selenium or Testing Library automation, or check which elements are missing test IDs.
---

# Add data-testids

Give automated tests stable selectors by finding elements that lack a test ID and adding one, without changing anything else in the code.

There are two modes:
- **Add mode (default):** add the missing IDs and report what changed.
- **Check mode:** if the user only asks to check, audit or list missing IDs, edit nothing. Return the report with a suggested ID for each element.

## 1. Find the scope

- If the user names files, folders or pasted code, work only on those.
- If they ask for the whole project, scan the UI source folders (such as `src/`, `app/`, `components/`, `pages/`).
- Skip: `node_modules`, build output (`dist`, `build`, `.next`, `coverage`), generated or vendored files, test files (`*.test.*`, `*.spec.*`, `__tests__`, `cypress/`, `e2e/`), Storybook stories and snapshots.
- Supported: JSX/TSX (React, Preact, Solid), Vue templates, Svelte, Angular templates and plain HTML.
- If code is pasted into the chat instead of files, return the full updated code followed by the report.

## 2. Learn the project's convention before adding anything

- Search for existing test attributes: `data-testid`, `data-test-id`, `data-test`, `data-cy`, `data-qa`.
- Check test config for a custom attribute: Playwright `testIdAttribute` in `playwright.config.*`, or Testing Library `configure({ testIdAttribute })`.
- If the project consistently uses a different attribute, use that one instead and say so in the report.
- If at least 3 existing IDs share a naming pattern (case style, prefixes, word order), follow it. Otherwise use the default convention in step 4.
- Never rename, remove or reformat an existing test ID. Tests already depend on it.

## 3. Decide which elements get an ID

**Add an ID to:**
- The root element of each component, named after the component, so tests can scope to it.
- Interactive elements: `button`, `a`, `input`, `select`, `textarea`, `form`, and anything with a click handler (`onClick`, `@click`, `(click)`, `on:click`) or an interactive role (`button`, `link`, `tab`, `menuitem`, `checkbox`, `switch`, `dialog`).
- Content that tests assert on: page and section headings, error, success and validation messages, alerts and toasts, empty states, loading indicators, modals, tables and their rows, lists and their items, and displayed values such as item names, totals, counts and prices.

**Do not add an ID to:**
- Wrappers used only for layout or styling, icons, decorative images, or fragments that render no DOM element.
- Elements that already have a test ID.

**Custom components:** a test ID on `<MyButton>` only works if the component passes it down to a DOM element. Open the component's definition and check:
- If it spreads props (`{...props}`, `{...rest}`, `v-bind="$attrs"`) onto a DOM element, it works as is.
- If it doesn't, make the smallest change that forwards only the test ID (for example, destructure `"data-testid": testId` and put `data-testid={testId}` on its root element). List this change under "Flagged" in the report, since other code uses the same component.
- For third-party library components (MUI, Chakra, Ant Design and similar), add the ID and flag it as "verify it reaches the DOM."

## 4. Name each ID

Default format, in kebab-case: `<component>-<purpose>-<type>`, for example `login-form-submit-button`, `checkout-email-input`, `cart-panel-error-message`.

- **component:** the component or file name in kebab-case (`CartPanel.jsx` → `cart-panel`).
- **purpose:** what the element does, taken from its `aria-label`, `name`, `placeholder`, label, handler name (`handleDelete` → `delete`) or short visible text. Keep it to 1–3 words. Don't copy long display text, which changes and gets translated.
- **type:** `button`, `link`, `input`, `select`, `checkbox`, `radio`, `textarea`, `toggle`, `tab`, `form`, `modal`, `heading`, `message`, `list`, `item`, `row`, `table`. For a displayed value such as a name, price or total, the purpose alone is enough (`cart-panel-item-price`).
- Every ID must be unique within its component. If two would collide, add context (`header-`, `footer-`), not numbers.

**Repeated elements** (`.map`, `v-for`, `*ngFor`, `{#each}`):
- Give the list container a static ID (`cart-panel-item-list`).
- Give each repeated item a dynamic ID built from a stable key, such as an ID or SKU: ``data-testid={`cart-panel-item-${item.sku}`}``.
- Give elements inside a repeated item static IDs (`cart-panel-remove-button`). Tests reach them by scoping through the item.
- Use the array index only if there is no stable key, and flag it, because index IDs shift when items are added or removed.
- Never build an ID from user content such as names or emails.

**Conditional rendering:** if two branches render the same element, give them the same ID. Different states get different IDs (`login-error-message`, `login-success-message`).

## 5. Edit safely

- Add the attribute and change nothing else: no reformatting, reordering, renaming or logic changes.
- Add it as the last attribute on the opening tag, matching the file's quote style and line wrapping.
- Syntax for dynamic IDs:
  - JSX: ``data-testid={`cart-panel-item-${item.sku}`}``
  - Vue: ``:data-testid="`cart-panel-item-${item.sku}`"``
  - Angular: `[attr.data-testid]="'cart-panel-item-' + item.sku"`
  - Svelte: `data-testid="cart-panel-item-{item.sku}"`
- After editing real files, run the project's lint or type-check script if one exists (for example `npm run lint` or `npx tsc --noEmit`) and report the result. Don't fix unrelated warnings.
- If the build strips test attributes in production (for example `babel-plugin-react-remove-properties`), mention it in the report. It's expected and fine.

## 6. Report

End every run with:

1. **Summary:** files scanned, elements that already had IDs, IDs added, items flagged.
2. **Table of changes:** `File | Line | Element | data-testid`. In check mode, the last column is the suggested ID.
3. **Flagged for review:** components changed to forward the ID, third-party components to verify, index-based IDs, and any name you weren't confident about.
4. **Selectors to copy:** a few of the new IDs written for the test framework found in `package.json` (`@playwright/test`, `cypress`, `selenium-webdriver`, `webdriverio`, `@testing-library/*`). If none is found, use Playwright.
   - Playwright: `page.getByTestId('cart-panel-checkout-button')`
   - Cypress: `cy.get('[data-testid="cart-panel-checkout-button"]')`
   - Selenium: `driver.findElement(By.cssSelector("[data-testid='cart-panel-checkout-button']"))`
   - Testing Library: `screen.getByTestId('cart-panel-checkout-button')`
