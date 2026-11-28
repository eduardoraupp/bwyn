# Story: Add a task from the document bar

- **Story**: 051-story-add-a-task-from
- **Created**: 2026-11-28
- **Status**: Draft

## Story
As a team member, I want to add a task to an open story from its bar, and to edit or delete any of its tasks from the task's own row, so that the story always lists the work a pair must do to deliver it, kept with the story without having to edit its text.

## Acceptance tests

**The Add task button**

- [ ] **Given** an open story, **Then** the bar shows "Add task" with a plus icon as its first button, with the hint "Add a task to this story".
- [ ] **Given** an open architectural enabler, driver or ADR, **Then** there is no Add task button.

**Adding a task**

- [ ] **Given** an open story, **When** the user presses Add task, **Then** a field labelled "New task", with the placeholder "Describe the task", opens below the task list, gets the focus, and is scrolled into view, with a "Save task" button (check) and a "Cancel" button (cross).
- [ ] **Given** a story with no task, **When** the user presses Add task, **Then** the section "Tasks 0" shows with the field.
- [ ] **Given** the field is empty or holds only spaces, **Then** "Save task" is disabled and Enter does nothing.
- [ ] **Given** the field holds a text, **When** the user presses Enter or "Save task", **Then** the task is added at the end of the list with the text trimmed, the count goes up by one, and the field closes.
- [ ] **Given** the field is open, **When** the user presses Escape or "Cancel", **Then** the field closes and nothing is added.
- [ ] **Given** the user pressed Add task and then the pencil of a task, **Then** the new-task field closes and only the edited row is a field.

**Each task has Edit and Delete**

- [ ] **Given** a story with tasks, **Then** the section shows "Tasks" and their count, and each task is a row with its text (long text wraps), an "Update task" button (pencil) and a "Remove task" button (bin). Their accessible names are "Update task: <text>" and "Remove task: <text>".
- [ ] **When** the user presses the pencil of a task, **Then** the row becomes a field labelled "Edit task" holding the text of the task, focused, with "Save task" and "Cancel". Only one task is edited at a time.
- [ ] **Given** the edited text is not empty, **When** the user presses Enter or "Save task", **Then** the row shows the new text, trimmed, in the same place in the list.
- [ ] **Given** the edited text is empty, **Then** "Save task" is disabled. Emptying a task does not delete it; the bin does.
- [ ] **When** the user presses Escape or "Cancel" while editing, **Then** the row shows its text as it was.
- [ ] **When** the user presses the bin of a task, **Then** the task is removed at once, without a confirmation, and the count goes down by one. If it was being edited, the edit ends.
- [ ] **Given** the last task is removed and no field is open, **Then** the Tasks section is hidden.

**A task is stored with the story at once**

- [ ] **Given** the user added, edited or removed a task, **Then** the story's tasks are stored in the project's database at once, without pressing Save, and no request is made to GitHub.
- [ ] **Given** the user added a task, **When** the page is reloaded or the story is closed and opened again, **Then** the task is there, with the same text and in the same order.
- [ ] **Given** the text of the story has unsaved changes, **When** the user adds a task, **Then** the task is stored and the text keeps its unsaved changes; Save still stores the text.
- [ ] **Given** the user added, edited or removed a task, **Then** the Markdown text of the story is unchanged.
- [ ] **Given** the story's tasks differ from what was last synced, **Then** the Sync button has its dot and the hint "Saved here, not on GitHub yet. Sync to push it.".

**The tasks of the text and the tasks of the list**

- [ ] **Given** a story created by the AI, **Then** its task list holds the tasks written under its `## Tasks`, in order (top-level items, bulleted or numbered, with or without a checkbox, up to the next heading or horizontal rule).
- [ ] **Given** the user saves the text and its `## Tasks` gained a task, **Then** that task is added at the end of the list. A task that is already in the list (same text, ignoring case and extra spaces) is not added twice.
- [ ] **Given** the user removed a task from the list, **When** the text is saved again without that line changing, **Then** the task does not come back.

**On GitHub**

- [ ] **Given** a story with tasks, **When** it is synced, **Then** its `meta.json` holds `"tasks": [{ "id": "...", "text": "..." }]` in the order of the list, and `story.md` is not changed by them.
- [ ] **Given** a story with no task, **Then** its `meta.json` has no `tasks` key.
- [ ] **Given** a `meta.json` pulled from GitHub, **Then** its tasks replace the ones stored locally; a task without an id or with an empty text is skipped.

## Business rules and constraints

- A task belongs to one story. It has an id (assigned on creation, stable across edits) and a text. No status, owner, or checkbox in this story.
- The order is the order of the list. A new task goes at the end; editing does not move a task.
- Tasks are stored on every change, like the links, diagrams, and UI example of the story. Only the text of the story waits for Save (Architecture: application layer / outbound adapter boundary).
- Tasks are not part of the Markdown. They live in the story's record in the database and in `meta.json` on GitHub.
- Only stories have tasks. Enablers, drivers, and ADRs do not (ADR-0007).
- The `PUT /stories/{id}/tasks` endpoint replaces the full ordered task list for a story; partial updates are not supported.
- Assumption: a task id is a UUID generated on the frontend at creation time and sent to the backend unchanged.
- No test may use real timers or introduce artificial delays; user-event runs with `delay: null` (Principle: UI/Front end Tests pay only for what they assert).

## Out of scope

- Ticking a task as done, assigning it to someone, or estimating it.
- Reordering tasks by drag and drop.
- Undo after deleting a task.
- Writing the tasks back into the `## Tasks` section of the story's Markdown text.
- A count of tasks on the Add task button.

## Tasks

- [ ] Add a `tasks` table (`id UUID`, `story_id`, `text`, `position`) to the database with a Flyway migration script in `bwyn-runtime` (ADR-0010).
- [ ] Define a `Task` value object and a `TaskRepository` port interface in `bwyn-core` (ADR-0017).
- [ ] Implement a `ReplaceStoryTasksCommand` and its handler in `bwyn-application` that replaces the full task list for a story (ADR-0017).
- [ ] Implement the `TaskRepository` outbound adapter using Spring Data JPA in `bwyn-infrastructure` (ADR-0011, ADR-0017).
- [ ] Add `PUT /stories/{id}/tasks` in `bwyn-rest`, annotated with OpenAPI `@Operation` and `@ApiResponse` (ADR-0014), and a `GET /stories/{id}/tasks` endpoint that returns the ordered list.
- [ ] Write a `TaskRepositoryIT` Testcontainers integration test verifying persist, replace, and load of tasks (ADR-0012).
- [ ] Add a `Task` entity model and a `TaskViewModel` in the frontend interface-adapters layer, with a use case that calls `PUT /stories/{id}/tasks` (ADR-0007, AE-0002).
- [ ] Register the `TaskViewModel` binding in the Inversify composition root (ADR-0006).
- [ ] Build the `TasksSection` React component in the `ui` layer: heading with count, task rows with pencil and bin, the shared task form (Enter, Escape, empty-disabled, trimmed text) (ADR-0007).
- [ ] Add the "Add task" button to `DocumentActionBar` for stories only, wired to open the new-task field in `TasksSection` (ADR-0007).
- [ ] Implement the `## Tasks` parser (use case layer) that extracts top-level bulleted or numbered items under `## Tasks` up to the next heading or horizontal rule, and merges them into the list on story load and on text save without duplicates and without restoring removed tasks.
- [ ] Implement the sync serialisation: write `tasks` into `meta.json` when the list is non-empty, omit the key when empty; read tasks back on pull, skipping entries without an id or with an empty text (ADR-0048 boundary).
- [ ] Write Jest tests for `TasksSection`: add, edit, cancel, remove, count, section hidden when empty, only one field open at a time, no artificial delays (Principle: UI/Front end Tests pay only for what they assert).
- [ ] Write Jest tests for the `## Tasks` parser: bulleted, numbered, with and without checkbox, stops at next heading, no duplicates, removed tasks do not return.
- [ ] Write a Jest test for the `meta.json` round-trip: serialise tasks, deserialise, skip invalid entries.
- [ ] Verify the full build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build).
