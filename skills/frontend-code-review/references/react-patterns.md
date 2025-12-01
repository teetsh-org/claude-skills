# React Patterns

Teetsh-specific React patterns and conventions.

## Container vs Component Separation

**Containers** (in `/containers/`) handle:

- Business logic and state management
- Data fetching via React Query hooks
- Routing and navigation
- Responsive branching (mobile vs desktop)

**Components** (in `/components/`) are:

- Presentational and reusable
- Receive data via props
- Handle local UI state only

```tsx
// Container - handles logic
export default function StudentsContainer() {
  const { data, isLoading } = useStudents();
  const isMobile = useIsBelowBreakpoint("lg");

  if (isMobile) return <StudentsMobile students={data} />;
  return <StudentsDesktop students={data} />;
}

// Component - presentational
export default function StudentCard({ student }: Props) {
  return (
    <Card>
      <h3>{student.name}</h3>
    </Card>
  );
}
```

## Compound Components Pattern

Complex UI groups use compound components with context:

```tsx
// Declaration
export default function Disclosure(props: Props) {
  const [open, setOpen] = useState(false);
  return (
    <DisclosureContext.Provider value={{ open, setOpen }}>
      {props.children}
    </DisclosureContext.Provider>
  );
}
Disclosure.Button = DisclosureButton;
Disclosure.Panel = DisclosurePanel;

// Usage
<Disclosure>
  <Disclosure.Button>Toggle</Disclosure.Button>
  <Disclosure.Panel>Content</Disclosure.Panel>
</Disclosure>;
```

## Context Hooks with Validation

Context hooks must validate the context exists:

```tsx
// Correct
export function useUser() {
  const context = useContext(UserContext);
  if (context === undefined) {
    throw new Error("useUser must be used within a UserProvider");
  }
  return context;
}

// Wrong - no validation
export function useUser() {
  return useContext(UserContext); // Could return undefined
}
```

Common context hooks:

- `useUser()` - Current user data
- `useSchoolSchoolyear()` - Current school and schoolyear context

## Custom Hooks Naming

All custom hooks must start with `use`:

```tsx
// Correct
export function useStudentGrades(studentId: string) { ... }
export function useScrollConfig() { ... }

// Wrong
export function getStudentGrades(studentId: string) { ... }
export function scrollConfig() { ... }
```

## Responsive Design

**Prefer CSS-based responsive design** using Tailwind classes:

```tsx
// Preferred - CSS handles responsive
<div tw="flex flex-col lg:flex-row">
<div tw="hidden lg:block">        // Desktop only
<div tw="block lg:hidden">        // Mobile only
```

**Only use `useIsBelowBreakpoint` when layouts are completely different**:

```tsx
// Only when mobile and desktop are fundamentally different components
export default function Dashboard() {
  const isMobile = useIsBelowBreakpoint("lg");

  if (isMobile) {
    return <MobileDashboard />; // Entirely different component
  }
  return <DesktopDashboard />;
}
```

For viewport height considerations:

```tsx
import { useMediaQuery } from "react-responsive";
const isSmallHeight = useMediaQuery({ maxHeight: 700 });
```

## Export Conventions

- **Components**: Default export
- **Hooks**: Named export
- **Utilities**: Named export
- **Types**: Named export

```tsx
// Component file
export default function Button(props: Props) { ... }

// Hook file
export function useStudents() { ... }

// Utility file
export function formatDate(date: Date) { ... }
export function parseTime(time: string) { ... }

// Types file
export interface Student { ... }
export type StudentStatus = 'active' | 'inactive';
```

## Error Boundaries

Use `ErrorPage` component for error boundaries:

```tsx
<ErrorBoundary fallback={<ErrorPage />}>
  <FeatureContainer />
</ErrorBoundary>
```

## Provider Composition

Providers should handle loading, error, and success states:

```tsx
export function SchoolProvider({ children }: Props) {
  const { data, isLoading, error } = useSchoolData();

  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorPage />;

  return (
    <SchoolContext.Provider value={data}>{children}</SchoolContext.Provider>
  );
}
```

## Common Mistakes

### Missing Loading States

```tsx
// Wrong - no loading handling
const { data } = useStudents();
return <StudentList students={data} />;

// Correct
const { data, isLoading } = useStudents();
if (isLoading) return <LoadingSpinner />;
return <StudentList students={data} />;
```

### Side Effects in JSX

```tsx
// Wrong - side effect in render
return (
  <div>
    {console.log(data)}
    {data.name}
  </div>
);

// Correct - side effects in useEffect
useEffect(() => {
  console.log(data);
}, [data]);
return <div>{data.name}</div>;
```
