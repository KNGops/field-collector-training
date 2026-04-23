# KNG Field Collector Training

Static GitHub Pages site for KNG field collector onboarding, manager code generation, and operations administration. No backend, no build step — all three files are self-contained HTML.

---

## Live URLs

| Page | URL | Audience |
|---|---|---|
| Training | `https://kngops.github.io/field-collector-training/` | Collectors (public) |
| Admin dashboard | `https://kngops.github.io/field-collector-training/admin.html#kng-admin-2025` | Managers only |
| Code generator | `https://kngops.github.io/field-collector-training/codegen.html#kng-admin-2025` | Managers only |

> **Important:** The `#kng-admin-2025` fragment is part of the access secret. Share the full URL only via internal channels (Slack, email). Never link to it from the training page or any public document.

---

## Repo structure

```
field-collector-training/
├── index.html      ← Collector training & code entry (public)
├── admin.html      ← Operations dashboard (manager-only)
├── codegen.html    ← Roster & hash code generator (manager-only)
└── README.md
```

---

## Access control

The manager pages use a two-layer gate that works entirely in the browser — no server required.

### Layer 1 — URL fragment secret
- Both `admin.html` and `codegen.html` check `window.location.hash` on load
- If the fragment doesn't match the required secret, the page shows a generic "Access Denied" screen with no hint of what's expected
- URL fragments are never sent to servers and never appear in server logs or search engine indexes
- **To rotate the secret:** update `REQUIRED_HASH` in both files, commit, then distribute the new bookmark URL to managers

### Layer 2 — PIN
- After the hash check passes, a PIN prompt appears
- The PIN is stored in `localStorage` on the manager's device
- Both pages share the same PIN key (`kng_admin_pin`), so setting it once in Admin covers both
- **Default PIN on a fresh device:** `kng-admin-2025` — managers should change this on first login via Admin → Settings → Change PIN
- **To rotate the PIN:** log in to the admin dashboard and change it in Settings (no redeploy needed)

### What collectors see
Collectors only ever visit `index.html`. The manager page filenames are not linked anywhere in the training flow and won't be discovered by browsing or search engines.

---

## Deploying updates

### Option A — GitHub web UI (simplest)
1. Open the repo on GitHub
2. Click a file → pencil icon to edit, or **Add file → Upload files** for new files
3. Paste/upload and commit — Pages rebuilds in ~60 seconds

### Option B — Git CLI
```bash
git clone https://github.com/kngops/field-collector-training.git
cd field-collector-training

# Copy updated files (rename as needed)
cp ~/Downloads/admin.html admin.html
cp ~/Downloads/codegen.html codegen.html
cp ~/Downloads/index.html index.html

git add .
git commit -m "describe what changed"
git push origin main
```

---

## Typical manager workflow

1. **Generate codes** — open `codegen.html#kng-admin-2025`, enter region prefix and collector count, click Generate
2. **Export roster** — download the CSV (contains plain-text codes, names, IDs, emails, and code hashes)
3. **Update training page** — copy the `VALID_CODE_HASHES` snippet from the code generator output, paste it into `index.html` replacing the existing array, commit and push
4. **Distribute codes** — send each collector their individual plain-text code from the roster CSV (not the hash)
5. **Monitor submissions** — use `admin.html#kng-admin-2025` to import collector CSV exports and review stats

---

## Data & privacy

- All processing is 100% client-side — no data is ever sent to a server
- The training page stores only hashed codes (`SHA-256`), never plain-text codes
- The admin dashboard stores hashes and the PIN in `localStorage` on the manager's device only
- To fully clear local admin state, use Admin → Settings → Wipe All Local Storage

---

## Changing the secret hash

If the manager URL needs to be rotated (e.g., someone who knew the old URL has left):

1. Open `admin.html` in a text editor
2. Find `const REQUIRED_HASH = 'kng-admin-2025';` and change the value
3. Repeat in `codegen.html`
4. Commit and push both files
5. Share the new bookmark URLs with current managers

No PIN change is required unless the PIN was also compromised.
