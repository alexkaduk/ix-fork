# Testing Non-Reflected Properties in IX Components

## Quick Answer

**Q: Why can't I test the `icon` property using `getAttribute('icon')`?**

**A:** The `icon` property is not reflected to a DOM attribute for performance reasons. Test it using the property directly instead:

```tsx
// ❌ Won't work
expect(button.getAttribute('icon')).toBe('check');

// ✅ Works
expect(button.icon).toBe('check');
```

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
console.log(button.disabled);  // true
console.log(button.icon);      // 'check'

// Only reflected properties work as attributes
console.log(button.getAttribute('disabled'));  // 'true'
console.log(button.getAttribute('icon'));      // null ❌
```

---

## How to Test Non-Reflected Properties

### 1. **Jest + DOM Testing Library**

```tsx
import { render, screen } from '@testing-library/dom';

test('button has correct icon', () => {
  document.body.innerHTML = '<ix-button icon="check">Save</ix-button>';
  
  const button = screen.getByRole('button');
  
  // Option 1: Direct property
  expect(button.icon).toBe('check');
  
  // Option 2: toHaveProperty matcher
  expect(button).toHaveProperty('icon', 'check');
});
```

---

### 2. **React Testing Library**

```tsx
import { render, screen } from '@testing-library/react';
import { IxButton } from '@siemens/ix-react';

test('button has correct icon', () => {
  render(<IxButton icon="check">Save</IxButton>);
  
  const button = screen.getByRole('button');
  
  // Access the underlying web component element
  expect(button).toHaveProperty('icon', 'check');
});
```

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
    props: { icon: 'check' }
  });
  
  const button = container.querySelector('ix-button');
  expect(button.icon).toBe('check');
});
```

---

### 5. **Playwright (E2E Testing)**

```typescript
import { test, expect } from '@playwright/test';

test('button has correct icon', async ({ page }) => {
  await page.goto('/your-page');
  
  const button = page.locator('ix-button');
  
  // Evaluate property in browser context
  const icon = await button.evaluate((el: HTMLIxButtonElement) => el.icon);
  expect(icon).toBe('check');
  
  // Or use handle
  const iconProperty = await button.evaluateHandle(el => el.icon);
  expect(await iconProperty.jsonValue()).toBe('check');
});
```

---

### 6. **Cypress**

```typescript
describe('Button Component', () => {
  it('should have correct icon', () => {
    cy.visit('/your-page');
    
    cy.get('ix-button').then($button => {
      // Access property via jQuery element
      expect($button[0].icon).to.equal('check');
    });
  });
});
```

---

## Visual Testing via Shadow DOM

You can also test by accessing the rendered icon element:

```tsx
test('icon is rendered in button', () => {
  const button = screen.getByRole('button');
  
  // Access shadow DOM
  const shadowRoot = button.shadowRoot;
  const iconElement = shadowRoot.querySelector('ix-icon');
  
  // Verify icon exists
  expect(iconElement).toBeInTheDocument();
  
  // Verify icon name (if ix-icon exposes it)
  expect(iconElement).toHaveAttribute('name', 'check');
});
```

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
// ❌ This fails
test('button icon attribute', () => {
  const button = document.querySelector('ix-button[icon="check"]');
  expect(button).toBeInTheDocument();  // Selector won't find it!
});

// ✅ This works
test('button icon property', () => {
  const button = document.querySelector('ix-button');
  expect(button.icon).toBe('check');
});
```

---

### Scenario 2: Testing Variant

```tsx
// ❌ This fails
expect(button.getAttribute('variant')).toBe('primary');

// ✅ This works
expect(button.variant).toBe('primary');
```

---

### Scenario 3: Testing Loading State

```tsx
// ❌ This fails
expect(button.getAttribute('loading')).toBe('true');

// ✅ This works
expect(button.loading).toBe(true);
```

---

### Scenario 4: Testing Disabled State (Reflected!)

```tsx
// ✅ Both work (disabled IS reflected)
expect(button.getAttribute('disabled')).toBe('true');
expect(button.disabled).toBe(true);

// CSS selector also works
const disabledButton = document.querySelector('ix-button[disabled]');
```

---

## TypeScript Support

If using TypeScript, make sure to import the types:

```typescript
import { HTMLIxButtonElement } from '@siemens/ix';

const button = document.querySelector('ix-button') as HTMLIxButtonElement;

// TypeScript knows about all properties
expect(button.icon).toBe('check');        // ✅ Type-safe
expect(button.variant).toBe('primary');   // ✅ Type-safe
expect(button.disabled).toBe(false);      // ✅ Type-safe
```

---

## Custom Test Utilities

Create helper functions for common tests:

```typescript
// test-utils.ts
export function getIxButtonProperty<K extends keyof HTMLIxButtonElement>(
  button: HTMLIxButtonElement,
  property: K
): HTMLIxButtonElement[K] {
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
      message: () => 
        `Expected icon to be ${expected}, but got ${actual}`,
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
const button = document.querySelector('ix-button');

console.log('icon' in button);              // true
console.log(button.hasAttribute('icon'));   // false
console.log(button.icon);                   // 'check'
console.log(button.getAttribute('icon'));   // null
```

### List All Properties

```typescript
const button = document.querySelector('ix-button');

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
- Test properties directly: `button.icon`
- Use `.toHaveProperty()` matcher
- Access via shadow DOM for visual tests
- Check documentation for which properties are reflected

❌ **DON'T:**
- Test non-reflected properties via `getAttribute()`
- Assume all properties are attributes
- Try to use CSS attribute selectors on non-reflected properties

### When in Doubt

```typescript
// This ALWAYS works for any property
expect(element.propertyName).toBe(expectedValue);

// This ONLY works for reflected properties
expect(element.getAttribute('property-name')).toBe(expectedValue);
```

---

## Need Help?

- Read: [REFLECT_PHILOSOPHY.md](./REFLECT_PHILOSOPHY.md)
- Check: Component API documentation
- Ask: [GitHub Discussions](https://github.com/siemens/ix/discussions)
- Report: [GitHub Issues](https://github.com/siemens/ix/issues)
