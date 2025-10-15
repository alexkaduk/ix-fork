# The `reflect` Property Philosophy in Siemens IX

## Why This Document Exists

When building Web Components with Stencil, every `@Prop()` can optionally have `reflect: true`. This document clarifies **when to use reflection and why**, ensuring consistency across the IX design system for both contributors and consumers.

---

## The Problem

Without clear guidelines, developers face questions like:
- "Should this property be reflected to a DOM attribute?"
- "Why does `disabled` use `reflect: true` but `variant` doesn't?"
- "How do I know when reflection is needed?"

Inconsistent reflection leads to:
- ❌ Performance issues (unnecessary DOM updates)
- ❌ Security concerns (exposed sensitive data)
- ❌ Debugging confusion (invisible state)
- ❌ CSS styling problems (can't target attributes)

---

## What is `reflect`?

### Without `reflect` (Default: `false`)
```tsx
@Prop() disabled = false;
```

**Behavior:** Property → DOM attribute (one-way)
```html
<!-- HTML attribute sets property -->
<ix-button disabled="true"></ix-button>

<!-- JavaScript changes property -->
<script>
  button.disabled = false;
</script>

<!-- Attribute NOT updated in DOM -->
<ix-button disabled="true"></ix-button>  
<!-- ❌ Still shows "true" in DOM, but button.disabled === false -->
```

### With `reflect: true`
```tsx
@Prop({ reflect: true }) disabled = false;
```

**Behavior:** Property ↔ DOM attribute (two-way sync)
```html
<!-- HTML attribute sets property -->
<ix-button disabled="true"></ix-button>

<!-- JavaScript changes property -->
<script>
  button.disabled = false;
</script>

<!-- Attribute SYNCS automatically -->
<ix-button disabled="false"></ix-button>  
<!-- ✅ Updates in DOM to match property -->
```

---

## The Philosophy: "Reflect Only What Needs Observation"

### Core Principle
**Reflect properties only when external systems (CSS, browsers, DevTools, accessibility tools) need to observe their changes.**

Keep everything else private for **performance, security, and encapsulation**.

---

## When to Use `reflect: true` ✅

### 1. **Form-Associated Properties** (Web Standards)

Mirror native HTML form element behavior:

```tsx
// Matches <input name=""> behavior
@Prop({ reflect: true }) name?: string;

// Matches <input value=""> behavior  
@Prop({ reflect: true }) value: string;

// Matches <input checked> behavior
@Prop({ reflect: true }) checked = false;

// Matches <input disabled> behavior
@Prop({ reflect: true }) disabled = false;

// Matches <input required> behavior
@Prop({ reflect: true }) required = false;
```

**Why?**
- ✅ **Standards Compliance**: Follows HTML form specification
- ✅ **Form Validation**: Browsers read attributes for validation (`[required]`)
- ✅ **Framework Integration**: Angular, React forms expect attributes
- ✅ **Accessibility**: Screen readers detect `[disabled]`, `[required]`

**Real Example:**
```tsx
// checkbox.tsx
@Component({
  tag: 'ix-checkbox',
  formAssociated: true,
})
export class Checkbox {
  @Prop({ reflect: true }) name?: string;
  @Prop({ reflect: true }) value: string = 'on';
  @Prop({ reflect: true }) checked: boolean = false;
  @Prop({ reflect: true }) disabled: boolean = false;
  @Prop({ reflect: true }) required = false;
}
```

---

### 2. **CSS-Dependent State** (Styling Requirements)

Reflect when CSS selectors need to target the attribute:

```tsx
@Prop({ reflect: true }) disabled = false;
```

```scss
// CSS can style based on attribute
:host([disabled]) {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}

// Or attribute selectors
ix-button[disabled] {
  background-color: var(--theme-color-weak);
}
```

**Why?**
- ✅ **CSS Selectors**: Enables `[attribute]` selectors
- ✅ **State Visibility**: Visual state matches DOM state
- ✅ **Theme Consistency**: Theme CSS can target attributes

**Real Example:**
```tsx
// button.tsx
@Prop({ reflect: true }) disabled = false;

// toggle-button.tsx  
@Prop({ reflect: true }) disabled = false;
```

---

### 3. **Boolean State Flags** (Visible State)

Reflect boolean properties that represent visible component state:

```tsx
@Prop({ reflect: true, mutable: true }) checked = false;
@Prop({ reflect: true }) indeterminate = false;
@Prop({ reflect: true }) expanded = false;
```

**Why?**
- ✅ **DevTools Visibility**: Easy to inspect state in browser
- ✅ **Debugging**: State changes visible in Elements panel
- ✅ **CSS Styling**: Conditional styles based on state

**Real Example:**
```tsx
// toggle.tsx
@Prop({ mutable: true, reflect: true }) checked = false;
@Prop({ mutable: true, reflect: true }) indeterminate = false;

// Usage
<ix-toggle checked="true" indeterminate="false"></ix-toggle>
```

---

### 4. **Accessibility Attributes** (A11y)

Reflect properties that map to ARIA attributes:

```tsx
@Prop({ reflect: true }) ariaExpanded = false;
@Prop({ reflect: true }) ariaSelected = false;
```

**Why?**
- ✅ **Screen Readers**: Assistive technology reads DOM attributes
- ✅ **Standards**: ARIA spec requires DOM attributes, not just properties

---

## When to Use `reflect: false` (Default) ❌

### 1. **Text Content / Strings** (Performance)

Do NOT reflect text content or large strings:

```tsx
// ❌ DON'T reflect text
@Prop() label?: string;
@Prop() placeholder?: string;
@Prop() helperText?: string;
@Prop() errorText?: string;

// ✅ Keep as properties only
```

**Why?**
- ⚡ **Performance**: Avoid serializing large strings to DOM
- 🔒 **Encapsulation**: Text is internal rendering concern
- 📦 **Size**: Large attributes bloat DOM

**Real Example:**
```tsx
// toggle.tsx
@Prop() textOn = 'On';              // NO reflect
@Prop() textOff = 'Off';            // NO reflect
@Prop() textIndeterminate = 'Mixed'; // NO reflect

// checkbox.tsx
@Prop() label?: string;              // NO reflect
```

---

### 2. **Complex Data Types** (Can't Serialize)

Never reflect objects, arrays, or functions:

```tsx
// ❌ NEVER reflect complex types
@Prop() items: any[] = [];
@Prop() config: object = {};
@Prop() onClick?: () => void;

// ✅ Properties only (but consider slots for items!)
```

**Why?**
- ❌ **Can't Serialize**: Objects/arrays can't convert to attribute strings
- ⚡ **Performance**: Huge performance cost if attempted
- 🔒 **Security**: May expose internal data structures

**Better Alternative for Lists:**
For arrays of items, consider using **slots** instead of props:

```tsx
// ❌ Don't pass items as array prop
<ix-select items={[{id: 1, label: 'Option 1'}, ...]}></ix-select>

// ✅ Use slots (Web Components best practice)
<ix-select>
  <ix-select-item value="1">Option 1</ix-select-item>
  <ix-select-item value="2">Option 2</ix-select-item>
</ix-select>
```

**Benefits of Slots:**
- ✅ Better performance (no serialization)
- ✅ More flexible (custom markup)
- ✅ Composable (nest components)
- ✅ Declarative (HTML-first)

---

### 3. **Configuration Values** (Internal Logic)

Do NOT reflect configuration or variant properties:

```tsx
// ❌ DON'T reflect variants/config
@Prop() variant: 'primary' | 'secondary' = 'primary';
@Prop() size: 'small' | 'medium' | 'large' = 'medium';
@Prop() alignment: 'start' | 'center' | 'end' = 'center';

// ✅ Use CSS classes instead
```

**Why?**
- 🎨 **CSS Classes**: Variants typically use classes, not attributes
- ⚡ **Performance**: No need for DOM sync
- 📦 **Encapsulation**: Configuration is internal concern

**Real Example:**
```tsx
// button.tsx
@Prop() variant: ButtonVariant = 'primary';  // NO reflect
@Prop() type: 'button' | 'submit' = 'button'; // NO reflect
@Prop() alignment: 'center' | 'start' = 'center'; // NO reflect

// pane.tsx
@Prop() variant: 'floating' | 'inline' = 'inline';  // NO reflect
@Prop() size: '240px' | '320px' | ... = '240px';    // NO reflect
```

---

### 4. **Boolean Flags (Non-Visual)** (Internal State)

Do NOT reflect boolean flags that are purely for internal logic:

```tsx
// ❌ DON'T reflect internal flags
@Prop() hideText = false;
@Prop() hideOnCollapse = false;
@Prop() closeOnClickOutside = false;
@Prop() loading = false;

// ✅ Internal logic only
```

**Why?**
- 🔒 **Encapsulation**: Implementation details
- ⚡ **Performance**: No external observer needs this
- 🎨 **CSS**: Use classes, not attributes

**Real Example:**
```tsx
// toggle.tsx
@Prop() hideText = false;            // NO reflect

// pane.tsx
@Prop() hideOnCollapse: boolean = false;  // NO reflect
@Prop() closeOnClickOutside = false;      // NO reflect

// button.tsx
@Prop() loading: boolean = false;         // NO reflect
```

---

### 5. **Sensitive Data** (Security)

NEVER reflect passwords, tokens, or sensitive information:

```tsx
// ❌ NEVER reflect sensitive data
@Prop() password = '';
@Prop() apiToken = '';
@Prop() secretKey = '';

// ✅ Keep in memory only
```

**Why?**
- 🔒 **Security**: Visible in DOM inspector
- 🔍 **Audit Risk**: Attributes logged/captured
- 🚨 **Best Practice**: Sensitive data should never touch DOM

---

### 6. **Frequently Changing Values** (Performance)

Do NOT reflect values that change rapidly:

```tsx
// ❌ DON'T reflect high-frequency changes
@Prop() currentTime = 0;
@Prop() scrollPosition = 0;
@Prop() mouseX = 0;

// ✅ State only
```

**Why?**
- ⚡ **Performance**: Each change triggers DOM write
- 📊 **Overhead**: Unnecessary serialization cost

---

### 7. **Internal/Private Props** (API Design)

Do NOT reflect internal or private properties:

```tsx
/** @internal */
@Prop() ignoreLayoutSettings = false;  // NO reflect

/** @internal */
@Prop() isMobile = false;              // NO reflect
```

**Why?**
- 🔒 **API Privacy**: Not part of public API
- 📦 **Encapsulation**: Implementation detail

**Real Example:**
```tsx
// pane.tsx
/**
 * @internal
 * Prevents overwriting of variant when used inside layout
 */
@Prop() ignoreLayoutSettings: boolean = false;  // NO reflect

/** @internal */
@Prop({ mutable: true }) isMobile: boolean = false;  // NO reflect
```

---

## Decision Tree 🌲

```
Does external system need to observe this property?
│
├─ YES → Is it a form property (name, value, checked, disabled)?
│   │
│   ├─ YES → ✅ USE reflect: true (Web Standards)
│   │
│   └─ NO → Does CSS need to style based on this?
│       │
│       ├─ YES → ✅ USE reflect: true (CSS Selectors)
│       │
│       └─ NO → Is it a boolean state flag (checked, expanded)?
│           │
│           ├─ YES → ✅ USE reflect: true (Visibility)
│           │
│           └─ NO → Is it for accessibility (ARIA)?
│               │
│               ├─ YES → ✅ USE reflect: true (A11y)
│               │
│               └─ NO → ❌ USE reflect: false (Default)
│
└─ NO → ❌ USE reflect: false (Default)
    │
    └─ Is it text, object, array, or sensitive?
        │
        ├─ YES → ❌ NEVER reflect (Security/Performance)
        │
        └─ NO → Is it configuration or internal?
            │
            └─ YES → ❌ NEVER reflect (Encapsulation)
```

---

## Quick Reference Table 📊

| Property Type | `reflect` | Example | Reason |
|---------------|-----------|---------|--------|
| **Form attributes** | ✅ `true` | `name`, `value`, `checked`, `disabled`, `required` | Web standards, form validation, CSS |
| **Interactive state** | ✅ `true` | `checked`, `expanded`, `selected` | CSS styling, DevTools visibility |
| **ARIA attributes** | ✅ `true` | `ariaExpanded`, `ariaSelected` | Accessibility, screen readers |
| **Text content** | ❌ `false` | `label`, `placeholder`, `helperText` | Performance, encapsulation |
| **Configuration** | ❌ `false` | `variant`, `size`, `alignment` | CSS classes, internal logic |
| **Boolean flags (internal)** | ❌ `false` | `hideText`, `loading`, `closeOnClickOutside` | Encapsulation, no CSS dependency |
| **Complex types** | ❌ `false` | `items[]`, `config{}`, functions | Can't serialize |
| **Sensitive data** | ❌ `false` | `password`, `token` | Security |
| **High-frequency** | ❌ `false` | `currentTime`, `scrollPosition` | Performance |
| **Internal/private** | ❌ `false` | `@internal` props | API privacy |

---

## Code Examples

### ✅ Good: Form Component (reflect: true)
```tsx
@Component({
  tag: 'ix-checkbox',
  formAssociated: true,
})
export class Checkbox {
  // ✅ Reflect form-related properties
  @Prop({ reflect: true }) name?: string;
  @Prop({ reflect: true }) value: string = 'on';
  @Prop({ reflect: true }) checked: boolean = false;
  @Prop({ reflect: true }) disabled: boolean = false;
  @Prop({ reflect: true }) required = false;
  
  // ❌ Don't reflect text content
  @Prop() label?: string;
}
```

### ✅ Good: Button Component (mixed)
```tsx
@Component({
  tag: 'ix-button',
})
export class Button {
  // ✅ Reflect disabled for CSS styling
  @Prop({ reflect: true }) disabled = false;
  
  // ❌ Don't reflect variant (uses CSS classes)
  @Prop() variant: ButtonVariant = 'primary';
  
  // ❌ Don't reflect type (internal logic)
  @Prop() type: 'button' | 'submit' = 'button';
  
  // ❌ Don't reflect loading (internal state)
  @Prop() loading = false;
  
  // ❌ Don't reflect text/icon
  @Prop() icon?: string;
}
```

### ❌ Bad: Over-reflecting
```tsx
// ❌ DON'T DO THIS
@Component({ tag: 'ix-card' })
export class Card {
  @Prop({ reflect: true }) title?: string;        // ❌ Text content
  @Prop({ reflect: true }) items: any[] = [];     // ❌ Complex data
  @Prop({ reflect: true }) config: object = {};   // ❌ Object
  @Prop({ reflect: true }) variant = 'default';   // ❌ Configuration
  @Prop({ reflect: true }) loading = false;       // ❌ Internal flag
}

// ✅ CORRECT VERSION
@Component({ tag: 'ix-card' })
export class Card {
  @Prop() title?: string;        // ✅ Text content (or use slot)
  // ❌ Don't use items prop - use slots instead!
  @Prop() config: object = {};   // ✅ Object (internal only)
  @Prop() variant = 'default';   // ✅ Configuration
  @Prop() loading = false;       // ✅ Internal flag
  
  render() {
    return (
      <Host>
        <slot name="header"></slot>  {/* ✅ Use slots for content */}
        <slot></slot>
        <slot name="footer"></slot>
      </Host>
    );
  }
}
```

### ✅ Good: Using Slots for Items
```tsx
// ✅ RECOMMENDED: Slots over array props
@Component({ tag: 'ix-list' })
export class List {
  render() {
    return (
      <Host>
        <slot></slot>  {/* Accept ix-list-item children */}
      </Host>
    );
  }
}

// Usage
<ix-list>
  <ix-list-item>Item 1</ix-list-item>
  <ix-list-item>Item 2</ix-list-item>
  <ix-list-item>Item 3</ix-list-item>
</ix-list>

// ❌ AVOID: Array props for content
@Component({ tag: 'ix-list' })
export class List {
  @Prop() items: string[] = [];  // Avoid this pattern
}
```

---

## Testing Reflection

### Check if reflection is working:
```tsx
// Component
@Prop({ reflect: true }) disabled = false;

// Test
const button = document.querySelector('ix-button');
button.disabled = true;

// Check DOM
console.log(button.getAttribute('disabled')); 
// ✅ With reflect: true → "true"
// ❌ Without reflect: null
```

### Verify in DevTools:
```html
<!-- With reflect: true -->
<ix-button disabled="true">Click</ix-button>

<!-- Without reflect -->
<ix-button>Click</ix-button>
<!-- disabled property exists but not visible -->
```

---

## Common Mistakes to Avoid

### ❌ Mistake 1: Reflecting everything "just in case"
```tsx
// ❌ BAD
@Prop({ reflect: true }) variant = 'primary';
@Prop({ reflect: true }) label = '';
@Prop({ reflect: true }) config = {};
```

**Impact:** Performance degradation, bloated DOM

---

### ❌ Mistake 2: Not reflecting form properties
```tsx
// ❌ BAD
@Component({ formAssociated: true })
export class Input {
  @Prop() name?: string;      // Should reflect!
  @Prop() value = '';         // Should reflect!
  @Prop() required = false;   // Should reflect!
}
```

**Impact:** Form validation breaks, CSS selectors fail

---

### ❌ Mistake 3: Reflecting sensitive data
```tsx
// ❌ BAD - PASSWORD VISIBLE IN DOM!
@Prop({ reflect: true }) password = '';

// ✅ GOOD
@Prop() password = '';
```

**Impact:** Security vulnerability

---

### ❌ Mistake 4: Reflecting without CSS need
```tsx
// ❌ BAD - No CSS uses this
@Prop({ reflect: true }) loading = false;

// CSS uses classes instead
.loading { /* ... */ }

// ✅ GOOD
@Prop() loading = false;
```

**Impact:** Unnecessary performance cost

---

## Summary

### The Golden Rule
**"Reflect only what needs to be observed. Keep everything else private."**

### Default Approach
```tsx
// Start with NO reflection (default)
@Prop() myProp = 'value';

// Add reflection ONLY when needed
@Prop({ reflect: true }) disabled = false;  // CSS/Forms need it
```

### Key Principles
1. ✅ **Follow Web Standards**: Mirror native HTML for forms
2. ⚡ **Optimize Performance**: Don't reflect unless necessary
3. 🔒 **Ensure Security**: Never reflect sensitive data
4. 🎨 **Enable Styling**: Reflect when CSS needs it
5. ♿ **Support Accessibility**: Reflect ARIA attributes
6. 🔍 **Aid Debugging**: Reflect visible state flags
7. 📦 **Maintain Encapsulation**: Keep internals private

---

## Questions?

If you're unsure whether to reflect a property, ask:
1. Does CSS need to style based on this attribute?
2. Is this a form property (matching native `<input>`)?
3. Is this a boolean state that should be visible in DevTools?
4. Do accessibility tools need to read this attribute?

**If NO to all → Don't reflect (default)**
**If YES to any → Reflect (`reflect: true`)**

---

## Testing Components with Non-Reflected Properties

### For Consumers

Many properties (like `icon`, `variant`, `label`) are **not reflected** for performance and encapsulation reasons. When testing, access them via **properties**, not attributes.

#### ❌ This Won't Work:
```tsx
const button = screen.getByRole('button');
expect(button.getAttribute('icon')).toBe('check');  // ❌ Returns null
expect(button.getAttribute('variant')).toBe('primary');  // ❌ Returns null
```

#### ✅ Test Properties Instead:
```tsx
const button = screen.getByRole('button');
expect(button.icon).toBe('check');
expect(button.variant).toBe('primary');
```

#### Using Testing Library:
```tsx
import { screen } from '@testing-library/dom';

const button = screen.getByRole('button');
expect(button).toHaveProperty('icon', 'check');
expect(button).toHaveProperty('variant', 'primary');
```

#### Using Playwright:
```tsx
const button = page.locator('ix-button');
const icon = await button.evaluate((el: HTMLIxButtonElement) => el.icon);
expect(icon).toBe('check');
```

#### Multiple Options:
```tsx
const button = screen.getByRole('button');

expect(button.icon).toBe('check');
expect(button).toHaveProperty('icon', 'check');

const iconElement = button.shadowRoot?.querySelector('ix-icon');
expect(iconElement).toBeInTheDocument();
```

### Quick Reference: How to Test

| Property Type | Reflected? | Test Via Attribute | Test Via Property |
|--------------|------------|-------------------|-------------------|
| `disabled`, `checked`, `required` | ✅ Yes | ✅ `getAttribute('disabled')` | ✅ `element.disabled` |
| `icon`, `variant`, `size` | ❌ No | ❌ Returns `null` | ✅ `element.icon` |
| `label`, `placeholder` | ❌ No | ❌ Returns `null` | ✅ `element.label` |

### Why Not Reflect Everything?

While reflecting all properties would make attribute testing easier, it comes with significant costs:

**Performance Impact:**
```tsx
// If we reflected icon (BAD)
@Prop({ reflect: true }) icon = 'check';

// Every icon change = DOM write
button.icon = 'save';    // Triggers DOM update
button.icon = 'delete';  // Triggers DOM update
button.icon = 'edit';    // Triggers DOM update
// 3 DOM updates = expensive!

// Without reflection (GOOD)
button.icon = 'save';    // Property change only
button.icon = 'delete';  // Property change only
button.icon = 'edit';    // Property change only
// 0 DOM updates = fast!
```

**Security Considerations:**
```tsx
// Don't expose internal configuration to DOM
@Prop() apiEndpoint = '/api/users';  // Keep private
@Prop() debugMode = true;            // Keep private
```

---

## Contributing

When adding new components or properties, follow this philosophy. During code review, we'll verify:
- ✅ Form properties are reflected
- ✅ CSS-dependent states are reflected  
- ✅ Text content is NOT reflected
- ✅ Complex data is NOT reflected
- ✅ Sensitive data is NOT reflected

---

**Document Version**: 1.1  
**Last Updated**: 2025-01-11  
**Maintainers**: Siemens IX Team

