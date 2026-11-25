# add-data-testids

An AI skill that finds UI elements missing `data-testid` attributes and adds them with consistent, stable names, so your Playwright, Cypress, Selenium or Testing Library tests have reliable selectors. Use it when a page or component isn't ready for automation, or ask it to just *check* and it will list what's missing without editing anything.

It follows your project's existing conventions, never renames IDs your tests already use, handles lists and conditional rendering, and catches custom components that silently drop the attribute.

## Example

**Input:** "Add data-testids to CartPanel.jsx." The component has one existing ID, a list rendered with `.map`, conditional branches, and a custom `Button` that does **not** pass extra props down, so any test ID put on it would never reach the page.

```jsx
// CartPanel.jsx (before)
import { useState } from "react";
import Button from "./Button";

export default function CartPanel({ items, onRemove, onApplyPromo, onCheckout }) {
  const [promo, setPromo] = useState("");
  const [error, setError] = useState(null);

  const total = items.reduce((sum, i) => sum + i.price * i.qty, 0);

  return (
    <div className="cart-panel">
      <h2>Your cart</h2>
      {items.length === 0 ? (
        <p className="empty">Your cart is empty.</p>
      ) : (
        <ul>
          {items.map((item) => (
            <li key={item.sku} className="cart-row">
              <span>{item.name}</span>
              <span>${item.price.toFixed(2)}</span>
              <button onClick={() => onRemove(item.sku)} aria-label="Remove item">
                ✕
              </button>
            </li>
          ))}
        </ul>
      )}
      <div className="promo">
        <input
          value={promo}
          onChange={(e) => setPromo(e.target.value)}
          placeholder="Promo code"
        />
        <Button onClick={() => onApplyPromo(promo).catch(setError)}>Apply</Button>
      </div>
      {error && <div className="error">{error.message}</div>}
      <p className="total" data-testid="cart-total">${total.toFixed(2)}</p>
      <Button primary onClick={onCheckout}>Checkout</Button>
    </div>
  );
}

// Button.jsx (before)
export default function Button({ primary, onClick, children }) {
  return (
    <button className={primary ? "btn btn-primary" : "btn"} onClick={onClick}>
      {children}
    </button>
  );
}
```

**Output:** only `data-testid` attributes were added. No other line changed.

```jsx
// CartPanel.jsx (after)
import { useState } from "react";
import Button from "./Button";

export default function CartPanel({ items, onRemove, onApplyPromo, onCheckout }) {
  const [promo, setPromo] = useState("");
  const [error, setError] = useState(null);

  const total = items.reduce((sum, i) => sum + i.price * i.qty, 0);

  return (
    <div className="cart-panel" data-testid="cart-panel">
      <h2 data-testid="cart-panel-heading">Your cart</h2>
      {items.length === 0 ? (
        <p className="empty" data-testid="cart-panel-empty-message">Your cart is empty.</p>
      ) : (
        <ul data-testid="cart-panel-item-list">
          {items.map((item) => (
            <li key={item.sku} className="cart-row" data-testid={`cart-panel-item-${item.sku}`}>
              <span data-testid="cart-panel-item-name">{item.name}</span>
              <span data-testid="cart-panel-item-price">${item.price.toFixed(2)}</span>
              <button onClick={() => onRemove(item.sku)} aria-label="Remove item" data-testid="cart-panel-remove-button">
                ✕
              </button>
            </li>
          ))}
        </ul>
      )}
      <div className="promo">
        <input
          value={promo}
          onChange={(e) => setPromo(e.target.value)}
          placeholder="Promo code"
          data-testid="cart-panel-promo-input"
        />
        <Button onClick={() => onApplyPromo(promo).catch(setError)} data-testid="cart-panel-apply-promo-button">Apply</Button>
      </div>
      {error && <div className="error" data-testid="cart-panel-error-message">{error.message}</div>}
      <p className="total" data-testid="cart-total">${total.toFixed(2)}</p>
      <Button primary onClick={onCheckout} data-testid="cart-panel-checkout-button">Checkout</Button>
    </div>
  );
}

// Button.jsx (after: now forwards the test ID to the real <button>)
export default function Button({ primary, onClick, children, "data-testid": testId }) {
  return (
    <button className={primary ? "btn btn-primary" : "btn"} onClick={onClick} data-testid={testId}>
      {children}
    </button>
  );
}
```

**Report the skill returned:**

> **Summary:** 2 files scanned · 1 element already had an ID · 12 IDs added · 1 item flagged
>
> | File | Line | Element | data-testid |
> |---|---|---|---|
> | CartPanel.jsx | 11 | root `div` | `cart-panel` |
> | CartPanel.jsx | 12 | `h2` heading | `cart-panel-heading` |
> | CartPanel.jsx | 14 | empty-cart message | `cart-panel-empty-message` |
> | CartPanel.jsx | 16 | `ul` item list | `cart-panel-item-list` |
> | CartPanel.jsx | 18 | `li` item (repeated) | `` `cart-panel-item-${item.sku}` `` |
> | CartPanel.jsx | 19 | item name | `cart-panel-item-name` |
> | CartPanel.jsx | 20 | item price | `cart-panel-item-price` |
> | CartPanel.jsx | 21 | remove `button` | `cart-panel-remove-button` |
> | CartPanel.jsx | 33 | promo `input` | `cart-panel-promo-input` |
> | CartPanel.jsx | 35 | Apply `Button` | `cart-panel-apply-promo-button` |
> | CartPanel.jsx | 37 | error message | `cart-panel-error-message` |
> | CartPanel.jsx | 39 | Checkout `Button` | `cart-panel-checkout-button` |
>
> **Flagged for review:** `Button.jsx` didn't pass extra props to its `<button>`, so the two IDs above would never have reached the page. I changed it to forward only `data-testid`. Other components use `Button`, so give this a quick look. The existing `cart-total` ID was kept as is.
>
> **Selectors to copy (Playwright):**
> ```js
> const row = page.getByTestId('cart-panel-item-A1');
> await row.getByTestId('cart-panel-remove-button').click();
> await page.getByTestId('cart-panel-promo-input').fill('SAVE10');
> await page.getByTestId('cart-panel-checkout-button').click();
> ```

## Install

**Claude Code:** copy the `add-data-testids` folder into `~/.claude/skills/` (available in every project) or into your project's `.claude/skills/` folder. Restart Claude Code if this is your first skill. Then ask in your own words ("add test IDs to the checkout page") or call it with `/add-data-testids`.

**Claude app:** zip the `add-data-testids` folder and upload it under **Settings → Capabilities → Skills**. Then paste a component and ask to add test IDs.
