> **Note:** This is the public bootstrap mirror of this file.
> Canonical copy: `projects/general/skills/SKILL-github-knowledge-sync.md` in the private `claude-knowledge` repo.
> If these differ, the private copy is authoritative.

# GitHub Knowledge Sync — SKILL.md

## About this file
Reusable skill for reading from and writing to the claude-knowledge GitHub
repository from within any Cowork session. Load this file whenever a session
needs to push findings back to a skill file.

Last updated: 2026-04-18

---

## Repository

- **Repo:** https://github.com/Sehnder/claude-knowledge (private)
- **Owner:** Sehnder
- **Default branch:** main

---

## Credentials

PAT is stored at:
`.secrets/github_pat.md`

Read it at the start of any push operation. The PAT is scoped to this repo
only with Contents: Read and Write permissions.

Note: Direct GitHub API calls via bash/curl work in the Cowork sandbox **if**
`api.github.com` is in your egress allowlist (Claude Desktop → Settings →
Capabilities → Code execution → Additional allowed domains). If not configured,
use JavaScript fetch() via browser instead (see Push Pattern).

---

## Critical Rules

> **CRITICAL — File placement:** Skills and prompts MUST go in their respective subfolders, never directly in the project root.
>
> - Skills → `projects/{workspace}/skills/SKILL.md` (default) or `SKILL-{name}.md` (only when explicitly requested)
> - Prompts → `projects/{workspace}/prompts/{name}.md`
>
> Example for workspace "GitTest":
> - A prompt called "order pizza" → `projects/GitTest/prompts/order-pizza.md`
> - A skill called "bees" → `projects/GitTest/skills/SKILL.md` (appended by default; only create a separate file if explicitly instructed)

> **CRITICAL — PAT location:** The PAT file is at `.secrets/github_pat.md` **within the workspace folder** (e.g., `GitTest/.secrets/github_pat.md`). Read it using the `Read` tool before any push operation.

> **CRITICAL — Bash works:** Direct GitHub API calls via bash/curl DO work in the Cowork sandbox. Use `curl` with the PAT for all push operations — no browser automation required unless the PAT lacks repo access.

---

## Folder Structure

```
claude-knowledge/
├── projects/
│   ├── general/                   ← cross-project reusable assets (not tied to any one workspace)
│   │   ├── skills/
│   │   └── prompts/
│   └── {workspace-name}/          ← one folder per Cowork workspace/project
│       ├── skills/
│       └── prompts/
└── session-logs/                  ← session closeout logs by project
```

---

## Determining Your Project Folder

The `[project]` path segment in all URLs must match the **current Cowork
workspace name** — i.e., the name of the folder the user has selected/mounted.
For example, if the user's workspace is called "GitTest", all skills and
prompts for that session belong under:

```
projects/GitTest/skills/
projects/GitTest/prompts/
```

**Do NOT default to an existing project folder** (e.g., `oracle-ai-studio`)
unless the user's workspace name explicitly matches it.

If the project folder doesn't yet exist in `claude-knowledge`, just write the
first file — GitHub creates intermediate directories automatically when a file
is committed into them.

### Exception: General assets

If the user explicitly asks to save something to "general" (e.g., "add this
as a general skill" or "save this to the general prompts"), use
`projects/general/` instead of the workspace folder:

```
projects/general/skills/
projects/general/prompts/
```

### Default structure for a new project

```
projects/{workspace-name}/
├── skills/
│   └── SKILL.md        ← top-level cross-cutting skill for this project
└── prompts/            ← reusable prompts for this project
```

---

## File Naming Conventions

### Skills

**Default behavior:** Unless the user explicitly requests a separate named file, always add skill content to `SKILL.md`. Only create a separate `SKILL-{name}.md` file when explicitly instructed (e.g., "save this as a new skill file called bees").

```
SKILL.md              ← default target for all skills
SKILL-{name}.md       ← only if explicitly requested
```

Examples:
- User says "add a skill called bees" → append to `SKILL.md`
- User says "save a skill for agent teams" → append to `SKILL.md`
- User says "create a separate skill file called bees" → `SKILL-bees.md`
- User says "save this as SKILL.md" → `SKILL.md` (use exact name as given)

The top-level skill file for a project is always `SKILL.md` (no suffix).

### Prompts

Prompt files use a lowercase hyphenated name with a `.md` extension — no prefix:

```
{name}.md
```

Examples:
- User says "save a prompt called get file contents" → `get-file-contents.md`
- User says "save this prompt as bees" → `bees.md`

### Summary table

| User says... | Destination | Filename |
|---|---|---|
| "save a skill called bees" | `projects/{workspace}/skills/` | `SKILL.md` (appended by default) |
| "save a prompt called get file contents" | `projects/{workspace}/prompts/` | `get-file-contents.md` |
| "save a general skill called bees" | `projects/general/skills/` | `SKILL-bees.md` |
| "save a general prompt called bees" | `projects/general/prompts/` | `bees.md` |

---

## Pull Pattern (reading a skill file)

Fetch the raw URL directly in any session:

```
https://raw.githubusercontent.com/Sehnder/claude-knowledge/main/projects/[project]/skills/[filename].md
```

Example:
```
https://raw.githubusercontent.com/Sehnder/claude-knowledge/main/projects/oracle-ai-studio/skills/SKILL.md
```

---

## Push Pattern (updating a skill file)

### Method A — JavaScript fetch() via browser (preferred for bulk operations)

**Important:** The browser tab must be on a real web page before running JS —
navigate to https://github.com first if the tab is on `chrome://newtab`.

Run this pattern in the browser via mcp__Claude_in_Chrome__javascript_tool:

```javascript
(async () => {
  const PAT = /* read from .secrets/github_pat.md */;
  const BASE = 'https://api.github.com/repos/Sehnder/claude-knowledge/contents';
  const H = { 'Authorization': `Bearer ${PAT}`, 'Content-Type': 'application/json' };

  // Read current file (get content + sha) — omit this block for brand-new files
  const r = await fetch(`${BASE}/[path]`, { headers: H });
  const d = await r.json();
  let content = decodeURIComponent(escape(atob(d.content.replace(/\n/g, ''))));

  // Modify content as needed
  content = content.replace('[old text]', '[new text]');

  // Write back (include sha for existing files; omit sha for new files)
  await fetch(`${BASE}/[path]`, {
    method: 'PUT', headers: H,
    body: JSON.stringify({
      message: 'update: [description]',
      content: btoa(unescape(encodeURIComponent(content))),
      sha: d.sha   // omit this line when creating a brand-new file
    })
  });
})();
```

### Reading directory contents

To list and read all files in a folder, store results in `window._skillContents`
and then read each file **individually** in separate tool calls:

```javascript
// Step 1 — fetch and store all file contents
(async () => {
  const PAT = '...';
  const BASE = 'https://api.github.com/repos/Sehnder/claude-knowledge/contents';
  const H = { 'Authorization': `Bearer ${PAT}`, 'Accept': 'application/vnd.github+json' };

  const r = await fetch(`${BASE}/projects/[project]/skills`, { headers: H });
  const files = await r.json();

  const results = {};
  for (const file of files) {
    const fr = await fetch(file.url, { headers: H });
    const fd = await fr.json();
    results[file.name] = decodeURIComponent(escape(atob(fd.content.replace(/\n/g, ''))));
  }
  window._skillContents = results;
  return Object.keys(results).join(', ');
})();

// Step 2 — read each file in a separate tool call
window._skillContents['SKILL.md']
window._skillContents['SKILL-agent-teams.md']
// etc.
```

**Critical:** Never return all file contents in one combined string (e.g., via
`.map().join()`). The browser tool blocks output containing URL query parameters
like `?token=...`, which GitHub embeds in its API responses. This triggers:

> `[BLOCKED: Cookie/query string data]`

Always read `window._skillContents['filename.md']` one file at a time in
separate tool calls.

### Method B — GitHub web editor (simpler for single-file updates)

1. Navigate to the edit URL:
   `https://github.com/Sehnder/claude-knowledge/edit/main/projects/[project]/skills/[filename].md`

2. Inject updated content via the CodeMirror 6 API:
```javascript
const view = document.querySelector('.cm-content').cmTile.view;
const current = view.state.doc.toString();
const updated = current.replace('[target text]', '[replacement text]');
view.dispatch({ changes: { from: 0, to: view.state.doc.length, insert: updated } });
```

3. Click "Commit changes..." (top right, ~x:1505, y:61 — may need two clicks)

4. Set commit message via form_input on the textbox ref in the dialog

5. Click the "Commit changes" submit button in the dialog

---

## Commit Message Conventions

| Prefix | Use for |
|--------|---------|
| `init:` | First-time creation of a file |
| `update:` | Adding new findings to an existing skill |
| `cleanup:` | Removing test entries or stale content |
| `fix:` | Correcting wrong information |

---

## Known Gotchas

- The commit dialog does not always open on the first click of "Commit changes..." —
  clicking the button a second time reliably opens it
- The GitHub editor uses CodeMirror 6; access the view via `cmTile.view` on the
  `.cm-content` element (not `.CodeMirror` which is CM5)
- The browser tab must be on a real web page before running JS — `chrome://newtab`
  will throw a "Can't interact with browser internal pages" error
