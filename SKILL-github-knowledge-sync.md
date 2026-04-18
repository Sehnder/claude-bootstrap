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

Note: Direct GitHub API calls via bash are blocked in the Cowork sandbox
(no outbound network). However, JavaScript fetch() within the browser DOES work
with the PAT — this is the preferred push mechanism for bulk operations.
For single-file updates, the browser web editor also works (see Push Pattern).

---

## Folder Structure

```
claude-knowledge/
├── projects/
│   ├── general/                   ← cross-project reusable assets
│   │   ├── skills/                ← this file lives here
│   │   └── prompts/
│   └── oracle-ai-studio/          ← project-specific assets
│       ├── skills/
│       └── prompts/
└── session-logs/                  ← session closeout logs by project
```

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

Run this pattern in the browser via mcp__Claude_in_Chrome__javascript_tool:

```javascript
const PAT = /* read from Cowork Basics/.secrets/github_pat.md */;
const BASE = 'https://api.github.com/repos/Sehnder/claude-knowledge/contents';
const H = { 'Authorization': `Bearer ${PAT}`, 'Content-Type': 'application/json' };

(async () => {
  // Read current file (get content + sha)
  const r = await fetch(`${BASE}/[path]`, { headers: H });
  const d = await r.json();
  let content = decodeURIComponent(escape(atob(d.content.replace(/\n/g, ''))));

  // Modify content as needed
  content = content.replace('[old text]', '[new text]');

  // Write back
  await fetch(`${BASE}/[path]`, {
    method: 'PUT', headers: H,
    body: JSON.stringify({
      message: 'update: [description]',
      content: btoa(unescape(encodeURIComponent(content))),
      sha: d.sha
    })
  });
})();
```

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
- base64 encoding for direct API calls is blocked by sandbox network restrictions —
  always use browser automation for pushes
