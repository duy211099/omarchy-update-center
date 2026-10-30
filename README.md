# Update Center

An [Omarchy](https://omarchy.org) bar widget that shows every pending update in one place (system packages, AUR, mise tools, and Omarchy itself) and gently reminds you when you haven't updated in a while.

Omarchy's built-in update icon only appears when a new Omarchy release is out. Security fixes for everything else you've installed (your browser, for example) arrive through ordinary package updates that it doesn't mention. Update Center covers all of them and still updates through `omarchy-update`.

## What it shows

- **Bar icon:** always visible. Dim when you're up to date, full brightness with a count when updates are waiting, and in the theme's urgent colour once you're overdue. The count and the urgent colour can each be turned off.
- **Panel** (left-click):
  - your status, and when you last ran a full system upgrade
  - when the next reminder is due
  - pending updates per source, which you can expand to see each package's old → new version
  - when it last checked, and an **Update now** button that opens the same floating `omarchy-update` terminal as the Omarchy menu
- **Right-click:** check now.
- **Reminder:** a notification once your last full upgrade is older than 7 days *and* updates are waiting. It repeats daily until you update, and clicking it starts the update. Nothing is waiting? No reminder.

## Install

```bash
omarchy plugin add https://github.com/liambeck99/omarchy-update-center.git --enable
```

New plugins are added to the right of the bar. Omarchy's own update icon does the same job for Omarchy releases only, so you'll probably want to remove it:

```bash
omarchy plugin disable omarchy.system-update
```

To get it back, run `omarchy plugin enable omarchy.system-update`.

## Settings

Omarchy has no settings screen for plugins. Set options with `omarchy bar set`, using `--json` so numbers and true/false aren't saved as text:

```bash
omarchy bar set liambeck99.update-center reminderDays 5 --json
omarchy bar set liambeck99.update-center highlightOverdue false --json
```

They're stored on the widget's entry in `~/.config/omarchy/shell.json`, which you can also edit by hand, e.g. `{"id": "liambeck99.update-center", "reminderDays": 5}`.

| Setting | Default | Meaning |
|---|---|---|
| `reminderDays` | 7 | Remind once the last full upgrade is this many days old |
| `checkIntervalHours` | 6 | How often to check for pending updates in the background |
| `reminderRepeatHours` | 24 | While overdue, how long between reminders |
| `checkSystem` | true | Check official repository packages (`checkupdates`) |
| `checkAur` | true | Check AUR packages (`yay -Qua`) |
| `checkMise` | true | Check mise tools (`mise outdated`) |
| `checkOmarchy` | true | Show a new Omarchy release as its own line |
| `showCount` | true | Show the number of waiting updates next to the bar icon (off keeps the icon a fixed width) |
| `highlightOverdue` | true | Colour the bar icon with the theme's urgent colour when overdue (off keeps the bar's normal colour; the panel still says you're overdue) |

## How it works

Everything is read-only and needs no root:

- **System:** `checkupdates`, which syncs into a temporary database and never touches your real one.
- **AUR:** `yay -Qua`.
- **mise:** `mise outdated`, with the same release-age override `omarchy-update` uses, so counts match what an update would install.
- **Last updated:** the most recent `starting full system upgrade` line in `/var/log/pacman.log`. It counts upgrades from `omarchy-update` and from a hand-run `pacman -Syu`.
- **Update running:** once a minute it checks Omarchy's update lock and pacman's `db.lck`, and checks again when an update finishes.

Checks run 45 seconds after login, every `checkIntervalHours`, when you open the panel (at most every two minutes), and after updates. The time of the last reminder is kept in `~/.local/state/liambeck99.update-center/state.json`, so restarting the shell doesn't repeat a reminder.

A source that fails to check (for example, while you're offline) shows its error, keeps its previous results, and never triggers a reminder on its own.

From a script:

```bash
omarchy-shell liambeck99.update-center refresh   # check now
omarchy-shell liambeck99.update-center toggle    # open/close the panel
```

## Development

```bash
bun test                       # logic and status-script tests (stubbed tools, no network)
bin/update-center-status | jq  # what the panel sees
```

Saving files under `~/.config/omarchy/plugins/` hot-reloads the plugin, but the shell can keep the previously compiled QML and `Model.js`. If a change doesn't show up, run `omarchy-restart-shell`.

## License

MIT
