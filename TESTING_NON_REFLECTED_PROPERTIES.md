# Testing Non-Reflected Properties in IX Components

## Quick Answer

**Q: Why does my test fail with `Property 'icon' does not exist on type 'HTMLElement'`?**

**A:** You need to cast the element to the correct type in TypeScript. Also, icons are SVG data URLs, not string identifiers.

### ❌ Failing Test

```tsx
import { IxButton } from '@siemens/ix-react';
import { iconStar } from '@siemens/ix-icons/icons';
import { render, screen } from '@testing-library/react';

test('button icon', () => {
  render(<IxButton icon={iconStar} data-testid="btn" />);
  const button = screen.getByTestId('btn');
  expect(button.icon).toBe('star'); // ❌ Two errors!
  // 1. Property 'icon' does not exist on type 'HTMLElement'
  // 2. Icon value is not 'star', it's a data URL
});
```

### ✅ Fixed Test

```tsx
import { IxButton } from '@siemens/ix-react';
import { iconStar } from '@siemens/ix-icons/icons';
import { render, screen } from '@testing-library/react';
import type { HTMLIxButtonElement } from '@siemens/ix';

test('button icon', () => {
  render(<IxButton icon={iconStar} data-testid="btn" />);

  // Cast to the correct type
  const button = screen.getByTestId('btn') as HTMLIxButtonElement;

  // Compare against the imported icon constant
  expect(button.icon).toBe(iconStar); // ✅ Works!
});
```

### Key Points

1. **TypeScript Type Casting:** `as HTMLIxButtonElement`
2. **Icons are Data URLs:** Compare against imported constants, not strings
3. **Non-Reflected Property:** Use `.icon` property, not `getAttribute('icon')`

---

## Understanding Reflected vs Non-Reflected Properties

### What's the Difference?

```tsx
// REFLECTED property (syncs to DOM attribute)
@Prop({ reflect: true }) disabled = false;

// NON-REFLECTED property (property only)
@Prop() icon = 'check';
```

### In the DOM:

```html
<!-- Reflected properties are visible as attributes -->
<ix-button disabled="true">
  <!-- icon is NOT visible here -->
</ix-button>
```

### In JavaScript:

```tsx
const button = document.querySelector('ix-button');

// Both work as properties
console.log(button.disabled); // true
console.log(button.icon); // 'check'

// Only reflected properties work as attributes
console.log(button.getAttribute('disabled')); // 'true'
console.log(button.getAttribute('icon')); // null ❌
```

---

## How to Test Non-Reflected Properties

### 1. **Jest + DOM Testing Library**

```tsx
import { render, screen } from '@testing-library/dom';
import { iconCheck } from '@siemens/ix-icons/icons';

test('button has correct icon', () => {
  const button = document.createElement('ix-button');
  button.icon = iconCheck;
  button.textContent = 'Save';
  document.body.appendChild(button);

  // Option 1: Direct property comparison
  expect(button.icon).toBe(iconCheck);

  // Option 2: toHaveProperty matcher
  expect(button).toHaveProperty('icon', iconCheck);

  // Option 3: Just verify icon is set
  expect(button.icon).toBeTruthy();
  expect(button.icon).toContain('data:image/svg+xml');
});
```

---

### 2. **React Testing Library**

```tsx
import { render, screen } from '@testing-library/react';
import { IxButton } from '@siemens/ix-react';
import { iconStar } from '@siemens/ix-icons/icons';
import type { HTMLIxButtonElement } from '@siemens/ix';

test('button has correct icon', () => {
  render(
    <IxButton icon={iconStar} data-testid="my-button">
      Save
    </IxButton>
  );

  // TypeScript: Cast to the correct element type
  const button = screen.getByTestId('my-button') as HTMLIxButtonElement;

  // ⚠️ Icons from @siemens/ix-icons are SVG data URLs, not strings!
  // Option 1: Compare against the imported icon constant
  expect(button.icon).toBe(iconStar);

  // Option 2: Just verify the icon property is set
  expect(button.icon).toBeTruthy();
  expect(button.icon).toContain('data:image/svg+xml');

  // Option 3: Verify the icon property is accessible (simplest)
  expect(button).toHaveProperty('icon', iconCheck);
});
```

**Important:**

- Icons imported from `@siemens/ix-icons/icons` are SVG data URLs (e.g., `"data:image/svg+xml;utf8,..."`), not string identifiers like `"star"`. Always compare against the imported constant.
- **TypeScript**: Cast to `HTMLIxButtonElement` to access the `icon` property: `as HTMLIxButtonElement`

---

### 3. **Angular Testing**

```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { IxButtonModule } from '@siemens/ix-angular';

describe('ButtonComponent', () => {
  let fixture: ComponentFixture<TestComponent>;

  it('should have correct icon', () => {
    const compiled = fixture.nativeElement;
    const button = compiled.querySelector('ix-button');

    // Access property directly
    expect(button.icon).toBe('check');
  });
});
```

---

### 4. **Vue Testing Library**

```typescript
import { render } from '@testing-library/vue';
import { IxButton } from '@siemens/ix-vue';

test('button has correct icon', () => {
  const { container } = render(IxButton, {
    props: { icon: 'check' },
  });

  const button = container.querySelector('ix-button');
  expect(button.icon).toBe('check');
});
```

---

### 5. **Playwright (E2E Testing)**

```typescript
import { test, expect } from '@playwright/test';
import { iconCheck } from '@siemens/ix-icons/icons';

test('button has correct icon', async ({ page }) => {
  await page.goto('/your-page');

  const button = page.locator('ix-button');

  // Evaluate property in browser context
  const icon = await button.evaluate((el: HTMLIxButtonElement) => el.icon);
  expect(icon).toBe(iconCheck); // Compare against constant

  // Or just verify it's set
  expect(icon).toBeTruthy();
  expect(icon).toContain('data:image/svg+xml');
});
```

---

### 6. **Cypress**

```typescript
import { iconCheck } from '@siemens/ix-icons/icons';

describe('Button Component', () => {
  it('should have correct icon', () => {
    cy.visit('/your-page');

    cy.get('ix-button').then(($button) => {
      // Access property via jQuery element
      const button = $button[0] as HTMLIxButtonElement;
      expect(button.icon).to.equal(iconCheck);
    });
  });
});
```

---

## Visual Testing via Shadow DOM

⚠️ **Warning:** Testing shadow DOM internals is **NOT RECOMMENDED**. It couples your tests to implementation details that can change between versions.

### Why You Shouldn't Test Shadow DOM

1. **Internal structure may change** - Component internals are not part of the public API
2. **Selectors are fragile** - What works today may break tomorrow
3. **No benefit** - Testing the property is simpler and more reliable

### The Right Approach

```tsx
import type { HTMLIxButtonElement } from '@siemens/ix';
import { iconCheck } from '@siemens/ix-icons/icons';

test('button has icon', () => {
  const button = screen.getByRole('button') as HTMLIxButtonElement;

  // ✅ Test the public API (property)
  expect(button.icon).toBe(iconCheck);

  // ❌ Don't test shadow DOM internals
  // const shadowRoot = button.shadowRoot;
  // const iconElement = shadowRoot?.querySelector('.some-internal-class');
  // This breaks when internal structure changes!
});
```

### If You Really Must Test Shadow DOM

Only do this for **visual regression testing** or **integration tests**, never for unit tests:

```tsx
test('button renders without crashing', () => {
  const button = screen.getByRole('button') as HTMLIxButtonElement;

  // Just verify shadow DOM exists
  expect(button.shadowRoot).toBeTruthy();

  // Don't query internal elements - they're implementation details
});
```

**Recommendation:** Use property testing (Options 1 & 2) instead. Test the component's public API, not its internals.

---

## Which Properties Are Reflected?

### ✅ Reflected Properties (Test via Attribute or Property)

```tsx
// Form-related
disabled, checked, required, name, value

// Common examples
<ix-button disabled="true">        // ✅ getAttribute('disabled') works
<ix-checkbox checked="true">       // ✅ getAttribute('checked') works
<ix-input required="true">         // ✅ getAttribute('required') works
```

### ❌ Non-Reflected Properties (Test via Property Only)

```tsx
// Configuration
icon, variant, size, type, alignment

// Text content
label, placeholder, helperText, errorText

// Internal state
loading, hideText

// Common examples
<ix-button icon="check">           // ❌ getAttribute('icon') returns null
<ix-button variant="primary">      // ❌ getAttribute('variant') returns null
<ix-input placeholder="Enter...">  // ❌ getAttribute('placeholder') returns null
```

---

## Common Scenarios

### Scenario 1: Testing Button Icon

```tsx
import { iconCheck } from '@siemens/ix-icons/icons';
import type { HTMLIxButtonElement } from '@siemens/ix';

// ❌ This fails - can't select by icon attribute
test('button icon attribute', () => {
  const button = document.querySelector('ix-button[icon]');
  expect(button).toBeInTheDocument(); // Selector won't find it!
});

// ✅ This works - test the property
test('button icon property', () => {
  const button = document.querySelector('ix-button') as HTMLIxButtonElement;

  // Compare against the imported icon constant
  expect(button.icon).toBe(iconCheck);

  // Or just verify an icon is set
  expect(button.icon).toBeTruthy();
  expect(button.icon).toContain('data:image/svg+xml');
});
```

---

### Scenario 2: Testing Variant

```tsx
import type { HTMLIxButtonElement } from '@siemens/ix';

// ❌ This fails
const button = document.querySelector('ix-button') as HTMLIxButtonElement;
expect(button.getAttribute('variant')).toBe('primary');

// ✅ This works
const button = document.querySelector('ix-button') as HTMLIxButtonElement;
expect(button.variant).toBe('primary');
```

---

### Scenario 3: Testing Loading State

```tsx
import type { HTMLIxButtonElement } from '@siemens/ix';

// ❌ This fails
const button = document.querySelector('ix-button') as HTMLIxButtonElement;
expect(button.getAttribute('loading')).toBe('true');

// ✅ This works
const button = document.querySelector('ix-button') as HTMLIxButtonElement;
expect(button.loading).toBe(true);
```

---

### Scenario 4: Testing Disabled State (Reflected!)

```tsx
import type { HTMLIxButtonElement } from '@siemens/ix';

// ✅ Both work (disabled IS reflected)
const button = document.querySelector('ix-button') as HTMLIxButtonElement;
expect(button.getAttribute('disabled')).toBe('true');
expect(button.disabled).toBe(true);

// CSS selector also works
const disabledButton = document.querySelector('ix-button[disabled]') as HTMLIxButtonElement;
```

---

## TypeScript Support

**Always cast to the correct element type** to access component properties:

```typescript
import type { HTMLIxButtonElement } from '@siemens/ix';
import { iconCheck } from '@siemens/ix-icons/icons';

// ✅ Cast the element to the correct type
const button = document.querySelector('ix-button') as HTMLIxButtonElement;

// Now TypeScript knows about all properties
expect(button.icon).toBe(iconCheck); // ✅ Type-safe
expect(button.variant).toBe('primary'); // ✅ Type-safe
expect(button.disabled).toBe(false); // ✅ Type-safe
```

**For React Testing Library:**

```typescript
import { render, screen } from '@testing-library/react';
import type { HTMLIxButtonElement } from '@siemens/ix';
import { iconStar } from '@siemens/ix-icons/icons';

const button = screen.getByTestId('my-button') as HTMLIxButtonElement;
expect(button.icon).toBe(iconStar);
```

**Without the type cast, you'll get TypeScript errors:**

```typescript
// ❌ TypeScript Error: Property 'icon' does not exist on type 'HTMLElement'
const button = screen.getByTestId('my-button');
expect(button.icon).toBe(iconStar); // Error!

// ✅ Fixed with type cast
const button = screen.getByTestId('my-button') as HTMLIxButtonElement;
expect(button.icon).toBe(iconStar); // Works!
```

---

## Custom Test Utilities

Create helper functions for common tests:

```typescript
// test-utils.ts
export function getIxButtonProperty<K extends keyof HTMLIxButtonElement>(button: HTMLIxButtonElement, property: K): HTMLIxButtonElement[K] {
  return button[property];
}

// Usage
const icon = getIxButtonProperty(button, 'icon');
expect(icon).toBe('check');
```

Or create custom matchers:

```typescript
// custom-matchers.ts
expect.extend({
  toHaveIconProperty(received: Element, expected: string) {
    const actual = (received as any).icon;
    const pass = actual === expected;

    return {
      pass,
      message: () => `Expected icon to be ${expected}, but got ${actual}`,
    };
  },
});

// Usage
expect(button).toHaveIconProperty('check');
```

---

## Debugging Tips

### Check if Property Exists

```tsx
import type { HTMLIxButtonElement } from '@siemens/ix';
import { iconCheck } from '@siemens/ix-icons/icons';

const button = document.querySelector('ix-button') as HTMLIxButtonElement;

console.log('icon' in button); // true
console.log(button.hasAttribute('icon')); // false
console.log(button.icon); // "data:image/svg+xml;utf8,<?xml..."
console.log(button.icon === iconCheck); // true
console.log(button.getAttribute('icon')); // null
```

### List All Properties

```typescript
import type { HTMLIxButtonElement } from '@siemens/ix';

const button = document.querySelector('ix-button') as HTMLIxButtonElement;

// Log all enumerable properties
console.log(Object.keys(button));

// Check specific property
console.log(Object.getOwnPropertyDescriptor(button, 'icon'));
```

---

## Why This Design?

See [REFLECT_PHILOSOPHY.md](./REFLECT_PHILOSOPHY.md) for the full explanation.

---

## Summary

### Quick Checklist

✅ **DO:**

- **Cast to correct type in TypeScript:** `as HTMLIxButtonElement`
- Test properties directly: `button.icon`
- Use `.toHaveProperty()` matcher with correct values
- Compare icons against imported constants, not strings
- Access via shadow DOM for visual tests
- Check documentation for which properties are reflected

❌ **DON'T:**

- Test non-reflected properties via `getAttribute()`
- Assume all properties are attributes
- Try to use CSS attribute selectors on non-reflected properties
- Compare icon properties to strings like `'star'` (they are SVG data URLs!)

### When in Doubt

```typescript
import type { HTMLIxButtonElement } from '@siemens/ix';
import { iconCheck } from '@siemens/ix-icons/icons';

// ✅ This ALWAYS works for any property (with proper type cast)
const button = element as HTMLIxButtonElement;
expect(button.icon).toBe(iconCheck);

// ❌ This ONLY works for reflected properties
expect(element.getAttribute('icon')).toBe(iconCheck); // Won't work!
```

---

## Need Help?

- Read: [REFLECT_PHILOSOPHY.md](./REFLECT_PHILOSOPHY.md)
- Check: Component API documentation
- Ask: [GitHub Discussions](https://github.com/siemens/ix/discussions)
- Report: [GitHub Issues](https://github.com/siemens/ix/issues)
