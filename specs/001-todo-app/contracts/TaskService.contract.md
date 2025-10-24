# TaskService Contract

**Feature**: 001-todo-app
**Date**: 2025-10-24
**Purpose**: Business logic layer for task management operations

## Overview

`TaskService` is the primary business logic layer that orchestrates task operations. It sits between the UI layer and the storage layer, handling validation, events, and coordinating CRUD operations.

## Interface

### Constructor

```javascript
/**
 * Creates a new TaskService instance
 * @param {TaskStorage} storage - Storage implementation for persisting tasks
 * @throws {Error} If storage is not provided or invalid
 */
constructor(storage)
```

**Preconditions**:
- `storage` must implement `TaskStorage` interface
- `storage.init()` should be called before service usage

**Example**:
```javascript
const storage = new TaskStorage();
await storage.init();
const service = new TaskService(storage);
```

---

### Methods

#### createTask

```javascript
/**
 * Creates a new task with the given title
 *
 * @param {string} title - The task title (1-500 characters)
 * @returns {Promise<Task>} The created task with generated ID
 * @throws {ValidationError} If title is empty or exceeds 500 characters
 * @throws {StorageError} If storage quota is exceeded
 *
 * @fires TaskService#taskCreated
 */
async createTask(title)
```

**Functional Requirement**: FR-001, FR-008

**Preconditions**:
- Service is initialized
- `title` is a non-null string

**Postconditions**:
- Task is persisted in storage
- `taskCreated` event is dispatched
- Task has unique ID, timestamps, and `completed: false`

**Validation**:
- Title after trim must be 1-500 characters (VR-001, VR-002)
- Throws `ValidationError` if validation fails

**Example**:
```javascript
const task = await service.createTask("Buy groceries");
// Returns: { id: 42, title: "Buy groceries", completed: false, ... }
```

**Error Cases**:
```javascript
// Empty title
await service.createTask("   ");
// Throws: ValidationError("Task title cannot be empty")

// Too long
await service.createTask("x".repeat(501));
// Throws: ValidationError("Task title cannot exceed 500 characters")
```

---

#### getAllTasks

```javascript
/**
 * Retrieves all tasks ordered by creation date (newest first)
 *
 * @returns {Promise<Task[]>} Array of all tasks
 * @throws {StorageError} If storage access fails
 */
async getAllTasks()
```

**Functional Requirement**: FR-002

**Preconditions**:
- Service is initialized

**Postconditions**:
- Returns all tasks from storage
- Tasks are ordered by `createdAt` descending (newest first)

**Performance**:
- Must complete in <100ms for 1000 tasks

**Example**:
```javascript
const tasks = await service.getAllTasks();
// Returns: [{ id: 100, ... }, { id: 99, ... }, { id: 98, ... }]
```

---

#### getTasksByStatus

```javascript
/**
 * Retrieves tasks filtered by completion status
 *
 * @param {boolean} completed - Filter by completion status
 * @returns {Promise<Task[]>} Array of matching tasks
 * @throws {StorageError} If storage access fails
 */
async getTasksByStatus(completed)
```

**Functional Requirement**: FR-009

**Preconditions**:
- Service is initialized
- `completed` is a boolean

**Postconditions**:
- Returns tasks matching the specified status
- Tasks are ordered by `createdAt` descending

**Performance**:
- Must use indexed query for O(log n) performance
- Must complete in <50ms for 1000 tasks

**Example**:
```javascript
// Get incomplete tasks
const activeTasks = await service.getTasksByStatus(false);

// Get completed tasks
const doneTasks = await service.getTasksByStatus(true);
```

---

#### updateTask

```javascript
/**
 * Updates an existing task's title and/or completion status
 *
 * @param {number} taskId - ID of task to update
 * @param {Object} updates - Fields to update
 * @param {string} [updates.title] - New title (1-500 characters)
 * @param {boolean} [updates.completed] - New completion status
 * @returns {Promise<Task>} The updated task
 * @throws {ValidationError} If title is invalid
 * @throws {NotFoundError} If task doesn't exist
 * @throws {StorageError} If storage update fails
 *
 * @fires TaskService#taskUpdated
 */
async updateTask(taskId, updates)
```

**Functional Requirement**: FR-006, FR-003

**Preconditions**:
- Service is initialized
- Task with `taskId` exists
- If `updates.title` provided, must be 1-500 characters

**Postconditions**:
- Task is updated in storage
- `updatedAt` timestamp is set to current time
- `taskUpdated` event is dispatched

**Validation**:
- If `title` provided, apply VR-001 and VR-002
- Throws `NotFoundError` if task doesn't exist

**Example**:
```javascript
// Update title only
const task = await service.updateTask(42, {
  title: "Buy groceries and milk"
});

// Update completion status only
const task = await service.updateTask(42, {
  completed: true
});

// Update both
const task = await service.updateTask(42, {
  title: "Buy groceries and milk",
  completed: true
});
```

**Error Cases**:
```javascript
// Task not found
await service.updateTask(999, { title: "New title" });
// Throws: NotFoundError("Task with ID 999 not found")

// Invalid title
await service.updateTask(42, { title: "" });
// Throws: ValidationError("Task title cannot be empty")
```

---

#### markTaskComplete

```javascript
/**
 * Marks a task as complete
 *
 * @param {number} taskId - ID of task to mark complete
 * @returns {Promise<Task>} The updated task
 * @throws {NotFoundError} If task doesn't exist
 * @throws {StorageError} If storage update fails
 *
 * @fires TaskService#taskUpdated
 */
async markTaskComplete(taskId)
```

**Functional Requirement**: FR-003

**Preconditions**:
- Service is initialized
- Task with `taskId` exists

**Postconditions**:
- Task's `completed` field is `true`
- `updatedAt` timestamp is updated
- `taskUpdated` event is dispatched

**Example**:
```javascript
const task = await service.markTaskComplete(42);
// Returns: { id: 42, completed: true, updatedAt: [current time], ... }
```

---

#### markTaskIncomplete

```javascript
/**
 * Marks a task as incomplete
 *
 * @param {number} taskId - ID of task to mark incomplete
 * @returns {Promise<Task>} The updated task
 * @throws {NotFoundError} If task doesn't exist
 * @throws {StorageError} If storage update fails
 *
 * @fires TaskService#taskUpdated
 */
async markTaskIncomplete(taskId)
```

**Functional Requirement**: FR-003

**Preconditions**:
- Service is initialized
- Task with `taskId` exists

**Postconditions**:
- Task's `completed` field is `false`
- `updatedAt` timestamp is updated
- `taskUpdated` event is dispatched

**Example**:
```javascript
const task = await service.markTaskIncomplete(42);
// Returns: { id: 42, completed: false, updatedAt: [current time], ... }
```

---

#### deleteTask

```javascript
/**
 * Deletes a task permanently
 *
 * @param {number} taskId - ID of task to delete
 * @returns {Promise<void>}
 * @throws {NotFoundError} If task doesn't exist
 * @throws {StorageError} If storage deletion fails
 *
 * @fires TaskService#taskDeleted
 */
async deleteTask(taskId)
```

**Functional Requirement**: FR-007

**Preconditions**:
- Service is initialized
- Task with `taskId` exists

**Postconditions**:
- Task is permanently removed from storage
- `taskDeleted` event is dispatched with task ID

**Example**:
```javascript
await service.deleteTask(42);
// Task 42 is permanently deleted
```

**Error Cases**:
```javascript
// Task not found
await service.deleteTask(999);
// Throws: NotFoundError("Task with ID 999 not found")
```

---

## Events

`TaskService` extends `EventTarget` and dispatches the following events:

### taskCreated

```javascript
/**
 * Dispatched when a new task is created
 * @event TaskService#taskCreated
 * @type {CustomEvent}
 * @property {Task} detail - The created task
 */
```

**Example**:
```javascript
service.addEventListener('taskCreated', (event) => {
  console.log('New task:', event.detail);
});
```

### taskUpdated

```javascript
/**
 * Dispatched when a task is updated
 * @event TaskService#taskUpdated
 * @type {CustomEvent}
 * @property {Task} detail - The updated task
 */
```

### taskDeleted

```javascript
/**
 * Dispatched when a task is deleted
 * @event TaskService#taskDeleted
 * @type {CustomEvent}
 * @property {number} detail - ID of the deleted task
 */
```

---

## Error Types

### ValidationError

```javascript
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ValidationError';
  }
}
```

**Usage**: Thrown when input validation fails (empty title, too long, etc.)

### NotFoundError

```javascript
class NotFoundError extends Error {
  constructor(message) {
    super(message);
    this.name = 'NotFoundError';
  }
}
```

**Usage**: Thrown when attempting to operate on non-existent task

### StorageError

```javascript
class StorageError extends Error {
  constructor(message, cause) {
    super(message);
    this.name = 'StorageError';
    this.cause = cause;
  }
}
```

**Usage**: Thrown when storage operations fail (quota exceeded, corrupted data, etc.)

---

## Performance Requirements

Based on Success Criteria from spec.md:

| Operation | Requirement | How Measured |
|-----------|-------------|--------------|
| **createTask** | <5 seconds total (including UI) | From user input to task visible |
| **getAllTasks** | <100ms for 1000 tasks | Time from call to Promise resolve |
| **getTasksByStatus** | <50ms for 1000 tasks | Time from call to Promise resolve |
| **updateTask** | <10ms | Time from call to Promise resolve |
| **deleteTask** | <10ms | Time from call to Promise resolve |

**Testing**: Use `performance.mark()` and `performance.measure()` to validate

---

## Usage Example

```javascript
// Initialize storage and service
const storage = new TaskStorage();
await storage.init();
const service = new TaskService(storage);

// Listen for events
service.addEventListener('taskCreated', (e) => {
  console.log('Task created:', e.detail);
});

service.addEventListener('taskUpdated', (e) => {
  console.log('Task updated:', e.detail);
});

// Create tasks
const task1 = await service.createTask("Buy groceries");
const task2 = await service.createTask("Call dentist");

// Get all tasks
const allTasks = await service.getAllTasks();
console.log('All tasks:', allTasks);

// Mark complete
await service.markTaskComplete(task1.id);

// Filter by status
const activeTasks = await service.getTasksByStatus(false);
const completedTasks = await service.getTasksByStatus(true);

// Update title
await service.updateTask(task2.id, {
  title: "Call dentist at 2pm"
});

// Delete task
await service.deleteTask(task1.id);
```

---

## Testing Contract

### Unit Tests Required

1. **Validation**:
   - Empty title rejection
   - Title >500 chars rejection
   - Valid title acceptance

2. **CRUD Operations**:
   - Create task with valid data
   - Get all tasks returns correct order
   - Filter by status returns correct subset
   - Update task modifies fields
   - Delete task removes from storage

3. **Error Handling**:
   - Update non-existent task throws NotFoundError
   - Delete non-existent task throws NotFoundError
   - Storage errors propagate correctly

4. **Events**:
   - taskCreated dispatched on create
   - taskUpdated dispatched on update/complete/incomplete
   - taskDeleted dispatched on delete

### Integration Tests Required

1. Service + Storage integration (real IndexedDB)
2. 1000-task performance validation
3. Concurrent operation handling

---

**Contract version**: 1.0
**Last updated**: 2025-10-24
**Stability**: Stable (ready for implementation)
