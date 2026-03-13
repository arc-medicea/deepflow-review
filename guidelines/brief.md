# DeepFlow Frontend — Technical Brief

> **Purpose:** Specification for building the DeepFlow web application frontend.
> **Target:** Claude Code / Codex implementation agent.
> **Layout reference:** [wireframes.html](wireframes.html) — Option B (chat always on).
> **Design system:** [guidelines/](index.html) — full component specs, colour, typography, spacing.

---

## 1. Product Overview

DeepFlow is an AI-powered workflow orchestration platform. Users create hierarchical DAG-based workflows, assign tasks to humans and AI agents, and interact with an always-on AI chat panel. This is a **reskin of an existing product** — the DAG engine, chat backend, Slack integration, and @mentions are already in production. This brief covers the **frontend shell** only.

### Core Views
1. **Graph View** — DAG canvas with draggable nodes, edges, zones, detail panel
2. **List View** — Sortable task table with expandable parent/child rows
3. **Workspace** — Leaf-node working environment. Three-pane: Chat + Canvas + Detail. Canvas is a markdown editor by default, but can embed Google Docs, code views, spreadsheets, or agent output logs. This is where work is done or an agent's work is inspected. Entered by double-clicking/opening a node with no children
4. **Dashboard** — Landing page with project cards, summary stats, attention items
5. **Gallery** — Project browser with templates and project cards
6. **Settings** — App configuration with section-based navigation

### Fixed Layout (Option B)
| Panel | Width | Behaviour |
|-------|-------|-----------|
| Sidebar | 56px | Always collapsed (icons only). Expands to 240px on hover/click in non-Option-B mode |
| Chat Panel | 320px | Always visible. Context updates per view/task |
| Top Bar | 44px h | Breadcrumbs + view switcher + search + filters |
| Hub Bar | 36px / ~220px | Collapsible. Graph + Active Task views only |
| Detail Panel | 360px | Graph + Active Task views. Overlay in List. Absent in Workspace/Gallery/Settings |
| Content Area | Remaining | 464px (with detail) / 824px (without detail) of 1200px |

---

## 2. Tech Stack

### Core
| Layer | Choice | Rationale |
|-------|--------|-----------|
| Framework | **React 19 + Vite 6** | Thick-client SPA. No SSR needed (behind auth). Faster HMR, no hydration headaches, no RSC complexity for a workspace app |
| Routing | **React Router v7** | Client-side routing. Lazy-loaded route chunks. Loader/action pattern for data |
| Language | **TypeScript (strict)** | Non-negotiable |
| Styling | **Tailwind CSS 4** + CSS custom properties | Design tokens from guidelines map directly to CSS vars |
| Component library | **Radix UI (headless)** | Accessible primitives. No visual opinions to fight |
| Graph engine | **React Flow (xyflow)** | Mature DAG library. Custom nodes, edges, minimap, zoom, keyboard nav |
| Panel layout | **react-resizable-panels** | Resizable panel groups with drag handles. Keyboard accessible. Persists sizes |
| State management | **Zustand** | Lightweight, no boilerplate. Slices per domain |
| Data fetching | **TanStack Query v5** | Cache, dedup, background refresh. Initial tree load + WS patch updates |
| Forms | **React Hook Form + Zod** | Validation shares schemas with backend |
| i18n | **react-intl** (FormatJS) | ICU MessageFormat. No SSR dependency. Lazy locale loading |
| Icons | **Lucide React** | Consistent, tree-shakeable. Supplement with custom DeepFlow icons |

> **Why not Next.js?** DeepFlow is a thick-client workspace app: persistent WebSocket, full DAG tree in memory, complex interactive canvas, resizable panels, zero SEO needs. Everything is behind Keycloak auth — the page is meaningless without user data. Next.js optimises for content delivery and SSR; we'd fight the framework with `"use client"` on nearly every component. React + Vite is simpler, faster, and purpose-built for this kind of app.

### Auth
| Layer | Choice | Rationale |
|-------|--------|-----------|
| Provider | **Keycloak** | SSO, RBAC, org-scoped tenancy |
| Client library | **keycloak-js** + **react-oidc-context** | Lightweight OIDC wrapper. No server-side dependency. Handles token refresh, silent renew, session management |
| Token flow | Authorization Code + PKCE (browser-native) | Access token in memory (not localStorage). Refresh via silent iframe or refresh token. Token attached to API calls via interceptor |
| RBAC | Keycloak realm roles → mapped to app permissions | 5 roles: Viewer, Editor, Manager, Admin, Owner (per guidelines §27 Permissions) |
| Multi-tenancy | Keycloak organisations / groups → tenant context | Org switcher in sidebar. All API calls scoped to active tenant |

### Data Loading — Tree-First Architecture
| Layer | Choice | Rationale |
|-------|--------|-----------|
| Initial load | **Full DAG tree** fetched on project open | Entire task hierarchy (all levels) loaded into Zustand store. Typically 50–500 nodes. React Flow renders visible viewport; off-screen nodes are virtualised |
| Store shape | **Normalised tree in Zustand** | `Record<nodeId, TaskNode>` with `parentId`, `childIds[]`, `dependencyIds[]`, `level`. Flat map, tree derived via selectors. Enables O(1) lookups and patches |
| Rendering | **Derived views from single source** | Graph View, List View, Hub Bar, Detail Panel all read from the same Zustand tree store. No data duplication between views |
| TanStack Query role | **Initial fetch + cache only** | Query fetches the tree on project open. After that, Zustand owns the live state. Query handles stale/refetch on window focus as a consistency check |

### Real-time — WebSocket Patches
| Layer | Choice | Rationale |
|-------|--------|-----------|
| Transport | **WebSocket** (native, via backend gateway) | Persistent connection from login. Server pushes patches, chat messages, presence |
| Client | Custom `useWebSocket` hook | Auto-reconnect, exponential backoff, heartbeat ping/pong |
| Pattern | **WS event → direct Zustand patch** (not re-fetch) | `task.updated` → update the node in the store directly. No round-trip. UI re-renders via Zustand selectors. Only fall back to full re-fetch if patch sequence gaps detected |
| Events | `tree.patch` (add/update/remove/move nodes), `chat.message`, `presence.update`, `workflow.status` | `tree.patch` carries a sequence number. Client tracks last-seen seq; if gap detected, requests full tree sync |
| Conflict resolution | **Server-authoritative, last-write-wins** | Optimistic local updates with server confirmation. If server rejects (conflict), revert local state and show toast |
| Fallback | SSE for environments blocking WS | Same event schema, different transport |

### Persistence (Client-side)
| Layer | Choice | Rationale |
|-------|--------|-----------|
| Engine | **IndexedDB** via `idb-keyval` | Lightweight wrapper. No complex queries needed |
| Persisted data | View preferences, filter states, sidebar state, theme, layout choice (A/B), recent projects, draft chat messages | Survives page refresh and browser restart |
| Zustand middleware | `persist` middleware with IndexedDB storage adapter | Automatic hydration. Selective persistence per slice |
| Sync | Local-first, server-authoritative | Preferences sync to user profile API on change. IndexedDB is fast cache. Server is source of truth |
| TTL | Cached query data: 5 min stale, 30 min GC | TanStack Query `gcTime` + `staleTime` |

---

## 3. Architecture

### Project Structure
```
src/
├── routes/                       # React Router v7 routes
│   ├── _auth.tsx                 # Auth layout (login, callback)
│   ├── _app.tsx                  # Authenticated shell layout
│   ├── _app.dashboard.tsx        # Workspace (dashboard)
│   ├── _app.projects.tsx         # Gallery view
│   ├── _app.projects.$projectId.tsx      # Graph view (default)
│   ├── _app.projects.$projectId.list.tsx # List view
│   ├── _app.projects.$projectId.task.$taskId.tsx  # Workspace (leaf-node editor)
│   ├── _app.settings.tsx         # Settings
│   └── _app.settings.$section.tsx # Settings sub-section
├── components/
│   ├── shell/                    # Sidebar, ChatPanel, TopBar, HubBar
│   ├── graph/                    # React Flow custom nodes, edges, zones
│   ├── list/                     # Table, expandable rows
│   ├── workspace/                # Canvas editor, embeds (GDoc, code, agent log)
│   ├── dashboard/                # Project cards, stats, attention banner
│   ├── gallery/                  # Template cards, project cards
│   ├── detail/                   # Detail panel, tabs, forms
│   ├── chat/                     # Message list, input, action chips
│   └── ui/                       # Radix primitives styled w/ design tokens
├── stores/                       # Zustand stores
│   ├── tree.ts                   # THE source of truth: normalised task tree (Record<id, TaskNode>)
│   ├── layout.ts                 # Sidebar, panel sizes, panel order, Option A/B
│   ├── graph.ts                  # Selected node, zoom, viewport, drill level
│   ├── chat.ts                   # Messages (optimistic), draft input
│   ├── filters.ts                # Active filters, sort, search
│   └── preferences.ts            # Theme, locale, view defaults, panel sizes (persisted)
├── hooks/                        # Custom hooks
│   ├── useWebSocket.ts
│   ├── useAuth.ts
│   ├── useKeyboardNav.ts
│   └── useMediaQuery.ts
├── lib/
│   ├── api/                      # API client, types, endpoints
│   ├── auth/                     # Keycloak config, token helpers
│   ├── i18n/                     # react-intl config, message loaders
│   └── ws/                       # WebSocket client, event types
├── messages/                     # i18n translation files
│   ├── en.json
│   └── ...
├── styles/
│   ├── tokens.css                # CSS custom properties from design guidelines
│   └── globals.css
└── types/                        # Shared TypeScript types
    ├── task.ts
    ├── project.ts
    ├── workflow.ts
    └── user.ts
```

### Zustand Store Design

```typescript
// stores/tree.ts — THE source of truth for all task data
interface TaskNode {
  id: string
  name: string
  status: 'pending' | 'ready' | 'in_progress' | 'needs_approval' | 'complete' | 'failed'
  level: number                   // L1, L2, L3...
  parentId: string | null
  childIds: string[]
  dependencyIds: string[]         // upstream dependencies
  downstreamIds: string[]         // what depends on this
  assignee: { id: string; name: string; type: 'human' | 'agent'; avatar?: string }
  dueDate: string | null
  priority: 'critical' | 'high' | 'medium' | 'low'
  description: string
  properties: Record<string, PropertyValue>  // custom property pills
  updatedAt: string
}

interface TreeStore {
  nodes: Record<string, TaskNode>  // normalised flat map
  rootIds: string[]                // top-level (L1) node IDs
  lastSeq: number                  // last WS patch sequence number
  loading: boolean
  
  // Bulk operations
  loadTree: (projectId: string) => Promise<void>   // initial fetch
  clearTree: () => void
  
  // Patch operations (called by WS handler)
  patchNode: (id: string, patch: Partial<TaskNode>) => void
  addNode: (node: TaskNode) => void
  removeNode: (id: string) => void
  moveNode: (id: string, newParentId: string) => void
  
  // Derived selectors (memoised)
  getChildren: (parentId: string) => TaskNode[]
  getAncestors: (nodeId: string) => TaskNode[]
  getSubtree: (nodeId: string) => TaskNode[]      // node + all descendants
  getVisibleNodes: (level: number, parentId?: string) => TaskNode[]
}

// stores/layout.ts
interface LayoutStore {
  sidebarExpanded: boolean        // 56px vs 240px
  chatPanelVisible: boolean       // always true in Option B
  detailPanelOpen: boolean
  detailPanelTaskId: string | null
  hubBarExpanded: boolean
  layoutMode: 'overlay' | 'always-on'
  panelSizes: { chat: number; detail: number }  // persisted pixel widths
  
  toggleSidebar: () => void
  openDetail: (taskId: string) => void
  closeDetail: () => void
  toggleHubBar: () => void
  setLayoutMode: (mode: 'overlay' | 'always-on') => void
  setPanelSize: (panel: 'chat' | 'detail', size: number) => void
}

// stores/graph.ts
interface GraphStore {
  selectedNodeId: string | null
  viewport: { x: number; y: number; zoom: number }
  drillPath: string[]             // stack of node IDs for drill-down breadcrumb
  
  selectNode: (id: string | null) => void
  setViewport: (viewport: Viewport) => void
  drillDown: (nodeId: string) => void   // push to drillPath
  drillUp: () => void                   // pop from drillPath
  drillTo: (nodeId: string) => void     // jump to specific level (breadcrumb click)
}

// stores/filters.ts — persisted to IndexedDB
interface FiltersStore {
  activeFilters: Record<string, FilterValue>
  sortField: string
  sortDirection: 'asc' | 'desc'
  searchQuery: string
  viewPreference: Record<string, 'graph' | 'list' | 'kanban'>
  
  setFilter: (key: string, value: FilterValue) => void
  clearFilters: () => void
  setSort: (field: string, direction: 'asc' | 'desc') => void
  setSearch: (query: string) => void
  setViewPreference: (projectId: string, view: string) => void
}
```

---

## 4. Navigation

### The Problem
"Back" can mean five different things in DeepFlow:
1. Browser back button → previous URL in `window.history`
2. Drill up in graph → L3 → L2 → L1
3. Close Workspace → return to parent graph with node selected
4. Breadcrumb click → jump to a specific level
5. Project switch → entirely different data context

If these aren't unified, users will click browser-back and get something unexpected. The rule: **every meaningful navigation state change is a URL change.** Browser back always does what the user expects.

### URL Structure
```
/                                           → Dashboard
/projects                                   → Gallery
/projects/:projectId                        → Graph View (L1 root)
/projects/:projectId?view=list              → List View (same project)
/projects/:projectId?view=kanban            → Kanban View (same project)
/projects/:projectId/node/:nodeId           → Drilled into a node (L2/L3 graph)
/projects/:projectId/task/:taskId           → Workspace (leaf-node editor)
/settings                                   → Settings
/settings/:section                          → Settings sub-section
```

### Navigation History — What Goes in the URL vs What Doesn't

| State | In URL? | Why |
|-------|---------|-----|
| Current project | ✅ `/projects/:projectId` | Shareable, bookmarkable |
| View type (graph/list/kanban) | ✅ `?view=list` | Shareable. But also persisted per-project in IndexedDB as default |
| Drill level (which node you're inside) | ✅ `/node/:nodeId` | Browser back = drill up. Shareable deep links |
| Workspace (open task) | ✅ `/task/:taskId` | Shareable. Browser back = return to graph |
| Selected node (clicked, not drilled) | ❌ Zustand only | Transient. Selecting a node shouldn't pollute history |
| Panel sizes | ❌ IndexedDB | Preference, not navigation |
| Filters / sort | ❌ Zustand + IndexedDB | Persisted locally, not in URL (too noisy). Exception: shared filter links could use query params in future |
| Chat scroll position | ❌ Transient | Resets on navigation |
| Graph viewport (zoom, pan) | ❌ Zustand only | Transient. Restored from memory when returning to a level |

### What "Back" Does in Each Context

| You're in | Browser Back goes to | Breadcrumb shows |
|-----------|---------------------|------------------|
| Dashboard | Browser's previous page (external) | Dashboard |
| Gallery | Dashboard | Projects |
| Graph L1 (project root) | Gallery (or Dashboard if entered directly) | All Projects › [Project] |
| Graph L2 (drilled into node) | Graph L1 | All Projects › [Project] › [Node] |
| Graph L3 (drilled deeper) | Graph L2 | All Projects › [Project] › [L2 Node] › [L3 Node] |
| Workspace (leaf task) | Parent graph level (with node selected) | All Projects › [Project] › … › [Task] |
| List/Kanban view | Same as if you were in Graph at that level | Same breadcrumbs, view switcher changes |
| Settings | Whatever was open before Settings | Settings › [Section] |

### Drill-Down as Navigation

Each drill pushes a new history entry:
```
User opens project       → /projects/abc                  (history: 1 entry)
Drills into "Capital"    → /projects/abc/node/capital-123  (history: 2 entries)
Drills into "Risk Assess"→ /projects/abc/node/risk-456    (history: 3 entries)
Opens leaf task          → /projects/abc/task/task-789     (history: 4 entries)

Browser back (×1)        → /projects/abc/node/risk-456    (back to L3 graph)
Browser back (×2)        → /projects/abc/node/capital-123  (back to L2 graph)
Browser back (×3)        → /projects/abc                  (back to L1 root)
```

Breadcrumb clicks are different — they **replace** history (not push), so clicking "ICAAP 2026" from L3 takes you to L1 without adding intermediate entries. This prevents the "back trap" where users have to click back through every breadcrumb level.

### View Switching (Graph ↔ List ↔ Kanban)

Switching views within the same project **replaces** the current history entry (not push). Rationale: the user is looking at the same data differently. Browser back should go to the previous *location*, not the previous *view*.

```
User on Graph View       → /projects/abc
Switches to List         → /projects/abc?view=list         (replaces, not pushes)
Browser back             → Gallery (not Graph View)
```

The last-used view per project is persisted in IndexedDB, so re-opening a project returns to the user's preferred view.

### Graph Viewport Restoration

When the user drills down and then comes back (via back button or breadcrumb), the graph should restore:
- Same zoom level
- Same pan position
- Same selected node (if any)

This is stored in the Zustand `graph` store as a **viewport stack** — each drill level saves its viewport state. On drill-up, the previous viewport is restored with a smooth animation.

```typescript
// stores/graph.ts
interface ViewportEntry {
  nodeId: string          // the node we drilled into
  viewport: { x: number; y: number; zoom: number }
  selectedNodeId: string | null
}

interface GraphStore {
  viewportStack: ViewportEntry[]   // pushed on drill-down, popped on drill-up
  // ...
}
```

### Deep Links & Sharing

Every URL is shareable. If User B opens a link from User A:
- `/projects/abc/node/risk-456` → opens project, drills into the correct node
- `/projects/abc/task/task-789` → opens Workspace for that task
- Auth required — Keycloak login redirects back to the deep link after auth
- Org context inferred from the project (if user has access). If not: "You don't have access to this project" screen

### Implementation
- **React Router v7** handles all routing. `loader` functions fetch project/tree data on navigation
- **`useNavigate`** for programmatic navigation (drill-down, workspace open)
- **`history.replaceState`** for view switches and breadcrumb jumps
- **`history.pushState`** for drills and workspace entry
- **Zustand `graph.viewportStack`** for viewport restoration (not in URL — too complex to serialise)

---

## 5. Internationalisation (i18n)

### Strategy
- **Library:** react-intl (FormatJS)
- **Default locale:** `en-GB`
- **Message format:** ICU MessageFormat (plurals, select, dates)
- **File structure:** `messages/{locale}.json` — flat namespace, dot-separated keys
- **Loading:** Lazy-loaded per locale. Default locale (en-GB) bundled. Others fetched on demand
- **RTL:** CSS logical properties throughout (`margin-inline-start` not `margin-left`). Tailwind logical utilities plugin
- **Date/Time:** `Intl.DateTimeFormat` — no moment/dayjs. Relative times via `Intl.RelativeTimeFormat`
- **Numbers:** `Intl.NumberFormat` — percentages, counts

### Key Decisions
- All user-facing strings in message files. No hardcoded text in components
- Component-level namespacing: `chat.input.placeholder`, `graph.zoom.label`
- Locale detection: browser preference → user profile setting → org default
- Locale switcher in Settings → Profile

### Example
```json
{
  "workspace.greeting": "Good {timeOfDay}, {name}",
  "workspace.stats.dueToday": "{count, plural, one {# task} other {# tasks}} due today",
  "hub.progress": "{percent}% complete",
  "task.status.inProgress": "In Progress",
  "chat.placeholder.task": "Ask about this task…",
  "chat.placeholder.general": "Ask about your workflow…"
}
```

---

## 6. Keyboard Navigation

### Global Shortcuts
| Key | Action |
|-----|--------|
| `⌘ + K` | Command palette (search anything) |
| `⌘ + /` | Focus chat input |
| `⌘ + B` | Toggle sidebar |
| `⌘ + .` | Toggle detail panel |
| `⌘ + 1/2/3` | Switch view (Graph/Kanban/List) |
| `Escape` | Close active panel/modal/overlay |

### Graph View
| Key | Action |
|-----|--------|
| `Tab` / `Shift+Tab` | Cycle through nodes |
| `Enter` | Open detail for focused node / drill into group |
| `Backspace` | Drill up one level |
| `Arrow keys` | Pan canvas (when canvas focused) |
| `+` / `-` | Zoom in/out |
| `0` | Fit to screen |
| `Space` | Toggle node selection |

### List View
| Key | Action |
|-----|--------|
| `↑` / `↓` | Move between rows |
| `Enter` | Open detail panel for selected row |
| `←` / `→` | Collapse/expand parent row |
| `Space` | Toggle task checkbox |
| `x` | Mark as done |

### Chat Panel
| Key | Action |
|-----|--------|
| `Enter` | Send message |
| `Shift + Enter` | New line |
| `↑` (empty input) | Edit last message |
| `Escape` | Blur chat input |

### Implementation
- **Library:** Custom `useKeyboardNav` hook + Radix `roving-tabindex` for lists
- **Scope:** Shortcuts are context-aware. Graph shortcuts only fire when graph is focused
- **Discovery:** `?` opens keyboard shortcut overlay (like GitHub)
- **Conflict resolution:** Component focus determines which shortcut scope is active

---

## 7. Accessibility (a11y)

### Standards
- **Target:** WCAG 2.2 Level AA
- **Testing:** axe-core in CI, manual screen reader testing (VoiceOver + NVDA)

### Requirements
| Area | Implementation |
|------|---------------|
| **Semantic HTML** | `<nav>`, `<main>`, `<aside>`, `<section>`, `<header>`. No div soup |
| **Landmarks** | Sidebar = `navigation`, Chat = `complementary`, Content = `main`, Detail = `complementary` |
| **Focus management** | Visible focus rings (2px brand blue). Focus trapped in modals. Focus returned on close |
| **Colour contrast** | All text meets 4.5:1 (AA). Large text 3:1. Status colours paired with shapes/text (never colour alone) |
| **Status indicators** | ○ ▶ ❚❚ ❗ ✓ ✕ — shape + colour + label. Colourblind safe by design |
| **Screen reader** | `aria-label` on icon-only buttons. `aria-live` on chat messages and toast notifications. `role="status"` on hub bar updates |
| **Graph a11y** | Nodes are focusable with `role="treeitem"`. Edge relationships described via `aria-describedby`. Alternative list view always available |
| **Reduced motion** | `prefers-reduced-motion` media query. Disable all transitions/animations. Instant state changes |
| **Keyboard** | Everything operable via keyboard (see §5). No mouse-only interactions |
| **Skip links** | "Skip to main content", "Skip to chat" hidden links at top |
| **Headings** | Proper heading hierarchy per view. One `<h1>` per page |
| **Forms** | All inputs have visible labels. Error messages linked via `aria-describedby`. Required fields marked |
| **Avatars** | Human (circle) vs AI (hexagon) distinction also communicated via `aria-label`: "Assigned to DeepFlow Agent (AI)" |

---

## 8. Responsive Behaviour

### Breakpoints
| Breakpoint | Range | Layout |
|------------|-------|--------|
| **Desktop XL** | ≥ 1440px | Full layout: sidebar + chat + canvas + detail |
| **Desktop** | 1200–1439px | Same layout, panels may flex slightly |
| **Tablet** | 768–1199px | Chat becomes toggleable drawer. Sidebar hidden (hamburger). Detail as full-screen overlay |
| **Mobile** | < 768px | Single-panel view. Bottom nav. Chat as full-screen. No graph canvas (List view default) |

### Panel Behaviour by Breakpoint
| Panel | Desktop (≥1200) | Small window / Tablet (768–1199) | Mobile (<768) |
|-------|---------|--------|--------|
| Sidebar | 56px fixed | 56px fixed (mouse present) or hidden (touch-only) | Hidden, bottom tab bar replaces |
| Chat | 320px fixed | 280px fixed (if mouse) or toggleable drawer (touch) | Full-screen tab |
| Top Bar | Full width | Full width | Simplified (no breadcrumbs) |
| Hub Bar | Collapsible | Collapsed-only, expand = overlay | Sticky card header |
| Graph Canvas | Inline | Inline — full editing if mouse detected | **Not shown** — defaults to List |
| Detail Panel | 360px inline | Full-screen overlay | Bottom sheet |
| Task Table | Full width | Horizontal scroll, fewer columns | Card layout |

### Key Decision: Small Viewports ≠ Touch-Only
Many users run laptops with multiple windows at ~1024px. A small viewport with a mouse pointer is NOT the same as a tablet with touch. Detect input method (`pointer: fine` media query) to decide:
- **Mouse present at small viewport:** keep sidebar (56px), keep chat panel (280px), enable graph editing
- **Touch-only at small viewport:** hide sidebar (hamburger), chat as drawer, graph is read-only

### Implementation
- **CSS:** Tailwind responsive prefixes (`md:`, `lg:`, `xl:`) + `@media (pointer: fine)` for input detection
- **Layout detection:** `useMediaQuery` hook → drives Zustand layout store. Separate `pointer` and `viewport` states
- **Conditional rendering:** Graph canvas component not mounted on mobile (not just hidden). Saves memory/CPU
- **Touch:** Graph canvas supports pinch-to-zoom and two-finger pan on touch devices (read-only navigation)
- **Bottom tab bar (mobile):** Home, Tasks, Chat, Notifications, You

---

## 9. Component Architecture

### Shell (always rendered)
```
<ShellLayout>                         ← app/(app)/layout.tsx
  <Sidebar />                         ← 56px, icon nav, org switcher, user
  <ChatPanel />                        ← 320px, contextual, always visible
  <div className="flex flex-col flex-1">
    <TopBar />                         ← breadcrumbs, view switcher, search
    <HubBar />                         ← conditional: graph/active views only
    <main>{children}</main>            ← routed content
  </div>
  <DetailPanel />                      ← conditional: 360px slide-in
</ShellLayout>
```

### Panel Layout System

The shell uses **react-resizable-panels** for a flexible, resizable panel layout. All panels have drag handles between them.

```
┌─────────┬───────────────┬──────────────────┬───────────────┐
│         │               │                  │               │
│ Sidebar │  Chat Panel   │  Content Area    │ Detail Panel  │
│  56px   │  min:280      │  flex:1          │  min:320      │
│  fixed  │  max:400      │  (graph/list/    │  max:480      │
│         │  default:320  │   dashboard)     │  default:360  │
│         │  ↔ drag       │                  │  ↔ drag       │
│         │               │                  │               │
└─────────┴───────────────┴──────────────────┴───────────────┘
```

**Resizable:** Chat panel and detail panel have drag handles. Users can widen chat to 400px or narrow it to 280px. Detail panel: 320–480px. Sizes persist to IndexedDB via Zustand `preferences` store.

**Collapsible:** Double-click a drag handle to collapse that panel. Chat collapses to 0px (icon in sidebar activates it). Detail collapses to 0px (node click re-opens).

**Constraints:**
- Content area has a minimum width of 320px (below this, panels start collapsing automatically)
- Panel order is fixed: Sidebar → Chat → Content → Detail. No drag-drop reordering (adds complexity without clear UX value for a workflow tool)

### Key Components
| Component | Responsibilities |
|-----------|-----------------|
| `Sidebar` | **Top zone:** Org switcher (logo + name, dropdown to switch). **Mid zone:** Dashboard, Tasks & Projects (expandable → pinned/recent projects), Agents & Automations (expandable → active agents with status dots). **Bottom zone:** Notification bell (dot badge collapsed, count expanded), Settings, User profile/avatar. Collapsed: 56px icon-only with tooltips + hover flyout mini-panels. Expanded: 240px (auto-expand in Settings view only). `aria-label="Main navigation"` |
| `ChatPanel` | Message list, input, context badge, action chips. Subscribes to WS for new messages. `aria-label="AI chat"` |
| `TopBar` | Breadcrumbs (driven by route), view switcher (Graph/Kanban/List), filter button, search. Dynamic per view |
| `HubBar` | Collapsed: progress + stats. Expanded: donut, breakdown, due dates, assignees. Animated toggle |
| `DetailPanel` | Tabs: Overview, Sub-tasks, Files, Comments, History. Task data via TanStack Query. Editable fields |
| `GraphCanvas` | React Flow wrapper. Custom node component. Custom edge component. Level zones. Minimap. Zoom controls |
| `TaskTable` | Virtualized table (TanStack Table). Expandable rows. Sticky headers. Sortable columns. Row selection |
| `ProjectCard` | Health meter, progress bar, status counts, avatar stack. Click → navigate to project |

### Graph Node — Visual Design

**Shape:** Rounded rectangle, consistent size across all nodes (approx 180×72px at 100% zoom).

**Parent vs leaf distinction:**
- **Parent nodes** (have children): stacked card effect — subtle second card shadow behind, plus a child count badge (circled number, bottom-right corner). Double-click to drill in.
- **Leaf nodes**: flat single card. No stack, no badge. Double-click to open Workspace.

**Parallel nodes** (execute per-input, fan-out): fork icon (⑂) in top-right corner. When running, shows live instance count: "⑂ 3/7" (3 of 7 complete). When idle, fork icon only.

**Visible data on node (at default zoom):**
| Element | Position | Notes |
|---------|----------|-------|
| Status symbol | Left edge | ○ pending · ▶ in-progress · ❚❚ paused · ❗ blocked · ✓ complete · ✕ failed — coloured by status. Not a dot — a recognisable symbol |
| Task name | Centre | Truncated with ellipsis if too long. Bold |
| Assignee avatar | Right edge | Circle = human, hexagon = AI agent |
| Progress count | Below name | Only shown when in-progress: "3/7 sub-tasks" |
| User-selected properties | Below progress | Configurable per project — user picks which properties show on nodes. If they don't fit, show "…" (tooltip reveals full list on hover) |
| Parent badge | Bottom-right | Child count in a small circle (parent nodes only) |
| Parallel icon | Top-right | ⑂ icon + instance count when running (parallel nodes only) |

**Node states (visual):**
| State | Visual treatment |
|-------|-----------------|
| Default | White card, thin border |
| Hovered | Subtle shadow lift. Tooltip appears with full detail (all properties, description preview, dependencies) |
| Selected (single-click) | Blue border + glow. Opens detail panel |
| Multi-selected | Blue border (no glow). Part of selection group |
| Overdue | Red accent on due date property (if visible). Red dot on top-left corner |
| Gated/locked | Lock icon overlay. Muted opacity until gate condition met |
| Agent running | Hexagon avatar pulses. Progress count animates |

**Interaction model:**
| Action | Result |
|--------|--------|
| Hover | Tooltip with full task detail |
| Single click | Select node → opens detail panel |
| Double click | Drill in (parent) or open Workspace (leaf) |
| Shift+click | Add to multi-selection |
| Drag-select (rubber band) | Multi-select all nodes in rectangle |
| Right-click | Context menu: Edit, Assign, Change status, Duplicate, Delete, Move to… |
| Drag node | Reposition on canvas (saved to backend) |

### Detail Panel — Specification

The detail panel (360px, resizable to 320–480px) appears when a node is selected. All fields are **editable inline** — no modals.

**Panel layout varies by node type:**

#### Leaf Node Detail Panel
```
┌─────────────────────────────────┐
│ [Task Name]                  ✕  │ ← editable title, close button
├─────────────────────────────────┤
│ Status: [▶ In Progress ▾]      │ ← dropdown, coloured pill
│ Assignee: [👤 Ravi Mehta ▾]    │ ← dropdown with search
│ Due: [15 Mar 2026 📅]          │ ← date picker
│ Priority: [● High ▾]           │ ← dropdown, coloured
├─────────────────────────────────┤
│ Properties                      │
│ Risk Rating: [Critical ▾]      │ ← single-select (coloured pills)
│ Reg Ref: [CRR-123          ]   │ ← free text
│ Tags: [capital] [stress] [+]   │ ← multi-select
│ + Add property                  │
├─────────────────────────────────┤
│ ┌─────┬────┬────┬────┬─────┐   │
│ │ Desc│File│Cmts│Deps│ Hist│   │ ← tabs
│ └─────┴────┴────┴────┴─────┘   │
│                                 │
│ [Active tab content below]      │
└─────────────────────────────────┘
```

#### Parent Node Detail Panel
Same header + properties as leaf, but the default tab is **Sub-tasks** instead of Description:

**Sub-tasks tab (parent nodes):**
```
┌─────────────────────────────────┐
│ Sub-tasks (4)              ▶ All│
├─────────────────────────────────┤
│ ✓ Data Collection    👤RM  Done │ ← click → drills into subflow
│ ▶ Risk Assessment    ⬡AI  Run  │
│ ○ Stress Testing     👤JS  Pend │
│ ○ Report Draft       ⬡AI  Pend │
├─────────────────────────────────┤
│ Progress: ████░░░░ 1/4 (25%)   │
│ Blocked: 0  │  Overdue: 0      │
└─────────────────────────────────┘
```
Each row: status symbol + task name + assignee avatar + status label. Click a row → drills into that node's subflow and selects it.

#### Tab Details

**Description tab:**
- Rich text (markdown rendered)
- Editable inline (click to edit)

**Files tab:**
- Attached files: icon + name + size + date + download link
- URL links: title + favicon preview + URL (auto-fetched via unfurl)
- Upload dropzone ("Drag files or click to upload")
- "+ Add URL" button
- **Inheritance note:** "Files on this task are passed to sub-tasks. Files on the workflow hub are summarised into each task."

**Comments tab:**
- Thread: avatar + name + time + rich text message
- @mentions: users, files, other tasks (autocomplete on @)
- Reactions on comments (emoji picker, click to add — API extension needed)
- No file attachments in comments (yet)
- Input: rich text editor with @ autocomplete

**Dependencies tab:**
- Upstream: list with status symbols + task names (clickable → navigate to that node)
- Downstream: same format
- "+ Add dependency" with task search

**History tab:**
- Timeline: timestamp + actor avatar + action description
- Includes AI actions: "AI inserted content", "Agent completed task", "Property changed by DeepFlow AI"
- Filter: All / Human / AI

### Custom Properties System

Properties are defined per project via API. Four types:

| Type | UI Control | On Node |
|------|-----------|---------|
| **Free text** | Inline text input | Truncated with "…" |
| **Numeric** | Number input with optional unit | Value shown |
| **Single select** | Dropdown with coloured options | Coloured pill |
| **Multi-select** | Tag input with coloured options | Coloured pills, "…" overflow |

- Users can add new select values (and colours) on the fly from the dropdown: "Type to add…"
- Chat can create/modify properties: "Add a 'Regulatory Ref' property to this project"
- Node display: users configure which properties appear on graph nodes (project-level setting). Order is drag-sortable in settings.

### Graph Toolbar

Floating toolbar above the graph canvas (top-centre or top-right). Always visible when graph is active.

```
┌──────────────────────────────────────────────────────────────┐
│ ↶ ↷  │  🔍+ 🔍− ⊞ Fit  │  ⊞ Auto │  ▸ Expand ▾  │  ⋮ More │
│ Undo Redo │ Zoom In/Out/Fit │ Auto-layout│ Expand menu │ Overflow│
└──────────────────────────────────────────────────────────────┘
```

| Tool | Action | Shortcut |
|------|--------|----------|
| **Undo** | Reverts last action (API-backed — sends PATCH to revert) | ⌘Z |
| **Redo** | Re-applies last undone action (API-backed) | ⌘⇧Z |
| **Zoom in** | Increase zoom level | + |
| **Zoom out** | Decrease zoom level | − |
| **Fit to screen** | Zoom + pan to fit all visible nodes in viewport | 0 |
| **Auto-layout** | Re-arrange all nodes using automatic DAG layout algorithm (dagre/elkjs). Confirmation if > 20 nodes moved | ⌘⇧L |
| **Expand selected** | Expand selected parent node inline (see below) | E |
| **Collapse selected** | Collapse expanded parent back to stacked card | E (toggle) |
| **Expand all** | Expand all parent nodes on current level | ⌘E |
| **Collapse all** | Collapse all expanded nodes back to cards | ⌘⇧E |
| **Expand to next level** | Expand all parents one level deeper only | — |
| **More ⋮** | Overflow: minimap toggle, grid snap toggle, export as image | — |

**Undo/Redo scope:** Covers both layout changes (node position) and data changes (status updates, reparenting, property edits, Flow, Fuse, Delete). The API supports undo — each action returns a revert operation. Undo stack is per-session (cleared on navigation away from project). Maximum depth: 50 actions.

### Graph Manipulation — Adding Nodes, Edges, Quick Actions

#### Adding Nodes
| Method | Behaviour |
|--------|-----------|
| **Double-click empty canvas** | Creates a new unconnected node at that position. Inline rename immediately active |
| **Plus button on edge** | Inserts a new node between two connected nodes. The edge arrow (▸) becomes a + button on hover. Click → new node appears mid-edge, inheriting both connections |
| **Chat** | "Add a data validation step after Risk Assessment" → AI creates and connects the node |
| **Right-click canvas** | Context menu: "Add node here" |

#### Edges (Dependencies)
**Connection ports:** Small dots appear on all four sides of a node on hover. Drag from a port to another node to create an edge (dependency).

| Action | Result |
|--------|--------|
| Drag from port to another node | Creates a dependency edge |
| Click an edge | Selects it (highlighted blue). Toolbar shows "Delete edge" (trash icon). Delete key also works |
| Drag port from expanded container's edge to a child inside | Edge flows from parent's port through container boundary to child node |
| Right-click edge | Context menu: Delete, Change type (if multiple edge types supported) |

**Edge-as-plus-button:** When not hovered, edges show a small directional arrow (▸) at their midpoint. On hover, the arrow transforms into a **+** button. Click to insert a new node between the two connected nodes.

#### Hover Action Bar
On hover, a slim floating action bar appears **below the node** (3–4 icon buttons). Disappears on mouse-out. Only the highest-frequency actions — everything else in detail panel or right-click menu.

| Button | Shown when | Action |
|--------|-----------|--------|
| **▶ Start** | Status is pending or ready | Sets status to in-progress |
| **✓ Done** | Status is in-progress | Sets status to complete |
| **↻ Rerun** | Status is complete or failed | Resets to pending and re-triggers |
| **⏎ Open** | Always (parent or leaf) | Drill into subflow (parent) or open Workspace (leaf) |

**Why hover, not always-visible:** Keeps the graph clean. The detail panel handles everything else (reassign, priority, properties, delete). Hover actions are for speed — "mark as done" is hover → click, not click → panel → dropdown → select.

### Inline Node Expansion (Parent → Container)

Parent nodes can be **expanded inline** on the graph canvas, transforming from a stacked card into a container that renders its child sub-graph inside.

**Collapsed state (default):**
```
┌─────────────────┐
│ ○ Task Name   👤│  ← stacked card (shadow behind = has children)
│   ▸ 4 sub-tasks │  ← expand indicator: ▸ chevron + child count
└─┬───────────────┘
  └─(shadow)
```

**Expanded state:**
```
┌─ Task Name ──────── ○ In Progress ─── 👤 RM ─── ▾ ─┐
│                                                       │
│   ┌──────┐     ┌──────┐     ┌──────┐    ┌──────┐    │
│   │ Sub1 │────▶│ Sub2 │────▶│ Sub3 │───▶│ Sub4 │    │
│   └──────┘     └──────┘     └──────┘    └──────┘    │
│                                                       │
└───────────────────────────────────────────────────────┘
```

**Container header:** Parent name + status symbol + assignee avatar + collapse chevron (▾). Styled as a titled region with a subtle tinted background (distinct from the canvas background).

**Key behaviours:**
- **Expand/collapse indicator:** ▸ chevron on collapsed cards (right of child count). ▾ chevron in container header when expanded. Clear visual affordance.
- **Nested expansion:** Expanded containers can contain parents that are themselves expandable. L1 container → expand L2 parent inside it → see L3 nodes inside that. Containers nest.
- **DAG edges flow through containers:** External edges connect to nodes inside expanded containers. An edge from an external node to a child inside a container draws a line that crosses the container boundary. React Flow sub-flows handle this.
- **Container resizes automatically** based on child node layout. Auto-layout (dagre/elkjs) positions children inside the container.
- **Expand preserves position:** When expanding, the container appears where the stacked card was. Other nodes shift to accommodate the larger container. Animated transition (300ms ease).
- **Performance:** Only render child nodes when expanded. Collapsed parents don't mount their children in React Flow. Lazy rendering for deep trees.

**Expand controls:**
| Control | Behaviour |
|---------|-----------|
| Click ▸ on a node | Expand that single parent |
| Toolbar "Expand selected" | Expand all selected parent nodes |
| Toolbar "Expand all" | Expand every parent node on the current level |
| Toolbar "Expand to next level" | Expand all collapsed parents one additional level deeper |
| Toolbar "Collapse all" | Collapse everything back to stacked cards |
| Keyboard E | Toggle expand/collapse on focused/selected node |

### Bulk Editing

Multi-select nodes via **shift+click** or **drag-select** (rubber band) on the graph canvas, or checkbox selection in list view.

**The detail panel becomes a bulk editor:**
```
┌─────────────────────────────────┐
│ 5 tasks selected             ✕  │ ← count + deselect all
├─────────────────────────────────┤
│ Status: [Multiple ▾]           │ ← shows "Multiple" if values differ
│ Assignee: [👤 Ravi Mehta ▾]    │ ← shows value if all selected share it
│ Due: [Multiple 📅]             │
│ Priority: [● High ▾]           │ ← all 5 are High, so shows "High"
├─────────────────────────────────┤
│ Properties                      │
│ Risk Rating: [Multiple ▾]      │
│ Tags: [capital] [+]            │ ← shows intersection of shared tags
├─────────────────────────────────┤
│ [🗑 Delete selected]           │ ← confirmation dialog before executing
└─────────────────────────────────┘
```

**Behaviour:**
- If all selected tasks share the same value → show that value
- If values differ → show "Multiple" (greyed placeholder)
- Changing any field applies **immediately** to all selected tasks (optimistic update, revert on error)
- Delete: confirmation dialog showing count + warning if any are parent nodes ("This will also delete N child tasks")
- Available in both graph and list view

### List View — Drag & Drop

The list view supports drag-and-drop for reparenting tasks:

| Action | Result |
|--------|--------|
| Drag row onto a parent row | Task moves to end of that parent's children |
| Drag row between expanded children | Task inserts at that position |
| Drag row onto a leaf row | Leaf is promoted to parent, dragged task becomes its child |
| Drag multi-selected rows | All selected tasks move together (same rules apply) |

**Visual feedback during drag:**
- Dragged row(s) shown as ghost card following cursor
- Valid drop targets highlighted with blue border
- Insert position shown with a horizontal blue line between rows
- Invalid drops (e.g. dropping a parent into its own child) shown with red indicator

### Task Operations — Flow, Fuse, Delete

#### Flow (Leaf → Parent)
Convert a leaf node into a parent with a sub-workflow. Two modes:

**Flow to blank:**
- Leaf becomes an empty parent node
- User manually adds children via chat, detail panel, or graph

**Flow to AI:**
- AI reads the task description and generates suggested child nodes + dependencies
- Children are created immediately (no preview dialog)
- User reviews and edits via chat: "Remove the third step", "Add a data validation step before report"
- Accessible via: right-click context menu → "Flow…", or command palette, or chat ("flow this task")

#### Fuse (Wrap siblings in new parent)
Select two or more sibling nodes and wrap them in a new parent. The new node takes their place in the dependency chain.

**Example:** A → B → C → D. Select B & C, Fuse:
- Result: A → **[New Parent]** → D
- [New Parent] contains B → C as children
- AI suggests the new parent's name based on the children's descriptions
- User can rename inline after creation

**Constraints:**
- Selected nodes must be siblings (same parent) or sequential in the dependency chain
- The new parent inherits the upstream dependencies of the first selected node and passes downstream dependencies from the last selected node

#### Delete
- **Single node:** Disconnects from all dependencies (upstream and downstream become orphaned), then deletes. Downstream nodes lose their upstream connection — they do NOT auto-reconnect.
- **Parent node:** Deletes the parent AND all children recursively. Confirmation dialog warns: "This will delete [Node] and N child tasks. This cannot be undone."
- **Bulk delete:** Confirmation dialog with count. If any selected nodes are parents, warns about cascading child deletion.
- All deletes require confirmation dialog. No undo (server-side delete is permanent), but tasks could be soft-deleted/archived in a future phase.

---

## 10. Data Flow

### API Layer
- **Direct API calls:** Fetch client with Keycloak token interceptor. No BFF — SPA talks directly to backend API
- **REST endpoints:** `/api/projects`, `/api/projects/:id/tree` (full DAG), `/api/tasks/:id`, `/api/chat/messages`
- **Tree endpoint:** `GET /api/projects/:id/tree` returns the full normalised task tree in one response. Typically 10–100KB for 50–500 nodes
- **Response format:** JSON with included relationships. Tree is flat `nodes[]` array with parent/child/dependency IDs
- **Error handling:** Typed error responses. Toast for user-facing errors. Retry for transient failures

### Data Loading Flow
```
1. User opens project
   → GET /api/projects/:id/tree
   → Response: { nodes: TaskNode[], seq: number }
   → Zustand tree.loadTree() normalises into Record<id, TaskNode>
   → React Flow renders visible nodes from store
   → List View derives sorted/filtered rows from same store

2. WebSocket connects (scoped to project)
   → WS subscribes to project channel
   → Server pushes patches as they happen

3. WS event: tree.patch { seq: 42, ops: [{ op: 'update', id, changes }] }
   → Check seq === lastSeq + 1 (no gaps)
   → Apply patch directly to Zustand store: tree.patchNode(id, changes)
   → All views re-render via selectors (React Flow, List, Hub Bar)
   → If seq gap detected: request full tree resync

4. WS event: chat.message { role, content, taskId }
   → Append to chat store
   → Scroll to bottom
   → If action chips: render inline buttons

5. WS event: presence.update { userId, status, cursor? }
   → Update presence store
   → Render avatar in top bar (Phase 2) / cursor on canvas (Phase 3)
```

### Optimistic Updates
- Task status change: patch Zustand store immediately → send API request → if rejected, revert patch + toast
- Chat message send: append to message list immediately → confirm on server ack → if failed, mark as failed
- Detail panel edits: save on blur/submit → optimistic patch → revert on error
- Node drag (graph): update position in store immediately → debounce API save (500ms)

---

## 11. Performance Budget

| Metric | Target | Measurement |
|--------|--------|-------------|
| **LCP** | < 2.5s | Largest contentful paint (dashboard cards or graph canvas) |
| **FID** | < 100ms | First input delay |
| **CLS** | < 0.1 | No layout shift from panel rendering |
| **JS bundle (initial)** | < 200KB gzipped | Code-split by route. React Flow lazy-loaded |
| **TTI** | < 3.5s | Time to interactive on 4G |

### Strategies
- **Code splitting:** React Flow + graph components lazy-loaded (only on graph routes)
- **Virtualisation:** Task table uses `@tanstack/react-virtual` for 1000+ row lists
- **Image optimisation:** Next.js `<Image>` for avatars and thumbnails
- **Font loading:** `font-display: swap`. Subset Euclid Circular B to Latin
- **Prefetch:** Prefetch adjacent routes on hover (Next.js `<Link>`)

---

## 12. Testing Strategy

| Layer | Tool | Coverage |
|-------|------|----------|
| **Unit** | Vitest | Zustand stores, utility functions, hooks |
| **Component** | Testing Library + Vitest | All components. Keyboard nav. a11y assertions |
| **Integration** | Playwright | Critical paths: login → dashboard → project → task → chat interaction |
| **Visual regression** | Playwright screenshots | Component snapshots at each breakpoint |
| **a11y** | axe-core (CI) + manual | Every component scanned. Screen reader testing quarterly |
| **Performance** | Lighthouse CI | Budget enforcement on every PR |

---

## 13. Design Tokens → Code

The [design guidelines](index.html) define tokens. Map them to Tailwind + CSS vars:

```css
/* styles/tokens.css — generated from guidelines */
:root {
  /* Brand */
  --brand-cyan: #00E5FF;
  --brand-blue: #2244FF;
  --brand-mid: #0099FF;
  --brand-gradient: linear-gradient(135deg, #00E5FF 0%, #0099FF 45%, #2244FF 100%);
  
  /* Status */
  --status-complete: #059669;
  --status-active: #2563EB;
  --status-blocked: #D97706;
  --status-failed: #DC2626;
  --status-pending: #9CA3AF;
  --status-approval: #DC2626;
  
  /* Spacing (4px base) */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;
  
  /* Radii */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-full: 9999px;
  
  /* Elevation */
  --shadow-sm: 0 1px 3px rgba(0,0,0,0.06);
  --shadow-md: 0 4px 12px rgba(0,0,0,0.08);
  --shadow-lg: 0 8px 32px rgba(0,0,0,0.12);
  
  /* Typography */
  --font-family: 'Euclid Circular B', -apple-system, sans-serif;
  --text-xs: 0.6875rem;   /* 11px */
  --text-sm: 0.8125rem;   /* 13px */
  --text-base: 0.875rem;  /* 14px */
  --text-lg: 1.125rem;    /* 18px */
  --text-xl: 1.5rem;      /* 24px */
  --text-2xl: 1.875rem;   /* 30px */
}
```

---

## 14. Decisions (Resolved)

| # | Question | Decision |
|---|----------|----------|
| 1 | **Kanban view** | Defer to Phase 2. Keep placeholder in view switcher (greyed out). Also add **Map view** as a future placeholder |
| 2 | **Collaboration cursors** | Phase 3. Build presence indicators first (avatars in top bar). Cursors are higher-frequency WS events on the same infrastructure — add as polish. Throttle to 10fps, canvas-space coordinates, fade after 3s idle |
| 3 | **Offline support** | Online-only. Chat requires connection. IndexedDB is local cache for preferences only |
| 4 | **Chat history** | Stored in backend DB. Frontend fetches via API with pagination. Context scoped per-task in UI but full history queryable |
| 5 | **Mobile app** | Native app needed long-term. Phase 1: responsive web. Phase 2: evaluate React Native / Expo. Layout must support native from the start |
| 6 | **Tablet / small viewport** | Full mouse support at tablet sizes. Many users run laptop with multiple windows at ~1024px. Not touch-only — graph editing enabled if mouse is present. Responsive breakpoints accommodate small-windowed desktop, not just tablets |
| 7 | **Notifications** | Multi-channel: in-app bell + email + Slack. Configurable per notification type per user in Settings. With native mobile app, also push notifications. Settings UI: matrix of notification types × channels |
| 8 | **Backend API spec** | Assume it exists. A separate agent will extract API contracts from the repo. Frontend defines TypeScript interfaces; backend is source of truth |

---

## 15. Phasing

### Phase 1 — Shell + Core Views (4–6 weeks)
- [ ] Auth (Keycloak + next-auth)
- [ ] Shell layout (sidebar, chat panel, top bar)
- [ ] Workspace (dashboard)
- [ ] Gallery (project browser)
- [ ] Settings (layout toggle, preferences)
- [ ] i18n scaffolding (en-GB)
- [ ] Responsive shell (all breakpoints)
- [ ] Keyboard shortcuts (global)
- [ ] Design tokens integrated

### Phase 2 — Graph + List Views (4–6 weeks)
- [ ] React Flow integration (custom nodes, edges, zones)
- [ ] Graph navigation (drill-down, breadcrumbs, hub bar)
- [ ] List view (table, expandable rows, sort/filter)
- [ ] Detail panel (all tabs)
- [ ] WebSocket integration (live updates)
- [ ] Active task view
- [ ] Graph keyboard navigation

### Phase 3 — Chat + Collaboration + Polish (4–5 weeks)
- [ ] Chat panel (message rendering, action chips, context switching)
- [ ] Optimistic updates
- [ ] Collaboration presence (user avatars in top bar showing who's viewing)
- [ ] Collaboration cursors (canvas-space coordinates, 10fps throttle, 3s fade)
- [ ] Notification settings UI (matrix: notification types × channels: in-app/email/Slack/push)
- [ ] Accessibility audit + fixes
- [ ] Performance optimisation (code splitting, virtualisation)
- [ ] Visual regression tests
- [ ] Additional locales (if needed)
- [ ] Kanban view placeholder → implementation (if prioritised)
- [ ] Map view placeholder (Phase 4+)
