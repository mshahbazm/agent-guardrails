# AirbaseHQ Frontend - AI Coding Agent Reference

> **Cross-package rule:** Before making changes in any other package of this
> monorepo, always read that package's `CLAUDE.md` first to understand its
> conventions and rules.

React + Vite frontend using TanStack Router and TanStack Query. Backend:
`packages/api`.

## Quick Reference

- **API types**: `import type { tX } from '@api/types'`
- **Error handling**: `getErrorMessage(response.messageCode)`
- **Data fetching**: TanStack Query (`useQuery`, `useMutation`)
- **Routing**: TanStack Router (`useNavigate`, `Route.useParams()`, `<Link>`)
- **Components**: `src/components/*` (`ui/*` for shadcn)
- **Hooks**: `src/hooks/*`
- **API layer**: `src/api/*`

## Content & i18n

- **Never hardcode user-facing strings.** Use the content files in `src/i18n/`.
- Components: `const t = useContent('namespace')`
- Non-React code: `const t = getContent('namespace')`
- `useContent` returns memoized content and is safe to use in dependency arrays.
- Resolve API error codes with `getErrorMessage(response.messageCode)`.

## API Integration

### Data Fetching

```typescript
// Queries - GET requests
const { data, isLoading } = useQuery({
  queryKey: ['jobs', companyId],
  queryFn: () => api.jobs.list(companyId),
})

// Mutations - POST/PUT/DELETE
const mutation = useMutation({
  mutationFn: api.jobs.create,
  onSuccess: () => queryClient.invalidateQueries({ queryKey: ['jobs'] }),
})
```

### API Functions

```typescript
// src/api/jobs.ts
import type { tJobsListResponse } from '@api/types'

export const jobs = {
  list: async (companyId: string): Promise<tJobsListResponse> => {
    const res = await fetch(`${API_URL}/companies/${companyId}/jobs`, {
      credentials: 'include',
    })
    return res.json()
  },
}
```

- Keep requests in the API layer and include credentials for authenticated
  requests.
- Import response types from the backend package instead of recreating them in
  the frontend.
- Fetch integration and workflow capabilities from their owning service. Never
  hardcode a second list in the UI.
- Navigation is server-driven. Do not hardcode menu items that the API owns.

## File Organization

- **Config**: `src/config/*` (`cookies.ts`, `redirects.ts`, `api.ts`)
- **Types**: `src/types/*`, except component props
- **Actions**: `src/actions/*`
- **API**: `src/api/*`
- **Components**: `src/components/*` (`ui/*` for shadcn, `feature/*` for pages)
- **Hooks**: `src/hooks/*` (`common/*` for shared hooks)
- **Routes**: `src/routes/*`
- **Store**: `src/store/*` (discuss before adding Zustand)
- **i18n**: `src/i18n/*`
- **Prompts**: `src/prompts/*` (**do not modify**)

## Component Rules

- One component, one responsibility.
- Keep components under 150 lines or split them.
- Declare props as `interface iProps {}` directly above the component.
- Extract reusable or complex logic into custom hooks.
- Use `React.memo` for components with expensive renders.

## Routing

- **Always use `<Link>` for internal navigation. Never use `<a href>`.** A raw
  anchor triggers a full browser reload, reboots the app, reruns session
  loading, and can leave the screen blank for several seconds. `<Link>` performs
  a client-side route change.
- Use typed params and search values instead of building an internal URL with a
  template string:

  ```tsx
  <Link
    to="/jobs/$code"
    params={{ code: `job-${job.code}` }}
  >
    View job
  </Link>
  ```

- Pass query parameters through `search`:

  ```tsx
  <Link to="/contacts" search={{ clientCompanyId: id }}>
    View contacts
  </Link>
  ```

- If a typed `<Link>` requires a search value that should be optional, correct
  the route's `validateSearch` return type. Do not work around it with an
  untyped URL.
- For programmatic navigation, use
  `navigate({ to: '/path' })` from `useNavigate()`.
- Never use `window.location.href` for an internal route.
- Use a raw `<a href>` only for external URLs, `mailto:`, `tel:`, or an
  intentional new-tab link.
- Use `Route.useParams()` for route params and `Route.useSearch()` for query
  params.
- Use `beforeLoad` for route guards and redirects.

## Type Safety

- Put application types in `src/types/*`.
- Use `tName` for type aliases and `iName` for interfaces.
- Declare component props as `interface iProps {}` above the component, not in
  `src/types/`.
- Import API types from `@api/types`.
- Import integration types from the integration-service package.
- Use `import type` for type-only imports. `verbatimModuleSyntax` is enabled.
- Prefix intentionally unused variables with `_`.

## UI States

- Empty data: use `<EmptyState />`.
- API errors: use `<ErrorState />`.
- Loading: use the query's `isLoading` state or `<Skeleton />`.

## Conventions

- Use the `@/` alias for application imports.
- Component filenames use `kebab-case.tsx`.
- Utility filenames use `camelCase.ts`.
- Handle API failures with `try/catch` or by checking `'error' in result`.
- Read the active company from the authentication context. Pass `companyId`
  explicitly to providers that need it.
- Add `cursor-pointer` to buttons, links, tabs, and other interactive elements.

## Zustand + Immer

- Group store actions under an `actions` key. Keep its reference stable so it
  does not cause re-renders.
- Export atomic selector hooks such as `useJobs()`. Do not expose raw
  `useStore(state => state.jobs)` selectors throughout components.
- Never call store methods inside selectors. Selectors must return stable
  references.
- Keep save and delete operations in plain async functions, not hooks or the
  store.
- Initialize page data through `actions.initialize()` when the page mounts.

## Simplicity

Prefer the simplest flow. Before adding code, ask whether it is needed, whether
it adds value, and whether the codebase already has a standard way to do it. Cut
anything that does not earn its place.

## DON'TS

- Never modify `src/prompts/*`. AI prompts are user-managed.
- Never allow API errors to crash the application.
- Never hardcode user-facing strings.
- Never duplicate backend types in the frontend.
- Never bypass TanStack Router for internal navigation.

## Environment Variables

When adding an environment variable, stop and remind the user to update the
deployment configuration for this package. Ask them to confirm that it has been
updated before proceeding. The package Dockerfile must receive the new variable
as well.
