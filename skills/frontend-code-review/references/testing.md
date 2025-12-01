# Testing Patterns

Teetsh testing conventions using Vitest (browser + jsdom), Storybook, and Playwright.

## Test Strategy

| Test Type     | Environment    | File Suffix         | Use For                      |
| ------------- | -------------- | ------------------- | ---------------------------- |
| Browser tests | Vitest browser | `.browser.test.tsx` | Components, UI interactions  |
| Unit tests    | Vitest jsdom   | `.test.tsx`         | Utilities, hooks, pure logic |
| E2E tests     | Playwright     | `*.spec.ts`         | Critical user flows          |

## Test File Organization

Tests are co-located with source files:

```
Component/
├── index.tsx
├── index.browser.test.tsx     # Browser tests for component
├── index.stories.tsx          # Storybook stories
├── useComponentState.ts       # Component-specific hook
└── useComponentState.test.tsx # jsdom unit test for hook

utils/
├── formatDate.ts
└── formatDate.test.tsx     # jsdom unit test
```

## Browser Tests (Components)

Use Vitest browser mode for testing components with real DOM:

```tsx
// Component.browser.test.tsx
import { describe, it, expect } from "vitest";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

describe("DeleteButton", () => {
  it("should call onDelete when confirmed", async () => {
    const onDelete = vi.fn();
    render(<DeleteButton onDelete={onDelete} />);

    await userEvent.click(screen.getByRole("button", { name: /delete/i }));
    await userEvent.click(screen.getByRole("button", { name: /confirm/i }));

    expect(onDelete).toHaveBeenCalledOnce();
  });
});
```

## Unit Tests (jsdom)

Use jsdom for testing utilities, hooks, and pure logic:

```tsx
// formatDate.test.tsx
import { describe, it, expect } from "vitest";

describe("formatDate", () => {
  it("should format date in French locale", () => {
    const date = new Date(2024, 0, 15);
    const result = formatDate(date, "fr-FR");
    expect(result).toBe("15 janvier 2024");
  });

  it("should handle null date", () => {
    const result = formatDate(null, "fr-FR");
    expect(result).toBe("");
  });
});
```

### Descriptive Test Names

Use `it('should...')` format describing expected behavior:

```tsx
// Correct
it("should return empty array when no students match filter");
it("should throw error when user is not authenticated");
it("should update cache after successful mutation");

// Wrong - vague names
it("works");
it("handles error");
it("test formatDate");
```

### Testing Edge Cases

Always test edge cases:

```tsx
describe('calculateTotal', () => {
  it('should calculate total for multiple items', () => { ... });
  it('should return 0 for empty array', () => { ... });
  it('should handle negative values', () => { ... });
  it('should handle decimal precision', () => { ... });
});
```

## Storybook Stories

All design system components should have Storybook stories:

```tsx
import type { Meta, StoryObj } from "@storybook/react";
import Button from "./index";

const meta = {
  title: "Components/Button",
  component: Button,
  tags: ["autodocs"],
  argTypes: {
    variant: {
      control: "select",
      options: ["primary", "secondary", "danger"],
    },
    size: {
      control: "select",
      options: ["small", "medium", "large"],
    },
    disabled: { control: "boolean" },
  },
  args: {
    children: "Click me",
    variant: "primary",
    disabled: false,
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = {
  args: { variant: "primary" },
};

export const Secondary: Story = {
  args: { variant: "secondary" },
};

export const Disabled: Story = {
  args: { disabled: true },
};

export const AllVariants: Story = {
  render: () => (
    <div tw="flex gap-4">
      <Button variant="primary">Primary</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="danger">Danger</Button>
    </div>
  ),
};
```

### Story Requirements

- Use `satisfies Meta<typeof Component>` for type safety
- Document all props via `argTypes`
- Include stories for all major variants and states
- Add `tags: ['autodocs']` for auto-generated docs

## Playwright E2E Tests

For critical user flows:

```tsx
import { test, expect } from "@playwright/test";

test.describe("Authentication", () => {
  test("should login with valid credentials", async ({ page }) => {
    await page.goto("/login");

    await page.fill('[data-testid="email"]', "test@example.com");
    await page.fill('[data-testid="password"]', "password123");
    await page.click('[data-testid="submit"]');

    await expect(page).toHaveURL("/dashboard");
    await expect(page.locator('[data-testid="user-menu"]')).toBeVisible();
  });

  test("should show error for invalid credentials", async ({ page }) => {
    await page.goto("/login");

    await page.fill('[data-testid="email"]', "wrong@example.com");
    await page.fill('[data-testid="password"]', "wrongpassword");
    await page.click('[data-testid="submit"]');

    await expect(page.locator('[data-testid="error-message"]')).toBeVisible();
  });
});
```

### E2E Test Guidelines

- Test critical user flows (auth, payments, core features)
- Use `data-testid` attributes for stable selectors
- Test both success and error paths
- Keep tests independent (no shared state)

## Common Mistakes

### Wrong Test Type

```tsx
// Wrong - using jsdom test for a component
// DeleteButton.test.tsx (jsdom) ❌

// Correct - use browser test for components
// DeleteButton.browser.test.tsx ✓

// Note: jsdom is fine for pure logic like formatDate.test.tsx
```

### Vague Test Names

```tsx
// Wrong
it("works");
it("test 1");

// Correct
it("should return formatted date for valid input");
```

### Missing Edge Cases

```tsx
// Wrong - only happy path
describe('divideNumbers', () => {
  it('should divide two numbers', () => { ... });
});

// Correct - includes edge cases
describe('divideNumbers', () => {
  it('should divide two numbers', () => { ... });
  it('should throw error when dividing by zero', () => { ... });
  it('should handle negative numbers', () => { ... });
});
```

### Testing Implementation Details

```tsx
// Wrong - testing internal state
expect(component.state.isOpen).toBe(true);

// Correct - testing behavior
expect(screen.getByRole("dialog")).toBeVisible();
```

### Shared State Between Tests

```tsx
// Wrong - tests affect each other
let counter = 0;
it("test 1", () => {
  counter++;
});
it("test 2", () => {
  expect(counter).toBe(0);
}); // Fails!

// Correct - isolated tests
beforeEach(() => {
  counter = 0;
});
```

### Missing Stories for Design System Components

```tsx
// Every component in /components should have a .stories.tsx file
// documenting its variants, states, and usage
```
