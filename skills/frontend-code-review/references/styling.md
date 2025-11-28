# Styling Patterns

Teetsh-specific styling conventions using twin.macro and Tailwind.

## twin.macro Usage

Use `tw` macro for Tailwind classes, NOT className strings:

```tsx
// Correct
<div tw="px-4 pt-4 pb-2 text-sm text-neutral-quiet">

// Wrong
<div className="px-4 pt-4 pb-2 text-sm text-neutral-quiet">
```

## Conditional Styles

Use array syntax for conditional styles:

```tsx
// Correct
<div
  css={[
    tw`border border-neutral-quiet rounded-md`,
    !hasError && tw`hover:border-brand-intense`,
    hasError && tw`border-danger-intense`,
  ]}
>

// Wrong - template literals
<div className={`border ${hasError ? 'border-danger' : 'border-neutral'}`}>
```

## Theme Constants

Colors are centralized in `src/styles/themeConstants.ts`. Never hardcode colors:

```tsx
// Correct - use theme colors
tw`text-brand-intense bg-neutral-subtle border-danger-intense`

// Wrong - hardcoded colors
tw`text-[#2E43BD]`
<div style={{ color: '#2E43BD' }}>
```

Available color categories:
- `brand-*`: Brand colors (intense, moderate, subtle)
- `neutral-*`: Gray scale (intense, moderate, quiet, subtle)
- `danger-*`: Error/danger states
- `success-*`: Success states
- `warning-*`: Warning states

## Styled Components

When creating styled components with dynamic props:

```tsx
import tw from 'twin.macro';
import styled from '@emotion/styled';

const Wrapper = styled.div(
  ({ hasLabel, shouldFill }: { hasLabel?: boolean; shouldFill?: boolean }) => [
    tw`relative`,
    hasLabel && tw`mt-1`,
    shouldFill && tw`w-full`,
  ],
);
```

## FormGroup Pattern

ALL form inputs MUST be wrapped with FormGroup:

```tsx
// Correct
<FormGroup
  id="email"
  label={t('common:form.email')}
  labelInfo={t('common:form.optional')}
  error={errors.email}
  helperText={t('common:form.emailHelper')}
  fill
>
  <Input
    id="email"
    value={formData.email}
    onChange={(e) => setFormData({ ...formData, email: e.target.value })}
    hasError={!!errors.email}
  />
</FormGroup>

// Wrong - input without FormGroup
<Input
  value={email}
  onChange={(e) => setEmail(e.target.value)}
/>
```

FormGroup provides:
- Consistent label styling
- Error message display
- Helper text
- Accessibility (label-input linking)

## Input Component Props

```tsx
<Input
  id="email"              // Required for accessibility
  value={value}
  onChange={onChange}
  hasError={!!error}      // Triggers error styling
  small                   // Size variant
  large                   // Size variant
  extraLarge              // Size variant
  icon={<SearchIcon />}   // Left icon
  rightElement={<Button />} // Right element
  selectAllOnFocus        // Select text on focus
  autoComplete="off"      // Autocomplete control
/>
```

## Responsive Design

**Prefer CSS-based responsive** with Tailwind classes (mobile-first, `lg:` for desktop):

```tsx
// Preferred - CSS handles responsive
<div tw="flex flex-col lg:flex-row">
<div tw="hidden lg:block">        // Desktop only
<div tw="block lg:hidden">        // Mobile only
<div tw="text-sm lg:text-base">   // Smaller on mobile
```

**Only use `useIsBelowBreakpoint` when layouts are completely different**:

```tsx
// Only when mobile and desktop need entirely different components
const isMobile = useIsBelowBreakpoint('lg');

if (isMobile) {
  return <MobileDashboard />;  // Completely different layout
}
return <DesktopDashboard />;
```

## Print Styles

For printable content:

```tsx
<div tw="print:hidden">           // Hide when printing
<div tw="hidden print:block">     // Show only when printing
```

## Common Size Classes

Standard spacing and sizing:
- Buttons: `button-ms`, `button-l` (custom utilities)
- Text: `text-sm`, `text-base`, `text-lg`
- Spacing: `gap-2`, `gap-4`, `p-4`, `px-6`

## Icons

Use HeroIcons from the design system:

```tsx
import { XMarkIcon, PlusIcon } from '@heroicons/react/24/outline';
import { CheckIcon } from '@heroicons/react/24/solid';

<Button>
  <PlusIcon tw="w-5 h-5 mr-2" />
  {t('common:buttons.add')}
</Button>
```

## Common Mistakes

### Hardcoded Colors
```tsx
// Wrong
tw`text-[#2E43BD]`
<div style={{ color: 'red' }}>

// Correct
tw`text-brand-intense`
tw`text-danger-intense`
```

### className Instead of tw
```tsx
// Wrong
<div className="flex items-center">

// Correct
<div tw="flex items-center">
```

### Missing FormGroup
```tsx
// Wrong
<label>Email</label>
<Input value={email} onChange={setEmail} />
<span>{error}</span>

// Correct
<FormGroup id="email" label="Email" error={error}>
  <Input id="email" value={email} onChange={setEmail} hasError={!!error} />
</FormGroup>
```

### Inconsistent Responsive Patterns
```tsx
// Wrong - mixing approaches
<div className="hidden md:block">
<div tw="lg:flex">

// Correct - consistent breakpoints
<div tw="hidden lg:block">
<div tw="lg:flex">
```

### Missing Error Styling on Inputs
```tsx
// Wrong - error state not reflected
<FormGroup error={error}>
  <Input value={value} />
</FormGroup>

// Correct
<FormGroup error={error}>
  <Input value={value} hasError={!!error} />
</FormGroup>
```
