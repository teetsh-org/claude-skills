# Common Mistakes

Teetsh-specific antipatterns to catch during code review.

## API & Data Fetching

### Using Axios Directly
```tsx
// Wrong
import axios from 'axios';
axios.get('/api/students');

// Correct - use request utility (handles auth, errors, interceptors)
import request from '@teetsh/app/src/utils/request';
request('/api/students');
```

### Query Keys Missing School Context
```tsx
// Wrong - cache won't be isolated per school
queryKey: [STUDENTS_KEY, classId]

// Correct
queryKey: [schoolyear.id, school.id, STUDENTS_KEY, classId]
```

### Data Transformation in Components
```tsx
// Wrong - transform in component
const { data } = useStudents();
const formatted = data?.map(s => ({ ...s, fullName: `${s.first} ${s.last}` }));

// Correct - transform in serializer or select
export function useStudents({ select }) {
  return useQuery({
    select: select || formatStudents,
  });
}
```

## Styling

### className Instead of tw
```tsx
// Wrong
<div className="flex items-center gap-2">

// Correct
<div tw="flex items-center gap-2">
```

### Hardcoded Colors
```tsx
// Wrong
tw`text-[#2E43BD]`
style={{ color: '#921E1E' }}

// Correct
tw`text-brand-intense`
tw`text-danger-intense`
```

### Input Without FormGroup
```tsx
// Wrong
<Input value={email} onChange={setEmail} />

// Correct
<FormGroup id="email" label={t('common:form.email')} error={error}>
  <Input id="email" value={email} onChange={setEmail} hasError={!!error} />
</FormGroup>
```

### Missing hasError on Input
```tsx
// Wrong - error shows but input not styled
<FormGroup error={error}>
  <Input value={value} />
</FormGroup>

// Correct
<FormGroup error={error}>
  <Input value={value} hasError={!!error} />
</FormGroup>
```

## i18n

### Hardcoded User-Facing Strings
```tsx
// Wrong
<Button>Save</Button>
<p>No results found</p>

// Correct
<Button>{t('common:buttons.save')}</Button>
<p>{t('common:empty.noResults')}</p>
```

### Missing Namespace in useTranslation
```tsx
// Wrong
const { t } = useTranslation();
t('buttons.save')

// Correct
const { t } = useTranslation(['common']);
t('common:buttons.save')
```

### Using date-fns Directly
```tsx
// Wrong
import { format } from 'date-fns';

// Correct
import { format } from '@teetsh/app/src/utils/time/format';
```

## TypeScript

### Unnecessary `any` Types
```tsx
// Wrong
const handleClick = (e: any) => { ... }
const data: any = response.data;

// Correct
const handleClick = (e: MouseEvent<HTMLButtonElement>) => { ... }
const data: StudentResponse = response.data;
```

### Missing Props Interface
```tsx
// Wrong
export default function Card({ title, children }) { ... }

// Correct
interface Props {
  title: string;
  children: React.ReactNode;
}
export default function Card({ title, children }: Props) { ... }
```

## React Patterns

### Missing Loading State
```tsx
// Wrong
const { data } = useStudents();
return <StudentList students={data} />;

// Correct
const { data, isLoading } = useStudents();
if (isLoading) return <LoadingSpinner />;
return <StudentList students={data} />;
```

### Missing Error State
```tsx
// Wrong
const { data, isLoading } = useStudents();

// Correct
const { data, isLoading, error } = useStudents();
if (error) return <ErrorPage />;
```

### Context Hook Without Validation
```tsx
// Wrong
export function useUser() {
  return useContext(UserContext);
}

// Correct
export function useUser() {
  const context = useContext(UserContext);
  if (context === undefined) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return context;
}
```

### Prop Drilling Through Many Levels
```tsx
// Wrong - drilling through 3+ components
<Parent data={data}>
  <Child data={data}>
    <GrandChild data={data} />

// Correct - use context
<DataProvider value={data}>
  <Parent>
    <Child>
      <GrandChild /> {/* uses useData() */}
```

## Accessibility

### Missing aria-label on Icon Buttons
```tsx
// Wrong
<button onClick={onDelete}>
  <TrashIcon />
</button>

// Correct
<button onClick={onDelete} aria-label={t('common:buttons.delete')}>
  <TrashIcon aria-hidden="true" />
</button>
```

### Missing htmlFor on Labels
```tsx
// Wrong
<label>Email</label>
<Input id="email" />

// Correct
<label htmlFor="email">Email</label>
<Input id="email" />

// Or use FormGroup which handles this
```

### Non-Semantic Elements for Interactive Content
```tsx
// Wrong
<div onClick={handleClick}>Click me</div>

// Correct
<button onClick={handleClick}>Click me</button>
```

## Testing

### Vague Test Names
```tsx
// Wrong
it('works')
it('test formatDate')

// Correct
it('should format date in French locale')
it('should return empty string for null date')
```

### Missing Edge Case Tests
```tsx
// Wrong - only happy path
it('should return students', () => { ... });

// Correct - include edge cases
it('should return students', () => { ... });
it('should return empty array when no students', () => { ... });
it('should throw error when unauthorized', () => { ... });
```

### Missing Storybook Stories
```tsx
// Every component in /components/ should have .stories.tsx
// Check: Does this new component have stories?
```

## Performance

### Missing useCallback for Event Handlers Passed to Children
```tsx
// Wrong - creates new function every render
<ChildComponent onClick={() => handleClick(id)} />

// Correct
const handleChildClick = useCallback(() => handleClick(id), [id]);
<ChildComponent onClick={handleChildClick} />
```

### Missing useMemo for Expensive Computations
```tsx
// Wrong - recalculates every render
const sortedStudents = students.sort((a, b) => a.name.localeCompare(b.name));

// Correct
const sortedStudents = useMemo(
  () => students.sort((a, b) => a.name.localeCompare(b.name)),
  [students]
);
```

## Quick Checklist

Before approving any PR, verify:

- [ ] No hardcoded strings (use i18n)
- [ ] No hardcoded colors (use theme constants)
- [ ] No className (use tw macro)
- [ ] No axios (use request utility)
- [ ] Query keys include school context
- [ ] Form inputs wrapped with FormGroup
- [ ] Loading and error states handled
- [ ] TypeScript types defined (no unnecessary any)
- [ ] Tests have descriptive names
- [ ] Accessibility attributes present
