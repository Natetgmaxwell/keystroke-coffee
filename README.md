# Coffee

Keep the screen and the computer awake from Keystroke, for a while or on a
recurring schedule.

```text
coffee                    open Coffee
coffee on                 caffeinate until you turn it off
coffee off                decaffeinate
coffee 30m                caffeinate for 30 minutes
coffee 1h30m              caffeinate for 90 minutes
coffee 5pm                caffeinate until 17:00 (tomorrow if it is already past)
coffee until tomorrow 8am caffeinate until tomorrow morning
coffee while Firefox      caffeinate until Firefox exits
coffee schedule add Mon,Thu 09:00-14:00
coffee status
```

The declared commands `caffeinate`, `decaffeinate`, `coffee-for`,
`coffee-until`, `coffee-while`, `coffee-status` and `coffee-schedule` are
shortcuts for the same actions.

## Install

Clone this repository into Keystroke's extension directory, then enable
**Coffee** in **Keystroke Settings → Extensions** and restart the shell:

```sh
mkdir -p ~/.local/share/keystroke/extensions
git clone https://github.com/Natetgmaxwell/keystroke-coffee.git \
  ~/.local/share/keystroke/extensions/coffee
```

Remove or rename an existing `coffee` directory first if you already have a
local copy.

## What it actually does

Two independent mechanisms are used:

- **System sleep** is blocked by a child process:
  `systemd-inhibit --what=idle:sleep:handle-lid-switch --who=Keystroke Coffee
  --why "Coffee keeps the computer awake" --mode=block sleep infinity`.
  Stopping Coffee sends SIGTERM to `systemd-inhibit`; it removes the inhibitor
  and kills its `sleep` child.
- **Omarchy's idle clock** (screen lock) is paused with
  `omarchy-shell idle disable`. Omarchy represents that as the stay-awake file
  `~/.local/state/omarchy/indicators/stay-awake`. Coffee writes its own marker,
  `~/.local/state/keystroke/coffee/idle-owned`, before changing that file and
  only calls `omarchy-shell idle enable` when its marker exists. A stay-awake
  the user set themselves is therefore never cleared by Coffee.

`bin/coffee idle-on` and `bin/coffee idle-off` own that marker logic.
`bin/coffee watch APP` polls for a process or Hyprland window and exits when it
disappears.

## Schedules

```text
coffee schedule add Mon,Thu 09:00-14:00
coffee schedule add weekdays 9am-5pm
coffee schedule add every day except Tuesday from 13:00 to 20:00
coffee schedule list
coffee schedule pause 14-0900-1400
coffee schedule resume 14-0900-1400
coffee schedule delete 14-0900-1400
```

A schedule has days (`0` Sunday through `6` Saturday), a start time and an end
time. With no day words the schedule runs every day. A window must end later
than it starts on the same day; windows that cross midnight are not supported.
Schedules are checked every 30 seconds and are stored in
`~/.local/state/keystroke/coffee/state.json`.

If a schedule is running and you decaffeinate, Coffee asks you to pause the
schedule first. Turn on **Decaffeinate pauses a schedule** to make the same
action pause the running schedule instead.

## Settings

- **Pause Omarchy's idle clock** (default on): stop the screen from locking
  while caffeinated.
- **Block system sleep** (default on): hold the systemd inhibitor, including
  lid close.
- **Show Coffee in the bar** (default on): a coffee cup and a countdown next to
  the menu button.
- **Decaffeinate pauses a schedule** (default off): see above.

With both prevention settings off, Coffee refuses to start and says so in the
palette.

## State and processes

- `~/.local/state/keystroke/coffee/state.json`: schedules only. Active
  caffeination is deliberately in memory, so restarting `omarchy-shell` stops
  an active Coffee rather than leaving an orphaned inhibitor behind.
- `~/.local/state/keystroke/coffee/idle-owned`: marker saying Coffee changed
  Omarchy's stay-awake file. It is removed by `idle-off` and swept at startup.
- `systemd-inhibit ... sleep infinity`: the sleep inhibitor, a child of the
  Keystroke service.
- `bin/coffee watch APP`: the process/window watcher for `coffee while`.
- `bin/coffee idle-on|idle-off`: short-lived helpers for the idle marker.

On load, `bin/coffee idle-off` clears a marker left by a crash. On unload,
Coffee stops its inhibitor and watcher and asks the helper to restore the idle
clock.

## Dependencies and limits

- Linux with `systemd-inhibit` and `sleep`.
- Omarchy's `omarchy-shell` for the idle clock.
- `pgrep`, `ps` and `grep` for `coffee while`; `hyprctl` and `jq` are used
  when available to match window class and title as well.
- No network access, no `sudo`, and no files of its own outside
  `~/.local/state/keystroke/coffee/`; it only changes Omarchy's stay-awake
  indicator as described above.
- `coffee while` is a name match, not a PID: a short or common name such as
  `bash` will keep Coffee on as long as any matching process exists.
- A schedule only starts while the extension is loaded and Keystroke is
  running. Active caffeination does not survive a shell restart.

## Development

The extension lives at `~/.local/share/keystroke/extensions/coffee`. Enable it
in **Keystroke Settings → Extensions**, then restart the shell after editing
code that was already loaded. Run the checks with:

```sh
~/.config/omarchy/plugins/evindor.keystroke/bin/keystroke check-extensions \
  ~/.local/share/keystroke/extensions/coffee
```
