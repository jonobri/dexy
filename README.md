# dexy ⚡

**Keep your Macbook wide awake. For agentic workflows, on the go.**

`dexy` is a one-word toggle for macOS's built-in "never sleep — even with the lid shut" mode. 

Flip it on before you close the laptop, to keep your agents running, a download progressing, or whatever else that helps you remain productive always. Just add an internet connection via your hotspot, and you can keep your Macbook in action all day. 

Functionally, this is just a cute little wrapper around `pmset`'s `SleepDisabled` flag. But it's easier to type the name of an ADHD medication than to remember the full `pmset` command every time.

```console
$ dexy
● dexy: ON  — wide awake (battery sleep 0, lid-closed sleep disabled)

$ dexy status
● dexy: ON  — wide awake (battery sleep 0, lid-closed sleep disabled)

$ dexy off
○ dexy: OFF — normal sleep (battery sleep 5m, lid-closed enabled)
```

## Install

### Homebrew (recommended)

```sh
brew install jonobri/tap/dexy
```

### From source

```sh
git clone https://github.com/jonobri/dexy
ln -s "$PWD/dexy/dexy" /usr/local/bin/dexy   # or anywhere on your PATH
```

## Usage

```
dexy            toggle on/off
dexy on         stay awake — disable sleep, even with the lid shut
dexy off        restore normal sleep
dexy status     show the current state
dexy -h         help
dexy -v         version
```

Toggling sleep is a privileged operation, so `on`/`off`/`toggle` run `pmset`
under `sudo` and may ask for your password. `status` never needs sudo.

### Tips

- **Restore a different sleep timeout.** By default `dexy off` sets battery
  sleep back to 5 minutes. Change it:
  ```sh
  export DEXY_SLEEP_MINUTES=10
  ```
- **Scripting.** `dexy status` prints a stable `dexy: ON` / `dexy: OFF` line you
  can grep. Set `NO_COLOR=1` to strip ANSI colours.

## How it works

macOS gates lid-closed sleep behind the `SleepDisabled` power-management flag.
`dexy on` runs:

```sh
sudo pmset -b sleep 0          # don't idle-sleep on battery
sudo pmset -b disablesleep 1   # and don't sleep when the lid closes
```

`dexy off` reverses both. That's the whole trick — no daemon, no menu-bar app,
nothing running in the background. When dexy is on, it's on because the system
flag is set, and `dexy status` reads that same flag back, so the two can never
drift out of sync.

> **Heads up:** while dexy is on, your Mac will *not* sleep on battery with the
> lid shut. Great for a long unattended job; less great if you toss it in a bag
> and forget. `dexy off` (or a reboot) clears it.

## Why "dexy"?

A little something to keep your Mac up past its bedtime. Use responsibly. ☕

## License

[MIT](LICENSE) © Jonathan O'Brien
