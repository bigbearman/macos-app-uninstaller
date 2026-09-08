# macos-app-uninstaller

A single-file CLI script that completely uninstalls a macOS app: it finds the
app's leftover files (Application Support, Caches, Preferences, Containers,
Group Containers, Saved Application State, Logs, LaunchAgents/Daemons...) by
matching app name and bundle ID, lists them with sizes, then moves the app
and all matched leftovers to the Trash after confirmation.

Same idea as AppDelete/AppCleaner, minimal CLI version, no GUI/Xcode build
required.

## Install

```bash
curl -o ~/.local/bin/uninstall-app \
  https://raw.githubusercontent.com/<your-github-username>/macos-app-uninstaller/main/uninstall-app
chmod +x ~/.local/bin/uninstall-app
```

Make sure `~/.local/bin` is in your `PATH`.

## Usage

```bash
uninstall-app <AppName>          # e.g. uninstall-app Figma
uninstall-app /path/to/App.app   # or pass a direct path
uninstall-app --test             # run the built-in self-check
```

The script prints the resolved app path, bundle ID, and every leftover file
it found, then asks `y/N` before touching anything. Deletion goes through
Finder (`osascript`), so removing a system-owned app (root-owned, under
`/Applications`) triggers macOS's normal admin authentication dialog instead
of requiring `sudo`.

## Limitations

- Leftover matching is name/bundle-ID substring search, one level deep in
  each known directory — good coverage for the common cases, but Group
  Containers sometimes use a team/group identifier that differs from the
  app's bundle ID, so a few files can be missed. Review the printed list
  before confirming.
- No dry-run flag; the confirmation prompt is the safety net.

## How it works

`scan_leftovers()` searches a fixed list of directories where macOS apps
typically store their data, using `find -iname "*name*" -o -iname "*bundleid*"`
one level deep. Everything found (plus the app bundle itself) is moved to
Trash via Finder, which also handles permission prompts for system apps.
