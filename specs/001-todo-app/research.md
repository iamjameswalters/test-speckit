# Research: To-Do List App

**Feature**: 001-todo-app
**Date**: 2025-10-24
**Purpose**: Resolve technical unknowns from Technical Context and establish best practices

## Research Questions

1. **Language/Framework Choice**: What technology stack best supports our requirements?
2. **Storage Strategy**: LocalStorage vs IndexedDB for 1000+ tasks?
3. **Testing Approach**: What testing framework and strategy suits this project?
4. **Performance Optimization**: How to ensure sub-2-second rendering for 1000 tasks?

---

## Decision 1: Language and Framework

**Decision**: Vanilla JavaScript (ES2020+) with optional TypeScript for type safety

**Rationale**:
- **Simplicity**: No build complexity for a single-feature app; direct browser execution
- **Performance**: No framework overhead; direct DOM manipulation can be highly optimized
- **Requirements fit**: Feature set (CRUD, filtering, persistence) doesn't require complex state management
- **Learning curve**: Lower barrier for future contributors
- **Bundle size**: Zero framework weight improves initial load time
- **Modern features**: ES2020+ provides classes, modules, async/await, optional chaining

**Alternatives Considered**:

| Framework | Why Not Selected |
|-----------|-----------------|
| **React** | Overkill for simple CRUD operations; adds 40KB+ min+gzip overhead; requires build tooling; state management adds complexity not needed for local-only app |
| **Vue 3** | Similar concerns to React but lighter (33KB); still requires build step for SFC; composition API adds unnecessary abstraction for this scope |
| **Svelte** | Requires build tooling; compiler adds development complexity; learning curve for future maintainers; performance gains negligible for this app size |
| **TypeScript** | Keep as optional; can add `.ts` files later if type safety becomes important; not essential for 500-line codebase |

**Best Practices**:
- Use ES modules for code organization (`import/export`)
- Leverage modern DOM APIs (`querySelector`, `addEventListener`, `classList`)
- Use template literals for HTML generation
- Consider adding TypeScript later if codebase grows beyond 1000 LOC

---

## Decision 2: Storage Strategy

**Decision**: IndexedDB with LocalStorage fallback

**Rationale**:
- **Capacity**: LocalStorage has 5-10MB limit; IndexedDB supports 50MB+ (varies by browser)
- **Performance**: IndexedDB handles 1000+ records efficiently with indexing; LocalStorage serializes all data on every read/write
- **Scalability**: IndexedDB supports queries, transactions, and indexes; critical for filtering 1000 tasks
- **Asynchronous**: IndexedDB is async by default; won't block UI during large read/write operations
- **Structured data**: IndexedDB stores objects directly; no JSON.parse/stringify overhead

**Alternatives Considered**:

| Storage | Why Not Selected |
|---------|-----------------|
| **LocalStorage only** | 5MB limit may be reached with 1000 tasks (especially with metadata); synchronous API blocks UI thread; no indexing makes filtering slow; no transaction support |
| **SessionStorage** | Data lost on tab close; doesn't meet persistence requirement |
| **Cookies** | 4KB limit per cookie; not designed for application data |
| **WebSQL** | Deprecated; no longer supported in modern browsers |
| **Cache API** | Designed for HTTP responses, not application data |

**Implementation Pattern**:
```javascript
// Wrapper class providing clean API
class TaskStorage {
  async init() { /* Open IndexedDB */ }
  async create(task) { /* Add task */ }
  async getAll() { /* Return all tasks */ }
  async update(taskId, updates) { /* Update task */ }
  async delete(taskId) { /* Remove task */ }
  async filter(predicate) { /* Query with filter */ }
}
```

**Best Practices**:
- Wrap IndexedDB in Promise-based API for cleaner async/await usage
- Add LocalStorage fallback for older browsers
- Use object stores with auto-incrementing keys
- Create index on `completed` status for fast filtering
- Handle quota exceeded errors gracefully

---

## Decision 3: Testing Framework and Strategy

**Decision**: Vitest + Playwright

**Rationale**:
- **Vitest**: Modern, fast, ESM-native test runner with Jest-compatible API
- **Playwright**: Browser automation for E2E tests; supports all major browsers
- **No build step conflict**: Vitest handles ES modules natively
- **Speed**: Vitest is 10x faster than Jest for ESM projects
- **Developer experience**: Hot module reload, parallel execution, built-in coverage

**Test Strategy**:

| Layer | Framework | Coverage | Examples |
|-------|-----------|----------|----------|
| **Unit** | Vitest | Models, storage layer | Task validation, IndexedDB wrapper methods |
| **Integration** | Vitest + JSDOM | Service + storage | TaskService.create() → IndexedDB → retrieve |
| **E2E** | Playwright | Full user flows | Create task → mark complete → filter → delete |

**Alternatives Considered**:

| Framework | Why Not Selected |
|-----------|-----------------|
| **Jest** | Slower for ESM; requires babel/transforms; not ESM-native; Vitest is Jest-compatible drop-in with better performance |
| **Mocha + Chai** | More boilerplate; separate assertion library; no built-in coverage; Vitest provides all-in-one solution |
| **Cypress** | Heavier than Playwright; slower startup; less multi-browser support; Playwright is faster and more flexible |
| **Testing Library** | Useful but not essential for vanilla JS; can add later if component complexity increases |

**Best Practices**:
- Write tests before implementation (TDD approach)
- Target 80%+ code coverage for models and services
- E2E tests for all 5 user stories from spec
- Mock IndexedDB for unit tests; use real DB for integration tests
- Test edge cases: empty titles, 1000+ tasks, storage quota exceeded

---

## Decision 4: Performance Optimization

**Decision**: Virtual scrolling + efficient rendering

**Rationale**:
- **Virtual scrolling**: Only render visible tasks (~20-30 items); dramatically reduces DOM nodes
- **Rendering budget**: 2 seconds for 1000 tasks = 2ms per task if all rendered; virtual scrolling makes this <100ms
- **IndexedDB indexing**: Query performance critical; indexed `completed` field for O(log n) filtering
- **Debouncing**: Debounce filter changes to avoid rapid re-renders

**Implementation Approach**:

| Technique | Purpose | Impact |
|-----------|---------|--------|
| **Virtual scrolling** | Render only visible items | Reduces 1000 DOM nodes to ~30 |
| **IndexedDB indexes** | Fast filtering | O(log n) vs O(n) for status filter |
| **Event delegation** | Single listener for all tasks | Reduces memory footprint |
| **RequestAnimationFrame** | Batch DOM updates | Smooth 60fps interactions |
| **CSS containment** | Isolate repaint areas | Faster browser rendering |

**Alternatives Considered**:

| Approach | Why Not Selected |
|----------|-----------------|
| **Render all 1000 tasks** | Would create 1000+ DOM nodes; exceeds 2-second budget even on fast devices; poor scrolling performance |
| **Pagination** | Worse UX than virtual scrolling; requires clicks to navigate; doesn't allow quick scanning |
| **Web Workers** | Overkill for this app; complexity not justified; IndexedDB already async |
| **Framework virtual lists** | Adds framework dependency; vanilla JS virtual scrolling is ~100 LOC |

**Virtual Scrolling Algorithm**:
```javascript
class VirtualList {
  constructor(container, itemHeight, totalItems) {
    this.container = container;
    this.itemHeight = itemHeight;
    this.totalItems = totalItems;
    this.visibleCount = Math.ceil(container.clientHeight / itemHeight) + 2; // +2 buffer
  }

  render(scrollTop) {
    const startIndex = Math.floor(scrollTop / this.itemHeight);
    const endIndex = Math.min(startIndex + this.visibleCount, this.totalItems);
    // Render only items from startIndex to endIndex
  }
}
```

**Best Practices**:
- Profile with Chrome DevTools Performance tab
- Use `performance.mark()` and `performance.measure()` for timing
- Test on low-end devices (throttle CPU 4x in DevTools)
- Lazy load images/icons if added in future
- Avoid layout thrashing (batch reads, then writes)

---

## Decision 5: Build and Development Setup

**Decision**: Minimal tooling with optional build step

**Rationale**:
- **Development**: Direct file serving via Python HTTP server or npx serve
- **Production**: Optional bundling with esbuild for single-file distribution
- **No framework build**: Avoid webpack/vite/rollup complexity for simple app
- **Future-proof**: Can add build step later if needed

**Development Workflow**:
```bash
# Development (no build)
python3 -m http.server 8000
# or
npx serve public

# Optional production build
npx esbuild src/app.js --bundle --minify --outfile=public/bundle.js
```

**Alternatives Considered**:

| Approach | Why Not Selected |
|----------|-----------------|
| **Vite dev server** | Adds build tooling dependency; not needed for vanilla JS |
| **Webpack** | Complex configuration; slow; overkill for this project size |
| **Parcel** | Auto-magic can cause confusion; prefer explicit control |
| **No dev server** | File:// protocol has CORS issues with modules |

**Best Practices**:
- Use ES modules directly in browser (modern browsers support)
- Add `type="module"` to script tags
- Use `.js` extensions in import statements
- Keep dev experience simple: edit → refresh → test

---

## Architecture Decisions

### Component Structure

**Decision**: Model-View-Service architecture (no framework)

```
models/Task.js          → Task entity with validation
storage/TaskStorage.js  → IndexedDB wrapper
services/TaskService.js → Business logic (CRUD operations)
ui/TaskList.js          → Virtual list component
ui/TaskItem.js          → Single task rendering
ui/FilterBar.js         → Filter controls
app.js                  → Entry point, initialization
```

**Rationale**: Clean separation of concerns; each module has single responsibility; testable in isolation

### State Management

**Decision**: Service-layer state with custom events

**Pattern**:
```javascript
class TaskService extends EventTarget {
  async createTask(title) {
    const task = await this.storage.create({title, completed: false});
    this.dispatchEvent(new CustomEvent('taskCreated', {detail: task}));
    return task;
  }
}
```

**Rationale**: No state library needed; browser's EventTarget API provides pub/sub; UI listens for changes and re-renders

---

## Security Considerations

### Input Validation

**Requirements**:
- **XSS Prevention**: Sanitize task titles before rendering
- **Title length**: Enforce 1-500 character limit (FR-001)
- **Empty titles**: Reject per FR-008

**Implementation**:
```javascript
function sanitizeHTML(str) {
  const temp = document.createElement('div');
  temp.textContent = str;
  return temp.innerHTML;
}
```

### Storage Security

**Considerations**:
- **No sensitive data**: To-do titles are not confidential
- **No authentication**: Single-user, local-only app
- **IndexedDB isolation**: Per-origin storage (same-origin policy applies)

---

## Browser Compatibility

**Target**: Modern browsers from last 2 years (per spec assumptions)

**Required APIs**:
- ES2020 features (optional chaining, nullish coalescing)
- IndexedDB (supported since IE10, all modern browsers)
- ES Modules (supported in all modern browsers)
- CSS Grid and Flexbox

**Fallback Strategy**:
- Detect IndexedDB support on initialization
- Fall back to LocalStorage if IndexedDB unavailable
- Show warning if storage quota exceeded

---

## Summary of Technical Decisions

| Category | Decision | Key Reason |
|----------|----------|------------|
| **Language** | Vanilla JavaScript ES2020+ | Simplicity, no build complexity, sufficient for scope |
| **Storage** | IndexedDB (LocalStorage fallback) | Handles 1000+ tasks, async, indexing for performance |
| **Testing** | Vitest + Playwright | Fast, modern, ESM-native, comprehensive coverage |
| **Performance** | Virtual scrolling | Meets <2s render requirement for 1000 tasks |
| **Build** | Optional (esbuild) | Keep development simple, bundle for production if needed |
| **Architecture** | Model-View-Service | Clean separation, testable, no framework needed |

## Next Steps (Phase 1)

1. Create detailed data model (Task entity)
2. Define service contracts (TaskService interface)
3. Generate API documentation (JSDoc comments)
4. Create quickstart guide for development setup

---

**Research completed**: 2025-10-24
**Reviewed by**: N/A (automated workflow)
**All NEEDS CLARIFICATION items resolved**: ✅
