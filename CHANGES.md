# Changes: Todoist API Migration

## Problem

Todoist shut down their REST v2 API (`https://api.todoist.com/rest/v2/`). Any call to that endpoint now returns:

```
410: This endpoint is deprecated.
If you're using the API directly, please update your use case to rely
on the new API endpoints, available under /api/v1/ prefixes.
```

The `@doist/todoist-api-typescript` library (v2.1.2) used in this plugin was hardcoded to the old `/rest/v2/` base URL, which caused every sync to fail.

## What Changed

### `main.ts`

**Removed** the external `@doist/todoist-api-typescript` library entirely. It was a heavyweight dependency (bundling axios, runtypes, and more) when the plugin only ever used a single method: `getProjects()`.

**Added** a local `Project` interface with the three fields the plugin actually uses:
```typescript
interface Project {
    id: string;
    name: string;
    parentId: string | null;
}
```

**Replaced** the API call with a direct `requestUrl` call to the new endpoint. Obsidian's built-in `requestUrl` is the correct way for plugins to make HTTP requests — it respects Obsidian's proxy settings and certificate handling.

The new API (`/api/v1/projects`) returns paginated results in the form:
```json
{ "results": [...], "next_cursor": "..." }
```
The code loops through all pages to collect every project (most users will only have one page).

The raw JSON uses `parent_id` (snake_case), which is mapped to `parentId` (camelCase) to match the rest of the code.

### `package.json`

Removed the `@doist/todoist-api-typescript` dependency. This also reduces the compiled bundle size significantly.

## What Did NOT Change

- All logic for creating, moving, and archiving Obsidian notes based on project structure is unchanged.
- The settings UI is unchanged.
- The sync interval behaviour is unchanged.
- No new features were added.

## Build

After these changes you need to reinstall dependencies and rebuild:

```bash
npm install
npm run build
```

The compiled `main.js` in the project root will be updated. See `TESTING.md` for how to load it safely in Obsidian.
