# Routing & State Patterns for Vite + React

## Table of Contents
1. [React Router v6+ Patterns](#1-react-router-v6-patterns)
2. [TanStack Query Patterns](#2-tanstack-query-patterns)
3. [Client State (Zustand)](#3-client-state-zustand)
4. [Form State (React Hook Form)](#4-form-state-react-hook-form)

---

## 1. React Router v6+ Patterns

### Route Definition (Centralized)

Define all routes in one file. Lazy load every page component.

```ts
// src/router.tsx
import { createBrowserRouter } from 'react-router-dom'
const Dashboard = React.lazy(() => import('./pages/Dashboard'))
const Profile = React.lazy(() => import('./pages/Profile'))

export const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    errorElement: <ErrorPage />,
    children: [
      { index: true, element: <Suspense fallback={<Spinner />}><Dashboard /></Suspense> },
      { path: 'profile', element: <Suspense fallback={<Spinner />}><Profile /></Suspense> },
      { path: '*', element: <NotFound /> },
    ],
  },
])
```

### Protected Routes

Auth check via loader or wrapper component — never inside page render.

```ts
// Protected route wrapper
function RequireAuth({ children }: { children: React.ReactNode }) {
  const { user, isLoading } = useAuth()
  if (isLoading) return <Spinner />
  if (!user) return <Navigate to="/login" replace />
  return <>{children}</>
}
```

### URL State for Filters/Pagination

Use `useSearchParams` for shareable state — not `useState`.

```ts
const [searchParams, setSearchParams] = useSearchParams()
const page = Number(searchParams.get('page') ?? '1')
const filter = searchParams.get('status') ?? 'all'
```

---

## 2. TanStack Query Patterns

### Query Key Convention

Include all variables the query depends on.

```ts
// Keys as factory
const todoKeys = {
  all: ['todos'] as const,
  list: (filters: Filters) => ['todos', 'list', filters] as const,
  detail: (id: string) => ['todos', 'detail', id] as const,
}

// Usage
useQuery({ queryKey: todoKeys.list({ status, page }), queryFn: () => fetchTodos({ status, page }) })
```

### Conditional Queries

Use `enabled` to prevent premature fetches.

```ts
const { data: profile } = useQuery({
  queryKey: ['profile', userId],
  queryFn: () => fetchProfile(userId),
  enabled: !!userId,  // don't run until userId is defined
})
```

### Mutation with Cache Invalidation

```ts
const mutation = useMutation({
  mutationFn: (data: CreateTodo) => createTodo(data),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: todoKeys.all })
  },
  onError: (error) => {
    toast.error('Failed to create todo')
  },
})
```

### Stale Time Guidance

| Data Type | staleTime |
|-----------|-----------|
| User profile | 5 min |
| Reference / config data | 10–30 min |
| Real-time dashboard | 0 (always refetch) |
| Static lists | 5–15 min |

---

## 3. Client State (Zustand)

Use Zustand only for UI state that multiple components share and that doesn't come from the server.

```ts
// src/stores/uiStore.ts
interface UIState {
  sidebarOpen: boolean
  toggleSidebar: () => void
}

export const useUIStore = create<UIState>((set) => ({
  sidebarOpen: false,
  toggleSidebar: () => set((s) => ({ sidebarOpen: !s.sidebarOpen })),
}))
```

**Rule**: If the data comes from an API, it belongs in TanStack Query — not Zustand.

---

## 4. Form State (React Hook Form)

Don't manage form fields with `useState`. Use React Hook Form.

```ts
const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<FormData>({
  resolver: zodResolver(schema),
})

const onSubmit = async (data: FormData) => {
  await mutation.mutateAsync(data)
}
```

- Integrate with Zod for schema validation (`@hookform/resolvers/zod`)
- Show field-level errors from `errors` object
- Disable submit button while `isSubmitting`
