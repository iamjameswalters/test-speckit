# Quickstart Guide: To-Do List App

**Feature**: 001-todo-app
**Date**: 2025-10-24
**Target Audience**: Developers implementing the to-do list app

## Overview

This guide provides step-by-step instructions to set up your development environment and start implementing the to-do list application. Follow these steps to go from zero to running tests in under 10 minutes.

---

## Prerequisites

### Required

- **Modern web browser**: Chrome 90+, Firefox 88+, Safari 14+, or Edge 90+
- **Text editor**: VS Code, Sublime Text, or any code editor
- **Basic HTTP server**: Python 3.6+ OR Node.js 14+ (for development server)

### Optional

- **Git**: For version control
- **Node.js 14+**: If you want to use npm packages (Vitest, Playwright)

---

## Project Setup

### Step 1: Create Project Structure

```bash
# Create project directory
mkdir todo-app
cd todo-app

# Create directory structure
mkdir -p src/{models,storage,services,ui}
mkdir -p tests/{unit,integration,e2e}
mkdir -p public/styles

# Create files
touch src/models/Task.js
touch src/storage/TaskStorage.js
touch src/services/TaskService.js
touch src/ui/{TaskList.js,TaskItem.js,FilterBar.js}
touch src/app.js
touch public/index.html
touch public/styles/main.css
```

**Result**: Your directory structure should look like:
```
todo-app/
├── src/
│   ├── models/
│   │   └── Task.js
│   ├── storage/
│   │   └── TaskStorage.js
│   ├── services/
│   │   └── TaskService.js
│   ├── ui/
│   │   ├── TaskList.js
│   │   ├── TaskItem.js
│   │   └── FilterBar.js
│   └── app.js
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
└── public/
    ├── index.html
    └── styles/
        └── main.css
```

---

### Step 2: Create HTML Entry Point

Create `public/index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>To-Do List</title>
  <link rel="stylesheet" href="styles/main.css">
</head>
<body>
  <div id="app">
    <header>
      <h1>My To-Do List</h1>
    </header>

    <main>
      <div id="task-input-container">
        <input
          type="text"
          id="new-task-input"
          placeholder="What needs to be done?"
          maxlength="500"
        >
        <button id="add-task-btn">Add Task</button>
      </div>

      <div id="filter-bar">
        <button class="filter-btn active" data-filter="all">All</button>
        <button class="filter-btn" data-filter="active">Active</button>
        <button class="filter-btn" data-filter="completed">Completed</button>
      </div>

      <div id="task-list-container">
        <!-- Tasks will be rendered here -->
      </div>

      <div id="task-count">
        <span id="active-count">0</span> tasks remaining
      </div>
    </main>
  </div>

  <script type="module" src="../src/app.js"></script>
</body>
</html>
```

---

### Step 3: Create Basic CSS

Create `public/styles/main.css`:

```css
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
               'Helvetica Neue', Arial, sans-serif;
  line-height: 1.6;
  color: #333;
  background: #f5f5f5;
  padding: 20px;
}

#app {
  max-width: 600px;
  margin: 0 auto;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  padding: 20px;
}

header h1 {
  text-align: center;
  color: #2c3e50;
  margin-bottom: 30px;
}

#task-input-container {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

#new-task-input {
  flex: 1;
  padding: 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

#new-task-input:focus {
  outline: none;
  border-color: #3498db;
}

#add-task-btn {
  padding: 12px 24px;
  background: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

#add-task-btn:hover {
  background: #2980b9;
}

#filter-bar {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 1px solid #eee;
}

.filter-btn {
  padding: 8px 16px;
  background: transparent;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.filter-btn.active {
  background: #3498db;
  color: white;
  border-color: #3498db;
}

.task-item {
  display: flex;
  align-items: center;
  padding: 12px;
  border-bottom: 1px solid #eee;
  transition: background 0.2s;
}

.task-item:hover {
  background: #f9f9f9;
}

.task-item.completed {
  opacity: 0.6;
}

.task-checkbox {
  width: 20px;
  height: 20px;
  cursor: pointer;
  margin-right: 12px;
}

.task-title {
  flex: 1;
  font-size: 16px;
}

.task-item.completed .task-title {
  text-decoration: line-through;
  color: #999;
}

.task-actions {
  display: flex;
  gap: 8px;
}

.task-btn {
  padding: 4px 8px;
  background: transparent;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
}

.task-btn:hover {
  background: #f0f0f0;
}

.delete-btn {
  color: #e74c3c;
}

.delete-btn:hover {
  background: #ffebee;
  border-color: #e74c3c;
}

#task-count {
  margin-top: 20px;
  padding-top: 15px;
  border-top: 1px solid #eee;
  text-align: center;
  color: #666;
  font-size: 14px;
}
```

---

## Development Setup

### Option 1: Python HTTP Server (No Installation)

```bash
# From project root
cd public
python3 -m http.server 8000

# Open browser to http://localhost:8000
```

### Option 2: Node.js HTTP Server

```bash
# Install serve globally
npm install -g serve

# Run from project root
serve public -p 8000

# Open browser to http://localhost:8000
```

### Option 3: VS Code Live Server Extension

1. Install "Live Server" extension in VS Code
2. Right-click `public/index.html`
3. Select "Open with Live Server"

---

## Implementation Workflow

### Phase 1: Task Model (Test-First)

1. **Write tests** in `tests/unit/Task.test.js`:
   ```javascript
   import { describe, it, expect } from 'vitest';
   import { Task } from '../../src/models/Task.js';

   describe('Task', () => {
     it('should create a task with title', () => {
       const task = new Task({ title: 'Buy groceries' });
       expect(task.title).toBe('Buy groceries');
       expect(task.completed).toBe(false);
     });

     it('should reject empty title', () => {
       expect(() => new Task({ title: '' }))
         .toThrow('Task title cannot be empty');
     });
   });
   ```

2. **Implement** `src/models/Task.js` (see Task.contract.md)

3. **Run tests**:
   ```bash
   npm test tests/unit/Task.test.js
   ```

4. **Iterate** until all tests pass

---

### Phase 2: Storage Layer

1. **Write tests** in `tests/unit/TaskStorage.test.js`
2. **Implement** `src/storage/TaskStorage.js` (see TaskStorage.contract.md)
3. **Test** with real IndexedDB (integration tests)

**Testing IndexedDB**:
```javascript
import { describe, it, expect, beforeEach } from 'vitest';
import { TaskStorage } from '../../src/storage/TaskStorage.js';

describe('TaskStorage', () => {
  let storage;

  beforeEach(async () => {
    storage = new TaskStorage();
    await storage.init();
    await storage.clear(); // Clean slate
  });

  it('should create and retrieve task', async () => {
    const task = await storage.create({
      title: 'Test task',
      completed: false,
      createdAt: Date.now(),
      updatedAt: Date.now()
    });

    const retrieved = await storage.getById(task.id);
    expect(retrieved.title).toBe('Test task');
  });
});
```

---

### Phase 3: Service Layer

1. **Write tests** in `tests/unit/TaskService.test.js`
2. **Implement** `src/services/TaskService.js` (see TaskService.contract.md)
3. **Test** event dispatching and validation

---

### Phase 4: UI Components

1. **Implement** UI components in `src/ui/`
2. **Wire up** event listeners
3. **Test** with E2E tests (Playwright)

---

## Running Tests

### Setup Testing (Once)

```bash
# Initialize package.json
npm init -y

# Install test dependencies
npm install -D vitest @vitest/ui jsdom
npm install -D playwright @playwright/test
```

### Run Unit Tests

```bash
# Run all tests
npm test

# Watch mode
npm test -- --watch

# With coverage
npm test -- --coverage

# With UI
npm test -- --ui
```

### Run E2E Tests

```bash
# Install browsers (once)
npx playwright install

# Run E2E tests
npx playwright test

# Interactive mode
npx playwright test --ui
```

---

## Development Tips

### Debugging

**Browser DevTools**:
1. Open browser console (F12)
2. Check "Application" tab for IndexedDB contents
3. Use `performance.mark()` and `performance.measure()` for timing

**VS Code Debugging**:
1. Install "Debugger for Chrome" extension
2. Add breakpoints in `.js` files
3. Press F5 to start debugging

### Hot Reload

Use a dev server with hot reload:
```bash
# Install Vite
npm install -D vite

# Run dev server
npx vite public

# Open http://localhost:5173
```

### Testing Individual Modules

```javascript
// In browser console
import { Task } from './src/models/Task.js';
const task = new Task({ title: 'Test' });
console.log(task);
```

---

## Common Issues

### Issue: "Failed to resolve module specifier"

**Cause**: Missing `.js` extension in import
**Fix**: Always include `.js` in imports:
```javascript
// ❌ Wrong
import { Task } from './models/Task';

// ✅ Correct
import { Task } from './models/Task.js';
```

### Issue: "CORS policy" error

**Cause**: Opening `index.html` directly (file:// protocol)
**Fix**: Use a web server (Python, Node, or VS Code extension)

### Issue: IndexedDB not working

**Cause**: IndexedDB blocked in private/incognito mode
**Fix**: Test in regular browser window or use LocalStorage fallback

### Issue: Module not found in tests

**Cause**: Missing module resolution in Vitest config
**Fix**: Create `vitest.config.js`:
```javascript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'jsdom',
    globals: true
  }
});
```

---

## Performance Testing

### Test with 1000 Tasks

```javascript
// Create 1000 tasks for testing
async function createTestTasks(storage) {
  const start = performance.now();

  for (let i = 0; i < 1000; i++) {
    await storage.create({
      title: `Task ${i}`,
      completed: i % 2 === 0,
      createdAt: Date.now(),
      updatedAt: Date.now()
    });
  }

  const end = performance.now();
  console.log(`Created 1000 tasks in ${end - start}ms`);
}

// Test retrieval
async function testRetrieval(storage) {
  const start = performance.now();
  const tasks = await storage.getAll();
  const end = performance.now();

  console.log(`Retrieved ${tasks.length} tasks in ${end - start}ms`);
}
```

### Chrome DevTools Performance

1. Open DevTools → Performance tab
2. Click Record
3. Perform operations (create, filter, etc.)
4. Stop recording
5. Analyze flame graph and timeline

---

## Next Steps

1. **Read contracts**: Review all `.contract.md` files in `specs/001-todo-app/contracts/`
2. **Implement Task model**: Start with `src/models/Task.js`
3. **Add tests**: Follow TDD approach (test first, then implement)
4. **Iterate**: Complete each layer before moving to next
5. **Test performance**: Validate with 1000 tasks
6. **Polish UI**: Add animations, error messages, loading states

---

## Resources

### Documentation Files

- [Feature Specification](spec.md) - User requirements and success criteria
- [Implementation Plan](plan.md) - Technical architecture and design decisions
- [Research Document](research.md) - Technology choices and rationale
- [Data Model](data-model.md) - Task entity and storage schema
- [Contracts](contracts/) - Service interfaces and API contracts

### External Resources

- [IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [ES Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [Vitest Documentation](https://vitest.dev/)
- [Playwright Documentation](https://playwright.dev/)

---

## Getting Help

### Common Commands

```bash
# Start dev server
python3 -m http.server 8000

# Run tests
npm test

# Run tests in watch mode
npm test -- --watch

# Run E2E tests
npx playwright test

# Check coverage
npm test -- --coverage
```

### Troubleshooting

1. Clear IndexedDB: Application tab → IndexedDB → Delete database
2. Clear cache: Hard refresh (Ctrl+Shift+R or Cmd+Shift+R)
3. Check console: Look for error messages
4. Verify structure: Ensure all files exist in correct locations

---

**Quickstart completed**: 2025-10-24
**Estimated setup time**: 5-10 minutes
**Ready to code**: ✅
