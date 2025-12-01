# React Query Patterns

Teetsh-specific React Query conventions.

## Query Key Structure

Query keys MUST include school context for proper cache isolation:

```tsx
// Correct - includes schoolyear and school IDs
const { schoolyear, school } = useSchoolSchoolyear();

useQuery({
  queryKey: [schoolyear.id, school.id, STUDENTS_KEY, classId],
  queryFn: () =>
    getStudents({ schoolyearId: schoolyear.id, schoolId: school.id, classId }),
});

// Wrong - missing school context
useQuery({
  queryKey: [STUDENTS_KEY, classId],
  queryFn: () => getStudents(classId),
});
```

Key constants are defined at the top of query files:

```tsx
const STUDENTS_KEY = "students";
const APPELS_KEY = "appels";
```

## Query Hook Structure

Standard pattern for query hooks:

```tsx
interface Props {
  date: Date;
  select?: (data: DayAppels) => T;
  notifyOnChangeProps?: string[];
}

export function useAppels<T = Appel | undefined>(props: Props) {
  const { date, select } = props;
  const user = useUser();
  const { schoolyear, school } = useSchoolSchoolyear();

  return useQuery<DayAppels, AxiosError, T>({
    queryKey: [schoolyear.id, school.id, APPELS_KEY, format(date, "fr-FR")],
    queryFn: () =>
      getAppelsBetweenDates({
        schoolyearId: schoolyear.id,
        schoolId: school.id,
        startDate: date,
        endDate: date,
        tz: user.location,
      }),
    select: select || ((data) => data[props.type]),
    staleTime: 1000 * 60 * 60, // 1 hour
    refetchOnWindowFocus: false,
  });
}
```

## Serializer Pattern

Data transformation happens in serializer functions, NOT in components:

```tsx
// In queries/appels/serializer.ts
export function deserializeAppel(data: any, tz: string): Appel {
  return {
    id: data.id,
    date: parseISO(data.date),
    students: data.students.map(deserializeStudent),
    // ... transformation logic
  };
}

// In queryFn
queryFn: async () => {
  const response = await request('/appels');
  return deserializeAppel(response.data, user.location);
},
```

## staleTime Guidelines

- **Frequently changing data** (attendance, live updates): `0` or `1000 * 60` (1 min)
- **Session data** (current school, user): `1000 * 60 * 60` (1 hour)
- **Static data** (settings, config): `1000 * 60 * 60 * 24` (24 hours)
- **Reference data** (countries, timezones): `Infinity`

## Error Handling

Always type errors as `AxiosError`:

```tsx
const { error } = useQuery<Data, AxiosError>({ ... });

if (error) {
  const status = error.response?.status;
  const is4xx = status ? status >= 400 && status < 500 : false;

  if (is4xx) {
    return <NotFoundPage />;
  }
  return <ErrorPage />;
}
```

## Mutations

Standard mutation pattern with cache invalidation:

```tsx
export function useUpdateStudent() {
  const queryClient = useQueryClient();
  const { schoolyear, school } = useSchoolSchoolyear();

  return useMutation({
    mutationFn: (data: UpdateStudentInput) =>
      request(`/students/${data.id}`, {
        method: "PATCH",
        data: serializeStudent(data),
      }),
    onSuccess: () => {
      queryClient.invalidateQueries({
        queryKey: [schoolyear.id, school.id, STUDENTS_KEY],
      });
    },
  });
}
```

## Optimistic Updates

For better UX on mutations:

```tsx
useMutation({
  mutationFn: updateStudent,
  onMutate: async (newStudent) => {
    await queryClient.cancelQueries({ queryKey: studentKeys });

    const previousStudents = queryClient.getQueryData(studentKeys);

    queryClient.setQueryData(studentKeys, (old) =>
      old.map((s) => (s.id === newStudent.id ? newStudent : s))
    );

    return { previousStudents };
  },
  onError: (err, newStudent, context) => {
    queryClient.setQueryData(studentKeys, context.previousStudents);
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: studentKeys });
  },
});
```

## Common Mistakes

### Missing Context in Query Keys

```tsx
// Wrong
queryKey: [STUDENTS_KEY];

// Correct
queryKey: [schoolyear.id, school.id, STUDENTS_KEY];
```

### Transformation in Component

```tsx
// Wrong - transforming in component
const { data } = useStudents();
const formattedStudents = data?.map((s) => ({
  ...s,
  fullName: `${s.firstName} ${s.lastName}`,
}));

// Correct - transform in serializer or select
const { data } = useStudents({
  select: (students) => students.map(formatStudent),
});
```

### Not Using Request Utility

```tsx
// Wrong - using axios directly
import axios from "axios";
axios.get("/students");

// Correct - use request utility
import request from "@teetsh/app/src/utils/request";
request("/students");
```

### Forgetting Error State

```tsx
// Wrong
const { data, isLoading } = useStudents();
if (isLoading) return <Spinner />;
return <List data={data} />;

// Correct
const { data, isLoading, error } = useStudents();
if (isLoading) return <Spinner />;
if (error) return <ErrorPage />;
return <List data={data} />;
```
