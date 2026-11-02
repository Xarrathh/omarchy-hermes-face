# Security notes

This repository is a bar widget plus the copy of the Hermes face it runs. It executes as your own
user, unsandboxed, like every other community plugin. This document states exactly what it touches
so the marketplace's automated baseline and a human reviewer do not have to infer it.

## What runs, and as whom

| Process | Started by | Runs as |
| --- | --- | --- |
| `bin/*.sh` helpers | the bar widget (clicks), the theme hook, the stop key | you |
| `face/server.py` | `systemd --user` unit `hermes-face.service` (or a detached process where systemd is unavailable) | you |
| `hermes chat` | `face/server.py`, one child per spoken turn | you |

No `sudo`, no `pkexec`, no root, no setuid, and no passwordless privilege rules. Nothing is
installed outside the user's home.

## Network

- The bridge binds `127.0.0.1:9120` only. It cannot be reached from the LAN or the internet.
- Scripts, the widget and the hook talk to that loopback port and nowhere else.
- **The face's speech is not local.** `edge-tts` synthesises through Microsoft's Edge read-aloud
  service (`wss://speech.platform.bing.com`). The text it speaks — the agent's reply — is sent
  there. No audio, no microphone input and no history is uploaded; only the sentence being spoken.
  If you do not want that, remove `edge-tts` (or leave the `venv` absent): the face then runs and
  displays silently, which it reports on screen.
- The reply itself is produced by your own Hermes agent, whose model traffic is governed by your
  Hermes configuration, not by this plugin.
- `pip install edge-tts` fetches that one package from PyPI the first time voice is needed. Nothing
  else is downloaded, and no code is fetched and executed at runtime.

## Local authorisation

The bridge is on loopback, which means "this machine", not "this user". Private endpoints are
therefore gated on the kernel's answer about the *client socket*: its local endpoint, the endpoint
it connected to, and the owning uid, resolved from `/proc/net/tcp{,6}`. A connection from another
local account is refused. `/health` and `/face/*` are deliberately open, and contain no user data.

`POST` additionally requires `content-type: application/json` and, when present, a same-origin
`Origin`/`Referer`, so a web page in any browser cannot drive the bridge.

A shared token remains as a fallback for systems where `/proc` is unreadable to the service; it is
never passed on a command line or in a URL by these scripts.

## Files

`~/.local/share/hermes-face/` is mode `0700`; `config.json`, `history.json`, the token file and
every audio and prompt file are `0600`. `harden_permissions()` re-applies this at each start, so an
install created under a looser umask is corrected rather than left readable.

Prompt text and reply text are never placed in process arguments: the CLI speech fallback writes to
a private file and passes `--file`, and the agent is invoked with `--query-file`. A Hermes too old
to support that is refused with an explanation rather than having the spoken words exposed in `ps`.

## What the automated baseline will flag

Declared in advance, so these are expected rather than surprising:

- `package-manager` — `bin/face-ensure.sh` runs `pip install edge-tts` into a private venv when no
  working copy exists.
- `service-management` — `bin/face-ensure.sh` and `bin/face-restart.sh` use
  `systemctl --user daemon-reload|enable|restart` on the user's own unit.
- `installer` — the vendored `face/` tree is copied into the user's data directory on first use.

There are no findings-pattern matches: no `curl | sh`, no unpinned remote git execution, no
privileged-process control from shared temporary files, and no privilege-escalation rules of any
kind. All temporary state is created with `O_CREAT` mode `0600` inside the owner's home; nothing
uses `/tmp`.

## Removing it

`omarchy plugin remove pierregoosen.hermes-face` removes the widget. The bridge and its data are
separate and stay until you delete them: `systemctl --user disable --now hermes-face` and
`rm -rf ~/.local/share/hermes-face ~/.config/systemd/user/hermes-face.service`.

## Reporting

Open an issue for ordinary bugs. For anything that looks like a vulnerability, use GitHub's private
vulnerability reporting on this repository rather than a public issue.
