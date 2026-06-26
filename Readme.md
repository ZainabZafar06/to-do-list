# A Simple To-Do App with localStorage: CRUD, Filtering, and Inline Editing

A to-do list app is a classic beginner project for a reason — it looks simple, but building one properly touches almost every core front-end skill: DOM manipulation, event handling, state management, and persistence. As the second deliverable in my Web Development internship at Arch Technologies, I built one from scratch in a single HTML file, with no frameworks and no backend — just the browser's own storage.

Here's how it works, and a few of the smaller details that I think actually mattered.

## The Data Model

Every task is a plain object with three properties: an id, the task text, and a `done` flag. The whole list lives in a single array, which is loaded from `localStorage` on page load and re-saved after every change:

```js
const STORAGE_KEY = 'todo-app-tasks';
let tasks = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');
let filter = 'all';

function save(){
  localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks));
}

function uid(){
  return Date.now().toString(36) + Math.random().toString(36).slice(2,7);
}
```

The `uid()` function combines the current timestamp with a short random string to generate IDs that are unique enough for a single-user, client-side app — no need for a UUID library here. Falling back to `'[]'` when nothing's in storage yet means a first-time visitor just sees an empty list instead of an error.

## CRUD, the Short Version

Add, delete, and toggle are the three operations a task can go through, and they all follow the same pattern: update the `tasks` array, persist it, then re-render:

```js
function addTask(text){
  const trimmed = text.trim();
  if (!trimmed) return;
  tasks.push({ id: uid(), text: trimmed, done: false });
  save();
  render();
}

function deleteTask(id){
  tasks = tasks.filter(t => t.id !== id);
  save();
  render();
}

function toggleTask(id){
  const t = tasks.find(t => t.id === id);
  if (t) { t.done = !t.done; save(); render(); }
}
```

Trimming and validating the input in `addTask` means you can't accidentally add a task that's just empty whitespace — a small thing, but it's the kind of edge case that's easy to forget until someone hits Enter on an empty field.

## Inline Editing Without a Separate "Edit Mode"

Instead of building a whole modal or separate edit form, each task's text is just rendered as a regular element with `contentEditable` turned on. Click it, and it becomes editable immediately:

```js
function editTask(id, newText){
  const t = tasks.find(t => t.id === id);
  if (t) {
    const trimmed = newText.trim();
    t.text = trimmed || t.text;
    save();
  }
}

// Wiring it up:
label.contentEditable = 'true';
label.addEventListener('blur', () => editTask(t.id, label.textContent));
label.addEventListener('keydown', (e) => {
  if (e.key === 'Enter') { e.preventDefault(); label.blur(); }
});
```

The edit is committed on `blur` — when the field loses focus, whatever's currently typed gets saved. Pressing Enter manually blurs the field (and prevents a stray line break from being inserted), so Enter acts as a quick "I'm done editing" shortcut without needing a separate save button.

## Filtering Without Touching the Underlying Data

The filter buttons (All / Active / Completed) don't ever modify the `tasks` array itself — they only change which subset gets rendered:

```js
function render(){
  let visible = tasks;
  if (filter === 'active') visible = tasks.filter(t => !t.done);
  if (filter === 'completed') visible = tasks.filter(t => t.done);
  // ...build the <li> elements for `visible` here
}

filterBtns.forEach(btn => {
  btn.addEventListener('click', () => {
    filterBtns.forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    filter = btn.dataset.filter;
    render();
  });
});
```

Keeping `filter` as a separate variable from `tasks` was a deliberate choice — it means the source of truth (the actual list of tasks) never gets mutated just because of how the user is currently viewing it. Switching filters back and forth never risks losing data.

## A Live Counter and an Empty State

The last piece is just user feedback: showing how many tasks are left, and showing a friendly message when there's nothing to display (either because the list is empty, or because a filter currently hides everything):

```js
emptyEl.style.display = visible.length === 0 ? 'block' : 'none';
const remaining = tasks.filter(t => !t.done).length;
counterEl.textContent = `${remaining} task${remaining === 1 ? '' : 's'} left · ${tasks.length} total`;
```

It's a small detail, but handling the singular/plural case (`task` vs `tasks`) is the kind of polish that's easy to skip and slightly annoying when you notice it's missing.

## What I Took Away From This

The interesting part of this project wasn't any single feature — it was keeping the in-memory array, `localStorage`, and the rendered DOM consistently in sync after every single action, without a framework managing that for me. It made me appreciate, in a very hands-on way, what tools like React's state management are actually solving under the hood.

**Live demo:** _add your hosted link here_
**Source code:** _add your GitHub repo link here_
