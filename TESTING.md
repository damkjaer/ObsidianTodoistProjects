# Testing the Plugin Safely

This guide explains how to test the updated plugin without touching your real Obsidian vault.

## Step 1: Build the plugin

In the repository root, run:

```bash
npm install
npm run build
```

This produces a fresh `main.js` in the project root. It also updates `manifest.json` (already present) and optionally `styles.css` (not used by this plugin).

## Step 2: Create a test vault

1. Open Obsidian.
2. Click **Open another vault** (the vault icon, bottom-left).
3. Click **Create new vault**.
4. Give it a name like `TodoistTest` and pick a safe location (e.g. your Desktop or a temp folder).
5. Click **Create**.

This vault is completely separate from your real notes.

## Step 3: Install the plugin into the test vault

The vault is stored at the path you chose in Step 2. Inside it:

```
TodoistTest/
└── .obsidian/
    └── plugins/
```

1. Create the plugin folder:
   ```
   TodoistTest/.obsidian/plugins/obsidian-todoist-projects/
   ```
2. Copy these three files from the repository into that folder:
   - `main.js`
   - `manifest.json`
   - `styles.css` (if it exists; otherwise skip it)

You can do this from the terminal:

```bash
VAULT="$HOME/Desktop/TodoistTest"   # adjust to where you created the vault
PLUGIN_DIR="$VAULT/.obsidian/plugins/obsidian-todoist-projects"
mkdir -p "$PLUGIN_DIR"
cp main.js manifest.json "$PLUGIN_DIR/"
```

## Step 4: Enable the plugin

1. In Obsidian (with the test vault open), go to **Settings → Community plugins**.
2. Turn off **Restricted mode** if prompted.
3. Find **Todoist Project Sync** in the list and enable it.
4. Go to **Settings → Todoist Project Sync** and enter your Todoist API token.
   - Get it from: Todoist → Settings → Integrations → Developer → API token.

## Step 5: Verify it works

1. Set the **Todoist project folder** to something like `Projects`.
2. Set the **sync frequency** to `10` (seconds) so you don't have to wait long.
3. Open the Obsidian Developer Console: press `Ctrl+Shift+I` (or `Cmd+Option+I` on Mac).
4. Wait for the sync interval to fire, or reload the plugin.

**Success looks like:**
- Console log: `Updating Todoist Project files` followed by `Todoist Project files updated`
- A `Projects/` folder appears in the test vault containing markdown files named after your Todoist projects.
- No `410` errors in the console.

**If you see errors:**
- `401 Unauthorized`: check your API token.
- `404 Not Found`: the endpoint path is wrong — check the console for the full URL.
- Any other errors: copy the full error from the console and open an issue.

## Step 6: Install into your real vault (once satisfied)

Repeat Step 3 for your real vault path, then reload the plugin there.

## Notes

- The plugin only **reads** from Todoist; it never writes back to Todoist.
- It only touches files/folders inside the configured **project folder** (default: `Projects`). Notes outside that folder are never modified.
- Archived/deleted projects are moved to `Projects/archive/` — they are not deleted.
- Your real vault is safe to leave running during testing in the test vault; the two are independent.
