---
name: fe-structure
description: Structure JS/TS frontend code — domain/feature folders with slices, a layered data layer, small query/mutation hooks with key factories, small global stores. Use when building, adding or refactoring features in any frontend app.
---

# fe-structure

A general way to organize frontend JavaScript/TypeScript code. The principles hold for any framework. Reference code uses React + TanStack Query + axios + Zustand and is TypeScript; in a JS project drop the types, and on another stack swap in the equivalents (§8).

If the project already has an established pattern, match it. Otherwise introduce these pieces incrementally — don't restructure unrelated code.

---

## 1. Principles

1. **Domain/feature folders.** Code lives at the lowest level that covers all its consumers — slice, feature, domain, then app — and moves up only when a new consumer appears.
2. **Public API per feature.** Other features import only through a feature's `index.js`.
3. **Layered data access.** UI → feature query/mutation hooks → generic data hooks → request core → HTTP client. The UI never touches the HTTP client or query keys.
4. **Small hooks, composed.** One hook per query or mutation; a page composes them. No hook owns everything.
5. **Right state in the right place.** Server data in the query cache, global client state in small stores, everything else local.
6. **Isolate the framework and the backend.** Router, env vars and navigation each sit behind one adapter; response shapes and errors are normalized once in the request core.

---

## 2. Folder structure

Use the project's file extension; `.js` is shown.

```
/<domain>/<feature>/
├── index.js                 public API: exports the page and anything other features may use
├── FeaturePage.js           the screen: layout + wiring, little logic
├── components/
│   ├── Component.js         simple component files live directly here
│   └── <feature-slice>/     optional, for a distinct slice with its own structure
│       ├── components/      only when the slice has cohesive child components
│       ├── context/
│       ├── helpers/
│       ├── hooks/
│       ├── styled/
│       ├── assets/
│       └── *.js
├── context/                 only for state shared by multiple sibling slices or the page shell
├── helpers/                 only for helpers shared by multiple sibling slices or the page shell
├── hooks/                   only for hooks shared by multiple sibling slices or the page shell
│   └── queries.js           the feature's query key factory + query/mutation hooks
├── styled/
└── assets/
```

Example:

```
/hr/employees/
├── index.js
├── EmployeesPage.js
├── types.ts                         feature types (TS projects), when shared by several files
├── hooks/queries.js                 employeeKeys + useEmployeeList, useSaveEmployee, useDeleteEmployee
├── components/
│   ├── EmployeeStatusBadge.js
│   └── employee-form/               slice with its own structure
│       ├── EmployeeForm.js
│       ├── components/AddressFields.js
│       ├── helpers/buildPayload.js
│       └── hooks/useEmployeeFormRules.js
└── styled/
```

App-wide code sits outside the domains: shared components, generic data hooks and adapters, `lib/` (HTTP client, query client), stores, design tokens, shared types, pure `utils/` (the request core). Match the names the project already uses.

**Placement** — the lowest level that covers every consumer:

| Used by | Goes in |
|---|---|
| One simple component | its own file in `components/` |
| One slice only | inside that slice |
| Several sibling slices or the page shell | feature-level `hooks/`, `helpers/`, `context/` |
| Several features in the domain | a shared folder at the domain level |
| Several domains | app-wide shared folders |

- Create a slice folder only when the piece has its own internal structure; create sub-folders only when they have content.
- `context/` holds UI state shared within the feature, never server data.
- Types: next to their only user; feature-level `types` file when shared inside the feature; app-wide only for cross-feature types (API envelope, shared models).

**Imports and barrels**
- `index.js` exists **only** at the feature boundary, and exports only what other features need.
- Inside a feature and in shared folders, import files directly (`@/shared/hooks/useList`), not through `index` barrels. Barrel chains slow builds, tests and type-checking.
- Enforce the boundary with lint, adjusted to the project's alias and depth:

```js
// eslint config
'no-restricted-imports': ['error', { patterns: [{
  group: ['@/*/*/*/*'],   // @/<domain>/<feature>/<anything> — deeper than the feature's index
  message: 'Import other features through their index.js.',
}] }],
```

**Naming:** components and pages PascalCase (`<Feature>Page`); slice folders kebab-case; query hooks `use<Resource>List` / `use<Resource>Detail`; mutation hooks `use<Verb><Resource>` (`useSaveEmployee`); key factory `<resource>Keys`; stores `use<Domain>Store`; helpers camelCase.

---

## 3. Data flow and state

```
<Feature>Page                     renders; owns UI state (modal open, editingId); shows toasts
  → hooks/queries.js              key factory + useXList / useXDetail / useSaveX / useDeleteX
    → useList / useQueryFetch / useMutationSubmit   generic data hooks
      → fetchApi                  request core: params cleanup, FormData, success check, error shape
        → http client             base URL, auth header, 401 handling, timeout
```

| State | Where |
|---|---|
| Server data | query cache, via feature query hooks |
| Auth, theme and other global client state | stores (persisted where needed) |
| Form values | form library or component |
| UI toggles | local component state |

- Never copy server data into a store, context or local state; derive from the cache.
- No toasts in the request core, no URL logic in components, no HTTP calls in components.

---

## 4. API contract

Define the backend envelope once in shared types. Example — adapt to the real backend:

```ts
export interface ApiResponse<T = unknown> {
  success: boolean;
  message: string;
  data: T;
  pagination?: { page: number; limit: number; total: number; next: number | null; prev: number | null };
  errors?: Record<string, string[]> | string[];
}
export interface AppError { message: string; status?: number; errors?: ApiResponse['errors'] }
```

If the backend changes, only `fetchApi` and these types change.

---

## 5. Generic data layer (reference: React + TanStack Query + axios)

Build once in shared folders; every feature reuses it.

### Request core — `utils/fetchApi.ts` (no framework imports)

```ts
type Method = 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE';
export interface FetchOptions { url: string; method?: Method; params?: Record<string, unknown>; payload?: unknown; signal?: AbortSignal }

const isEmpty = (v: unknown) => v === undefined || v === null || v === '';
const hasFile = (p: unknown) => p && typeof p === 'object' &&
  Object.values(p).some((v) => (Array.isArray(v) ? v.some((i) => i instanceof Blob) : v instanceof Blob));
const toFormData = (p: Record<string, unknown>) => {
  const fd = new FormData();
  for (const [k, v] of Object.entries(p)) {
    if (Array.isArray(v)) v.forEach((i) => fd.append(`${k}[]`, i as Blob));
    else if (!isEmpty(v)) fd.append(k, v as Blob);
  }
  return fd;
};

export default async function fetchApi<T = unknown>({ url, method = 'GET', params = {}, payload, signal }: FetchOptions): Promise<ApiResponse<T>> {
  try {
    const res = await http.request<ApiResponse<T>>({
      url, method, signal,
      params: Object.fromEntries(Object.entries(params).filter(([, v]) => !isEmpty(v))),
      data: payload === undefined ? undefined : hasFile(payload) ? toFormData(payload as Record<string, unknown>) : payload,
    });
    if (res.data?.success === false) throw { message: res.data.message, status: res.status, errors: res.data.errors } satisfies AppError;
    return res.data;
  } catch (e) {
    if (!(e instanceof AxiosError)) throw e;
    throw { message: e.response?.data?.message ?? e.message, status: e.response?.status, errors: e.response?.data?.errors } satisfies AppError;
  }
}
```

### HTTP client — `lib/http`

One instance: base URL from env, timeout, JSON headers. Request interceptor adds the token via `useAuthStore.getState()`. Response interceptor on 401 clears auth and calls the navigation adapter.

### Reads — `useQueryFetch`

```ts
export default function useQueryFetch<T>({ queryKey, url, params, ...options }:
  { queryKey: readonly unknown[]; url: string; params?: Record<string, unknown> } & Partial<UseQueryOptions<ApiResponse<T>, AppError>>) {
  return useQuery<ApiResponse<T>, AppError>({
    queryKey,                                          // built by the feature's key factory
    queryFn: ({ signal }) => fetchApi<T>({ url, params, signal }),
    ...options,
  });
}
```

### Writes — `useMutationSubmit`

```ts
export default function useMutationSubmit<V, TData = unknown>({ url, method = 'POST', payload, ...options }: {
  url: string | ((vars: V) => string);                 // resolved per call → one hook serves every row
  method?: 'POST' | 'PUT' | 'PATCH' | 'DELETE' | ((vars: V) => 'POST' | 'PUT' | 'PATCH' | 'DELETE');
  payload?: (vars: V) => unknown;                      // default: vars as body; DELETE: no body
} & Partial<UseMutationOptions<ApiResponse<TData>, AppError, V>>) {
  return useMutation<ApiResponse<TData>, AppError, V>({
    mutationFn: (vars) => {
      const m = typeof method === 'function' ? method(vars) : method;
      return fetchApi<TData>({
        url: typeof url === 'function' ? url(vars) : url,
        method: m,
        payload: payload ? payload(vars) : m === 'DELETE' ? undefined : vars,
      });
    },
    ...options,
  });
}
```

### Lists — `useList` (paginated, URL-synced)

```ts
useList({
  queryKey: (params) => employeeKeys.list(params),   // from the feature's key factory
  url: '/employees',
  defaultParams: { page: 1, limit: 20 },
  filters: { search: ['search', 'q'], status: ['status'] },   // alias → param names
})
// → items, pagination, isLoading, isFetching, error
// → goToPage(page, limit?)     limit change resets to page 1
// → setFilter(alias, value)    filter change resets to page 1
// → setParams({...}) / resetParams() / params / isFiltered
```

Reads params from the URL merged over `defaultParams`, fetches with `useQueryFetch({ queryKey: queryKey(params), url, params })`, writes cleaned params back with replace. URL access goes through one adapter — the only file that knows the router:

```ts
// useUrlParams
export function useUrlParams(): [URLSearchParams, (next: Record<string, string>) => void] {
  // React Router: const [sp, setSp] = useSearchParams(); return [sp, (n) => setSp(n, { replace: true })];
  // Next.js:      const sp = useSearchParams(); const r = useRouter(); const p = usePathname();
  //               return [new URLSearchParams(sp.toString()), (n) => r.replace(`${p}?${new URLSearchParams(n)}`)];
  // No router:    window.location.search + history.replaceState + local state to re-render.
}
```

---

## 6. Feature queries

Each feature has one `hooks/queries.js`: a key factory plus one small hook per query or mutation. Components use the hooks; keys never leave the file.

**Key factory** — generic → specific, so invalidation can target any level:

```ts
export const employeeKeys = {
  all: ['employees'] as const,
  lists: () => [...employeeKeys.all, 'list'] as const,
  list: (params: Record<string, unknown>) => [...employeeKeys.lists(), params] as const,
  details: () => [...employeeKeys.all, 'detail'] as const,
  detail: (id: string) => [...employeeKeys.details(), id] as const,
};
```

**Hooks** — data consequences (invalidation) live here; UI consequences (toast, closing a modal) stay with the caller:

```ts
export const useEmployeeList = () =>
  useList({ queryKey: employeeKeys.list, url: '/employees', filters: { search: ['search'], status: ['status'] } });

export const useEmployeeDetail = (id?: string) =>
  useQueryFetch<Employee>({ queryKey: employeeKeys.detail(id ?? ''), url: `/employees/${id}`, enabled: Boolean(id) });

export function useSaveEmployee() {
  const qc = useQueryClient();
  return useMutationSubmit<EmployeeInput & { id?: string }>({
    url: ({ id }) => (id ? `/employees/${id}` : '/employees'),
    method: ({ id }) => (id ? 'PUT' : 'POST'),
    onSuccess: (_res, { id }) => {
      qc.invalidateQueries({ queryKey: employeeKeys.lists() });
      if (id) qc.invalidateQueries({ queryKey: employeeKeys.detail(id) });
    },
  });
}

export function useDeleteEmployee() {
  const qc = useQueryClient();
  return useMutationSubmit<{ id: string }>({
    url: ({ id }) => `/employees/${id}`,
    method: 'DELETE',
    onSuccess: () => qc.invalidateQueries({ queryKey: employeeKeys.lists() }),
  });
}
```

**Page** — composes the hooks and owns UI feedback through per-call callbacks:

```tsx
// /hr/employees/EmployeesPage
export default function EmployeesPage() {
  const [editing, setEditing] = useState<Employee | null>(null);
  const [open, setOpen] = useState(false);
  const toast = useToast();

  const { items, pagination, isLoading, setFilter, goToPage } = useEmployeeList();
  const save = useSaveEmployee();
  const remove = useDeleteEmployee();

  const handleSave = (values: EmployeeInput) =>
    save.mutate({ ...values, id: editing?.id }, {
      onSuccess: () => { toast.success('Employee saved'); setOpen(false); },
      onError: (e) => toast.error(e.message),
    });

  const handleDelete = (id: string) =>
    remove.mutate({ id }, { onSuccess: () => toast.success('Employee deleted'), onError: (e) => toast.error(e.message) });

  return (
    <>
      <EmployeeToolbar onSearch={(v) => setFilter('search', v)} onCreate={() => { setEditing(null); setOpen(true); }} />
      <EmployeeTable rows={items} loading={isLoading} pagination={pagination} onPageChange={goToPage}
                     onEdit={(row) => { setEditing(row); setOpen(true); }} onDelete={handleDelete} />
      <EmployeeForm open={open} initialValues={editing} loading={save.isPending} errors={save.error?.errors}
                    onSubmit={handleSave} onCancel={() => setOpen(false)} />
    </>
  );
}
```

Rules:
- One hook per query or mutation. If a page wires many of them, a thin `use<Feature>Page` hook may compose them — it only combines, it holds no fetch logic.
- Invalidate by key instead of calling `refetch()` on a specific list; use optimistic updates only when the UX needs them.
- Child components get props and callbacks; they fetch only when they're self-contained widgets, and then through the feature's query hooks.

---

## 7. Stores, styling, translation

**Stores**
- One small store per domain (`useAuthStore`, `useThemeStore`, …); actions are verbs (`setX`, `toggle`, `clearX`).
- Persist only what's needed; storage keys prefixed with the app slug.
- Client state only — never API responses.

**Styling**
- Colors, spacing, radii and shadows come from design tokens; no literal colors in components.
- If the app has dark mode, new UI must work in both themes.
- Prefer the project's UI kit; customize through its theme config before overriding styles.

**Translation**
- Follow however the project already handles translation (library, helper, key format, file locations). Don't introduce a new i18n setup.
- If the project translates UI text, new strings go through that mechanism; if it doesn't, don't add it.
- Examples here use plain strings for brevity.

**UX states**
- Every query handles loading, empty and error; every mutation disables its trigger while pending, shows `AppError.message`, and maps `errors` to fields.

---

## 8. Other stacks and frameworks

| Layer | React reference | Equivalents |
|---|---|---|
| HTTP client | axios | `fetch` wrapper, ky, ofetch |
| Query cache | TanStack Query | SWR, RTK Query, TanStack Query for Vue/Svelte/Solid |
| Stores | Zustand | Redux Toolkit, Jotai, Pinia, Svelte stores |
| Feature query hooks | hooks in `queries.js` | Vue composables, Svelte modules, Angular services |
| URL adapter | `useUrlParams` | `useRoute`/`useRouter` (Vue), `$page.url` (SvelteKit) |

Framework touch points, each in one place: URL params (adapter), navigation for logout/401 (one helper used by the HTTP interceptor), env vars (`lib/http` and config only), route files (thin — render the page from the feature's `index.js`), server/client boundaries (mark hook-using files as client). Don't name the domain root something the framework auto-routes (e.g. `src/pages` in Next.js).

---

## Checklist before finishing

- Code sits at the lowest level that covers its consumers; slices and sub-folders exist only where they have content.
- Other features import only through `index.js`; no barrels elsewhere; the lint rule passes.
- Components reach the API only through the feature's query/mutation hooks.
- Keys come from the feature's key factory; mutations invalidate by key.
- Mutation URLs and methods resolve per call; DELETE sends no body unless the API needs one.
- No literal colors; text follows the project's translation approach.
- Loading, empty, error and pending states handled.
- Type-check, lint and tests pass.
