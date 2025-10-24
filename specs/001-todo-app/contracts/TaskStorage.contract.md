# TaskStorage Contract

**Feature**: 001-todo-app
**Date**: 2025-10-24
**Purpose**: Storage abstraction layer for task persistence

## Overview

`TaskStorage` provides a unified interface for persisting tasks using IndexedDB (primary) with LocalStorage fallback. It abstracts storage implementation details from the service layer.

## Interface

### Constructor

```javascript
/**
 * Creates a new TaskStorage instance
 * @param {Object} [options] - Configuration options
 * @param {boolean} [options.useLocalStorage=false] - Force LocalStorage instead of IndexedDB
 */
constructor(options = {})
```

**Example**:
```javascript
// Use IndexedDB (default)
const storage = new TaskStorage();

// Force LocalStorage
const storage = new TaskStorage({ useLocalStorage: true });
```

---

### Methods

#### init

```javascript
/**
 * Initializes the storage backend
 * Opens IndexedDB connection or verifies LocalStorage access
 *
 * @returns {Promise<void>}
 * @throws {StorageError} If storage initialization fails
 */
async init()
```

**Preconditions**:
- None (first method called)

**Postconditions**:
- IndexedDB connection established OR LocalStorage verified
- Object stores and indexes created if first run
- Storage is ready for operations

**Example**:
```javascript
const storage = new TaskStorage();
await storage.init();
// Storage is now ready
```

**Error Cases**:
```javascript
// IndexedDB blocked
await storage.init();
// Throws: StorageError("IndexedDB is not available")
```

---

#### create

```javascript
/**
 * Creates a new task in storage
 *
 * @param {Object} taskData - Task data without ID
 * @param {string} taskData.title - Task title
 * @param {boolean} taskData.completed - Completion status
 * @param {number} taskData.createdAt - Creation timestamp
 * @param {number} taskData.updatedAt - Update timestamp
 * @returns {Promise<Task>} The created task with generated ID
 * @throws {StorageError} If storage quota exceeded or operation fails
 */
async create(taskData)
```

**Preconditions**:
- Storage is initialized
- `taskData` contains all required fields except `id`

**Postconditions**:
- Task is persisted in storage
- Task has unique auto-generated ID
- IndexedDB indexes updated

**Performance**: O(1) - Target <5ms

**Example**:
```javascript
const task = await storage.create({
  title: "Buy groceries",
  completed: false,
  createdAt: Date.now(),
  updatedAt: Date.now()
});
// Returns: { id: 42, title: "Buy groceries", ... }
```

---

#### getAll

```javascript
/**
 * Retrieves all tasks from storage
 *
 * @param {Object} [options] - Query options
 * @param {string} [options.orderBy='createdAt'] - Field to sort by
 * @param {string} [options.direction='desc'] - Sort direction ('asc' or 'desc')
 * @returns {Promise<Task[]>} Array of all tasks
 * @throws {StorageError} If storage access fails
 */
async getAll(options = {})
```

**Preconditions**:
- Storage is initialized

**Postconditions**:
- Returns all tasks in storage
- Tasks are ordered according to options

**Performance**: O(n) - Target <100ms for 1000 tasks

**Example**:
```javascript
// Get all tasks, newest first (default)
const tasks = await storage.getAll();

// Get all tasks, oldest first
const tasks = await storage.getAll({ direction: 'asc' });
```

---

#### getById

```javascript
/**
 * Retrieves a single task by ID
 *
 * @param {number} taskId - Task ID
 * @returns {Promise<Task|null>} The task or null if not found
 * @throws {StorageError} If storage access fails
 */
async getById(taskId)
```

**Preconditions**:
- Storage is initialized
- `taskId` is a positive integer

**Postconditions**:
- Returns task if found, null otherwise
- No storage modification

**Performance**: O(1) - Target <5ms

**Example**:
```javascript
const task = await storage.getById(42);
if (task) {
  console.log('Found:', task);
} else {
  console.log('Task not found');
}
```

---

#### getByStatus

```javascript
/**
 * Retrieves tasks filtered by completion status
 *
 * @param {boolean} completed - Filter by completion status
 * @param {Object} [options] - Query options
 * @param {string} [options.orderBy='createdAt'] - Field to sort by
 * @param {string} [options.direction='desc'] - Sort direction
 * @returns {Promise<Task[]>} Array of matching tasks
 * @throws {StorageError} If storage access fails
 */
async getByStatus(completed, options = {})
```

**Preconditions**:
- Storage is initialized
- `completed` is a boolean

**Postconditions**:
- Returns tasks matching completion status
- Tasks are ordered according to options
- Uses `by_completed` index for efficiency

**Performance**: O(log n + m) where m = matches - Target <50ms for 1000 tasks

**Example**:
```javascript
// Get incomplete tasks
const activeTasks = await storage.getByStatus(false);

// Get completed tasks, oldest first
const doneTasks = await storage.getByStatus(true, { direction: 'asc' });
```

---

#### update

```javascript
/**
 * Updates an existing task
 *
 * @param {number} taskId - ID of task to update
 * @param {Object} updates - Fields to update (partial task object)
 * @returns {Promise<Task>} The updated task
 * @throws {StorageError} If storage update fails or task not found
 */
async update(taskId, updates)
```

**Preconditions**:
- Storage is initialized
- Task with `taskId` exists
- `updates` contains at least one field

**Postconditions**:
- Task is updated in storage with merged fields
- Original fields not in `updates` are preserved
- IndexedDB indexes updated if indexed fields changed

**Performance**: O(1) - Target <5ms

**Example**:
```javascript
// Update title only
const task = await storage.update(42, {
  title: "Buy groceries and milk",
  updatedAt: Date.now()
});

// Update completion only
const task = await storage.update(42, {
  completed: true,
  updatedAt: Date.now()
});
```

**Error Cases**:
```javascript
// Task not found
await storage.update(999, { title: "New" });
// Throws: StorageError("Task with ID 999 not found")
```

---

#### delete

```javascript
/**
 * Deletes a task permanently
 *
 * @param {number} taskId - ID of task to delete
 * @returns {Promise<boolean>} true if deleted, false if not found
 * @throws {StorageError} If storage deletion fails
 */
async delete(taskId)
```

**Preconditions**:
- Storage is initialized
- `taskId` is a positive integer

**Postconditions**:
- Task is permanently removed from storage if it existed
- Returns true if task was deleted, false if not found

**Performance**: O(1) - Target <5ms

**Example**:
```javascript
const deleted = await storage.delete(42);
if (deleted) {
  console.log('Task deleted');
} else {
  console.log('Task not found');
}
```

---

#### count

```javascript
/**
 * Returns the total number of tasks
 *
 * @returns {Promise<number>} Total task count
 * @throws {StorageError} If storage access fails
 */
async count()
```

**Preconditions**:
- Storage is initialized

**Postconditions**:
- Returns accurate count of tasks in storage

**Performance**: O(1) in IndexedDB, O(n) in LocalStorage - Target <10ms

**Example**:
```javascript
const total = await storage.count();
console.log(`You have ${total} tasks`);
```

---

#### clear

```javascript
/**
 * Deletes all tasks (dangerous operation)
 *
 * @returns {Promise<void>}
 * @throws {StorageError} If storage clear fails
 */
async clear()
```

**Preconditions**:
- Storage is initialized

**Postconditions**:
- All tasks are permanently deleted
- Storage is empty

**Performance**: O(1) - Target <10ms

**Warning**: This operation cannot be undone

**Example**:
```javascript
await storage.clear();
// All tasks are deleted
```

---

#### close

```javascript
/**
 * Closes the storage connection
 * For IndexedDB: closes database connection
 * For LocalStorage: no-op
 *
 * @returns {Promise<void>}
 */
async close()
```

**Preconditions**:
- Storage is initialized

**Postconditions**:
- IndexedDB connection closed
- No further operations allowed until re-initialized

**Example**:
```javascript
await storage.close();
// Storage is closed
```

---

## Properties

### isIndexedDB

```javascript
/**
 * Indicates if storage is using IndexedDB (true) or LocalStorage (false)
 * @type {boolean}
 * @readonly
 */
get isIndexedDB()
```

**Example**:
```javascript
console.log(storage.isIndexedDB ? 'Using IndexedDB' : 'Using LocalStorage');
```

---

## Storage Backend: IndexedDB

### Database Schema

**Database Name**: `TodoAppDB`
**Version**: 1

**Object Store**: `tasks`
- Key path: `id` (auto-increment)
- Indexes:
  - `by_completed`: keyPath `completed`, not unique
  - `by_created`: keyPath `createdAt`, not unique

### Implementation Notes

```javascript
// Database setup
const request = indexedDB.open('TodoAppDB', 1);

request.onupgradeneeded = (event) => {
  const db = event.target.result;

  // Create object store
  const store = db.createObjectStore('tasks', {
    keyPath: 'id',
    autoIncrement: true
  });

  // Create indexes
  store.createIndex('by_completed', 'completed', { unique: false });
  store.createIndex('by_created', 'createdAt', { unique: false });
};
```

---

## Storage Backend: LocalStorage

### Storage Key

**Key**: `todoapp_tasks`

### Value Format

JSON-serialized array of tasks:
```json
[
  { "id": 1, "title": "Task 1", "completed": false, ... },
  { "id": 2, "title": "Task 2", "completed": true, ... }
]
```

### ID Generation

Next ID = `Math.max(...tasks.map(t => t.id), 0) + 1`

### Performance Characteristics

- **Read**: O(1) key lookup + O(n) JSON.parse
- **Write**: O(n) JSON.stringify + O(1) key write
- **Filter**: O(n) array iteration (no indexes)

---

## Error Handling

### StorageError

```javascript
class StorageError extends Error {
  constructor(message, cause) {
    super(message);
    this.name = 'StorageError';
    this.cause = cause; // Original error if available
  }
}
```

### Common Error Scenarios

| Scenario | Error Message | Cause |
|----------|---------------|-------|
| **Quota exceeded** | "Storage quota exceeded" | Storage limit reached |
| **IndexedDB blocked** | "IndexedDB is not available" | User blocked in browser |
| **Corrupted data** | "Failed to parse stored data" | Invalid JSON in LocalStorage |
| **Connection failed** | "Failed to open database" | IndexedDB initialization error |
| **Task not found** | "Task with ID {id} not found" | Update/delete non-existent task |

---

## Performance Requirements

Based on requirements in spec.md and research.md:

| Operation | Target | Actual (IndexedDB) | Actual (LocalStorage) |
|-----------|--------|-------------------|----------------------|
| **init** | <100ms | 10-50ms | <1ms |
| **create** | <5ms | 1-5ms | 10-50ms (rewrites all) |
| **getAll (1000)** | <100ms | 10-50ms | 20-100ms |
| **getByStatus** | <50ms | 5-20ms | 20-100ms (no index) |
| **update** | <5ms | 1-5ms | 10-50ms (rewrites all) |
| **delete** | <5ms | 1-5ms | 10-50ms (rewrites all) |

---

## Usage Example

```javascript
// Initialize storage
const storage = new TaskStorage();
await storage.init();

console.log('Using IndexedDB:', storage.isIndexedDB);

// Create tasks
const task1 = await storage.create({
  title: "Buy groceries",
  completed: false,
  createdAt: Date.now(),
  updatedAt: Date.now()
});

// Get all tasks
const allTasks = await storage.getAll();
console.log(`Total tasks: ${allTasks.length}`);

// Get by ID
const task = await storage.getById(task1.id);

// Filter by status
const activeTasks = await storage.getByStatus(false);
const completedTasks = await storage.getByStatus(true);

// Update task
const updated = await storage.update(task1.id, {
  completed: true,
  updatedAt: Date.now()
});

// Count tasks
const count = await storage.count();

// Delete task
const deleted = await storage.delete(task1.id);

// Close connection
await storage.close();
```

---

## Testing Contract

### Unit Tests Required

1. **Initialization**:
   - IndexedDB initialization success
   - LocalStorage fallback when IndexedDB unavailable
   - Error handling for blocked storage

2. **CRUD Operations**:
   - Create task generates unique ID
   - Get by ID returns correct task
   - Get all returns all tasks in correct order
   - Get by status filters correctly
   - Update modifies task
   - Delete removes task

3. **Edge Cases**:
   - Update non-existent task
   - Delete non-existent task
   - Empty storage queries
   - Quota exceeded handling

4. **Performance**:
   - 1000 tasks insert < 5 seconds
   - 1000 tasks retrieve < 100ms
   - Filter 1000 tasks < 50ms

### Integration Tests Required

1. Real IndexedDB operations (not mocked)
2. Real LocalStorage operations
3. Fallback mechanism (IndexedDB → LocalStorage)
4. Concurrent operations handling

---

**Contract version**: 1.0
**Last updated**: 2025-10-24
**Stability**: Stable (ready for implementation)
