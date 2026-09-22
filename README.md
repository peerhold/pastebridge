## What is `pastebridge`?

**The error is on one screen. The answer is on another.**

`pastebridge` is a two-lane shared clipboard for debugging across two devices you own on the same network. Terminal output goes up the **log** lane, and commands come back down the **command** lane, one per entry, each with its own copy button.

It's one binary for Linux, macOS, Windows, or ARM. There is no account, no sign-in, and no server belonging to anyone else. Nothing you paste is written to disk, nothing is ever executed, and it makes no outbound requests.

## Contents

- [How It Works in 3 Steps](#how-it-works-in-3-steps)
- [The Loop It's For](#the-loop-its-for)
- [Core Features](#core-features)
- [Security Overview](#security-overview)
- [Quick Start](#quick-start)
- [Using pastebridge](#using-pastebridge)
- [Piping Output In](#piping-output-in)
- [Sharing with Another Device](#sharing-with-another-device)
- [Running Headless](#running-headless)
- [Quitting, and Quitting by Itself](#quitting-and-quitting-by-itself)
- [Options](#options)
- [What It Keeps, and Where](#what-it-keeps-and-where)
- [Troubleshooting](#troubleshooting)
- [What It Doesn't Do](#what-it-doesnt-do)
- [Privacy](#privacy)
- [Legal](#legal)

## How It Works in 3 Steps

1. **Run the binary:** Run the single file on the machine with the problem. Your browser opens at `http://127.0.0.1:8787/`, or use `-headless` where there is no browser at all.
2. **Switch sharing on:** A TLS listener starts on your local network and shows an address and a six-digit PIN. Sharing stays off until you ask for it, and off means no listener exists.
3. **Open it on the other device:** In the browser it already has. Enter the PIN once. Nothing to install.

## The Loop It's For

The copying is the problem this solves. If you have a terminal in front of you and SSH set up, use that instead.

1. **The commands are here. The help is over there.** You're on a phone, a console with no clipboard passthrough, or a machine whose only screen is on another desk. The command you need is long and full of quotes and braces, and the output you need to read is two hundred lines. Both move badly between screens.
2. **The output goes up the log lane.** Paste it, or pipe it straight in with `journalctl -n 100 | pastebridge` on a machine with no browser at all. It lands on the other device immediately, with its whitespace intact and its own copy button.
3. **You work the problem where the keyboard is.** Search it, read the docs, ask a colleague, or hand it to whatever model you use. That part happens in your client: `pastebridge` has no model, no API key, and no outbound connection, so it isn't in that conversation.
4. **The command comes back and copies with one tap.** One command per entry, because selecting text on a phone is a fight and retyping a long command is worse. Run it, pipe the result back, and go round again.

## Core Features

* **Two Lanes, One Feed:** Output and commands stay in separate columns instead of interleaving, and every command gets its own copy button.
* **Piped Input:** `dmesg | pastebridge` puts output in a lane with no browser and no clipboard involved. See [Piping Output In](#piping-output-in).
* **Prompt Stripping:** A leading `$ ` or `# ` is removed from every command line on the way in, so what you copy is the command and not somebody's prompt.
* **Readable Long Output:** Entries longer than about nine lines are clipped with a **Show all** control, so one huge stack trace doesn't bury everything below it.
* **Memory Only:** Entries live in the process, capped at the most recent 500 across both lanes, and disappear when it stops.
* **Zero Mobile Footprint:** The other device only needs a browser. Nothing to install, no account, and no internet connection required.
* **Headless & Raspberry Pi Support:** Run `-headless -exit-on-idle 0` on a machine with no screen and pair from your phone or laptop.

## Security Overview

Read this before you switch sharing on.

* **The Gate:** Anyone who can reach the address can try the PIN. The six-digit PIN locks pairing after ten wrong guesses, and only the host can reset it.
* **Full Access:** A paired device is not read-only. It can send to and clear both lanes, exactly as the host can.
* **Encryption:** The certificate is self-signed, so your browser warns you once. It defeats passive listening on the network, but not someone already positioned between your two devices.
* **What You Paste:** Terminal output can contain tokens, connection strings, hostnames, paths, and environment variables. Nothing is redacted, so look before you send.
* **Anything Local Can Write:** The console listener treats any loopback caller as the host, with no PIN. That's what makes piping work, and it means any process running on that machine can add entries.
* **Commands Are Just Text:** Nothing runs, inspects, parses, or validates what arrives in either lane. The absence of a warning is not an assurance. You are the execution step, in your own terminal.
* **Lifespan:** Pairing expires after 12 hours. Switching sharing off ends every paired session, and turning it back on means pairing again.

## Quick Start

Download, check the hash, unpack, run.

### Linux x86-64

```bash
curl -fLO https://github.com/PeerHold/pastebridge/releases/download/v1.1/pastebridge-linux-v1_1.tar.gz && \
curl -fLO https://github.com/PeerHold/pastebridge/releases/download/v1.1/pastebridge-linux-v1_1.tar.gz.sha256 && \
awk '{print $1" *pastebridge-linux-v1_1.tar.gz"}' pastebridge-linux-v1_1.tar.gz.sha256 | sha256sum -c - && \
tar -xzf pastebridge-linux-v1_1.tar.gz && \
chmod +x pastebridge-linux-v1_1 && \
./pastebridge-linux-v1_1
```

### macOS Apple Silicon

```bash
curl -fLO https://github.com/PeerHold/pastebridge/releases/download/v1.1/pastebridge-mac-v1_1.tar.gz && \
curl -fLO https://github.com/PeerHold/pastebridge/releases/download/v1.1/pastebridge-mac-v1_1.tar.gz.sha256 && \
awk '{print $1" *pastebridge-mac-v1_1.tar.gz"}' pastebridge-mac-v1_1.tar.gz.sha256 | shasum -a 256 -c - && \
tar -xzf pastebridge-mac-v1_1.tar.gz && \
chmod +x pastebridge-mac-v1_1 && \
./pastebridge-mac-v1_1
```

### macOS Intel

```bash
curl -fLO https://github.com/PeerHold/pastebridge/releases/download/v1.1/pastebridge-mac-intel-v1_1.tar.gz && \
curl -fLO https://github.com/PeerHold/pastebridge/releases/download/v1.1/pastebridge-mac-intel-v1_1.tar.gz.sha256 && \
awk '{print $1" *pastebridge-mac-intel-v1_1.tar.gz"}' pastebridge-mac-intel-v1_1.tar.gz.sha256 | shasum -a 256 -c - && \
tar -xzf pastebridge-mac-intel-v1_1.tar.gz && \
chmod +x pastebridge-mac-intel-v1_1 && \
./pastebridge-mac-intel-v1_1
```

The binary is unsigned, so Gatekeeper asks the first time. Right-click and choose **Open**, or run `xattr -d com.apple.quarantine pastebridge-mac-intel-v1_1`.

---

### Windows x86-64 · PowerShell

```powershell
curl.exe -fLO https://github.com/PeerHold/pastebridge/releases/download/v1.1/pastebridge-v1_1.zip
curl.exe -fLO https://github.com/PeerHold/pastebridge/releases/download/v1.1/pastebridge-v1_1.zip.sha256
$want = (Get-Content pastebridge-v1_1.zip.sha256 -Raw).Trim().Split(' ')[0].ToLower()
$got  = (Get-FileHash pastebridge-v1_1.zip -Algorithm SHA256).Hash.ToLower()
if ($got -eq $want) { Expand-Archive pastebridge-v1_1.zip -DestinationPath . -Force; .\pastebridge-v1_1.exe } else { Write-Error 'Checksum failed' }
```

After the first run, you can also start it by double-clicking `pastebridge-v1_1.exe`.

---

### Windows on ARM Snapdragon · PowerShell

```powershell
curl.exe -fLO https://github.com/PeerHold/pastebridge/releases/download/v1.1/pastebridge-arm64-snapdragon-v1_1.zip
curl.exe -fLO https://github.com/PeerHold/pastebridge/releases/download/v1.1/pastebridge-arm64-snapdragon-v1_1.zip.sha256
$want = (Get-Content pastebridge-arm64-snapdragon-v1_1.zip.sha256 -Raw).Trim().Split(' ')[0].ToLower()
$got  = (Get-FileHash pastebridge-arm64-snapdragon-v1_1.zip -Algorithm SHA256).Hash.ToLower()
if ($got -eq $want) { Expand-Archive pastebridge-arm64-snapdragon-v1_1.zip -DestinationPath . -Force } else { Write-Error 'Checksum failed' }
```

---

### Raspberry Pi OS 64-bit · Headless (Shared to Phone)

```bash
curl -fLO https://github.com/PeerHold/pastebridge/releases/download/v1.1/pastebridge-linux-arm64-pios-modern-v1_1.tar.gz && \
curl -fLO https://github.com/PeerHold/pastebridge/releases/download/v1.1/pastebridge-linux-arm64-pios-modern-v1_1.tar.gz.sha256 && \
awk '{print $1" *pastebridge-linux-arm64-pios-modern-v1_1.tar.gz"}' pastebridge-linux-arm64-pios-modern-v1_1.tar.gz.sha256 | sha256sum -c - && \
tar -xzf pastebridge-linux-arm64-pios-modern-v1_1.tar.gz && \
chmod +x pastebridge-linux-arm64-pios-modern-v1_1 && \
./pastebridge-linux-arm64-pios-modern-v1_1 -headless -exit-on-idle 0
```

All archives and their SHA-256 files are on the [Releases page](https://github.com/PeerHold/pastebridge/releases). Check the hash before you unpack.

> **Note:** The examples below use `./pastebridge` for short. Use the file name for your platform, such as `./pastebridge-linux-v1_1`.

When it starts, your browser opens at `http://127.0.0.1:8787/`, your own machine. The terminal also prints the address and a six-digit PIN, which you only need for sharing. Plain HTTP is deliberate here: browsers treat `localhost` as a secure origin, so one-click copy works and there's no certificate warning on the host.

Leave the program running while you use it. **By default it stops about five seconds after the last browser tab closes.** See [the idle timer](#the-idle-timer).

## Using pastebridge

### The Two Lanes

| Lane | For | Behaviour |
| --- | --- | --- |
| **Log** | Terminal output, errors, stack traces | Kept exactly as pasted, with whitespace intact |
| **Commands** | One command per entry | A leading `$ ` or `# ` is stripped from every line |

The direction is a convention, not a rule: both devices can write to both lanes. On screens narrower than 720px, the lanes become tabs.

### Sending

Paste into the box at the top of a lane and press **Send**, or use <kbd>Ctrl</kbd>/<kbd>Cmd</kbd> + <kbd>Enter</kbd>.

* Trailing whitespace is trimmed, and empty sends are ignored.
* A single paste can be up to about 512 KB. Larger requests are rejected.
* New entries appear immediately on every connected device, newest at the top.

### Copying

Every entry has its own **Copy** button, which copies the full text, including any part hidden by clipping.

On HTTPS or `localhost`, the browser's clipboard API is used. On a plain-HTTP address that isn't localhost, the browser blocks that API, so pastebridge falls back to a selection-based method and the header shows `http, using fallback copy`. If both are blocked, the button says **Select it manually**.

### Clearing

**Clear** at the top of a lane empties it on the host and on every paired device at once. There is no undo and no trash.

## Piping Output In

On a machine with no browser, or when you'd rather not select text at all, pipe it:

```bash
dmesg | pastebridge                                  # into the log lane
journalctl -u nginx -n 100 | pastebridge             # same
echo "systemctl restart nginx" | pastebridge -lane cmd   # into the command lane
```

What happens depends on whether pastebridge is already running on that machine:

* **Already running:** the text is handed to it over loopback and the pipe exits immediately. This is the everyday case. Leave one running and pipe into it as often as you like.
* **Not running:** this run takes the text, starts up as usual, and the entry is waiting for the first browser that attaches.

Notes:

* `-lane` accepts `log` (the default) or `cmd`. Anything else is an error.
* Input is capped at about 512 KB, the same limit the paste box has. Longer input is truncated, with a warning on stderr.
* Piping into a fresh start with `-headless` or `-no-browser` needs `-exit-on-idle 0`, or it quits before you can read it. It warns you if you forget.
* The handoff is a loopback request to the console listener, which needs no PIN, so nothing leaves the machine.

## Sharing with Another Device

**Sharing is off when the program starts**, unless you pass `-share` or `-headless`. Off means no network listener exists at all, not a listener refusing connections.

Switch it on with the toggle in the header. The panel shows an address and a six-digit PIN. On the other device, open the address and enter the PIN.

### What the Other Device Gets

A paired device has full access: it can send to and clear both lanes. Three things stay with the host:

* Switching sharing on and off
* Resetting the PIN lockout
* Quitting the program

### The PIN

* Six digits, regenerated on every start unless you pass `-pin`.
* Compared in constant time, so a wrong guess reveals nothing through timing.
* **Ten wrong attempts locks pairing.** Reset it with **Reset attempts** in the share panel, or restart for a new PIN.
* A successful pairing lasts **12 hours**, after which the device is asked again.

### Switching Sharing Off

Switching sharing off stops the listener and invalidates every paired session at once. Turning it back on, or restarting, means each device pairs again.

### The Certificate Warning

Sharing uses TLS with a certificate your own machine generated, so the browser warns you the first time. Accepting it once on your own device is expected. It protects against passive listening, not against someone already intercepting traffic between your devices, because nothing tells your browser which self-signed certificate is the right one.

## Running Headless

On a server or Raspberry Pi with no browser:

```bash
./pastebridge -headless -exit-on-idle 0
```

`-headless` starts without a browser, switches sharing on, and prints the address and PIN to the terminal. Open it from a laptop or phone. Once it's running, `command | pastebridge` from the same machine drops output straight into a lane.

**Always pass `-exit-on-idle 0` with `-headless`.** The idle timer counts time since any browser was attached, which on a headless machine is the whole uptime, so with the default it stops before you can connect.

`-no-browser` is the milder version: it starts normally, leaves sharing alone, and just doesn't open a browser.

## Quitting, and Quitting by Itself

Three deliberate ways out:

* **Quit** in the header, which asks for confirmation, since everything in both lanes is lost.
* <kbd>Ctrl</kbd>+<kbd>C</kbd> in the terminal.
* A `SIGTERM` signal, handled the same way.

Both listeners are dropped before the process exits, the network one first.

### The Idle Timer

`-exit-on-idle` quits the program a set time after the last browser detaches. **It defaults to 5 seconds.** That's aggressive on purpose: a forgotten pastebridge is a listener on your network with a live PIN, and the usual cause is closing the tab and walking away.

Two consequences:

* Closing your last tab stops the program within seconds, and reopening the page won't reach anything.
* A page reload detaches and reattaches. The reconnect normally wins, but a slow reload on a slow machine can lose the race.

To change it:

```bash
./pastebridge -exit-on-idle 10m   # quit after ten minutes with nothing attached
./pastebridge -exit-on-idle 0     # never quit on its own
```

An open tab holds a live connection, so the timer stays at zero however long the tab sits idle. It only starts counting once the last tab is gone.

## Options

| Option | Default | What it does |
| --- | --- | --- |
| `-port` | `8787` | Port for the local console, bound to `127.0.0.1` only |
| `-share-port` | `8788` | Port for the shared listener, reachable from your network |
| `-pin` | random | Use a fixed six-digit PIN instead of a new one each run |
| `-share` | off | Switch sharing on at startup |
| `-headless` | off | No browser, sharing on, for a server or Pi |
| `-no-browser` | off | Don't open a browser at startup |
| `-lane` | `log` | Lane for piped input: `log` or `cmd` |
| `-exit-on-idle` | `5s` | Quit this long after the last browser detaches. `0` never quits |

`PORT`, `SHARE_PORT`, and `PIN` also work as environment variables. The command-line flag wins over the variable.

There is no option to bind the console anywhere other than loopback. Other devices connect through [sharing](#sharing-with-another-device), which is the path with a PIN and TLS.

## What It Keeps, and Where

### What You Paste

Held in memory on the host and in each connected browser, capped at the **most recent 500 entries across both lanes**. Older entries fall off the end, and everything goes when the process stops. There is no database, history file, cache, or log of what crossed.

### The Certificate

These two files are the only things pastebridge writes:

```
~/.pastebridge/cert.pem
~/.pastebridge/key.pem
```

They're a self-signed certificate and its private key, kept so your browser doesn't ask you to accept a new one on every restart. The directory and files have owner-only permissions.

The certificate covers `localhost`, your hostname, `127.0.0.1`, and every IPv4 address your machine currently answers on, and it's valid for 825 days. It regenerates automatically when it's close to expiry or when your machine picks up an address it doesn't cover, such as after moving networks. Browsers reject a certificate that's missing the typed address outright, so a stale one is worse than none.

Delete the directory and the next run makes a fresh pair, at the cost of accepting the warning again on every device.

## Troubleshooting

**The browser didn't open.** Go to `http://127.0.0.1:8787/` yourself, or use your `-port` number.

**It quit on its own.** Almost certainly the idle timer. See [the idle timer](#the-idle-timer), and use `-exit-on-idle 0` while you decide.

**`-headless` starts and then exits.** Same cause. Use `-headless -exit-on-idle 0`.

**"Port already in use."** Either pastebridge is already running (check your other tabs) or something else has the port. Use `-port 8900`, and `-share-port 8901` if the conflict is on the sharing side.

**A pipe didn't reach the running copy.** It only looks on `127.0.0.1` at the console port. If you started that copy with `-port 8900`, pipe with `-port 8900` too.

**Sharing won't switch on.** Either the share port is busy, which the panel tells you, or no certificate could be created at startup, which the terminal reported. In the second case, sharing is unavailable for that whole run.

**My phone can't reach the shared address.** Both devices must be on the same network. Many guest and public networks isolate clients from each other, and the app can't work around that. The share panel lists every address the machine answers on, so try the others if the first fails.

**The certificate warning is back.** Your machine's address changed, so the certificate was regenerated. Accept it once more.

**I'm locked out of pairing.** Press **Reset attempts** on the host, or restart for a fresh PIN.

**The status says "reconnecting".** The browser lost its connection and retries every couple of seconds. Anything that reached the server is delivered when it reconnects.

**Copy says "Select it manually".** The browser blocked both clipboard methods. Select the text and copy it the usual way.

## What It Doesn't Do

* **It never runs anything.** Nothing arriving in either lane is executed on either end.
* **It doesn't check whether anything is correct or safe.** Text arrives exactly as typed, and no warning is not the same as no risk.
* **It doesn't redact.** Every paired device sees what you paste, tokens and all.
* **It doesn't work over the internet.** There's no relay or rendezvous server, so both devices must be on the same network, or joined by something that makes them look that way, such as a VPN or a tailnet.
* **It doesn't keep anything.** That's a feature until you close the tab expecting to come back.
* **It has no terminal interface.** The interface is a web page, and the shell is for startup options and pipes.
* No account, no telemetry, no update check, no subscription, and nothing left behind when the process exits.

### A Note on Language Models

pastebridge is often used to move an error into a model and a suggested command back. The program plays no part in that: no model, no API key, no outbound connection.

Two things follow. A model can be confidently wrong, so a suggested command may be destructive, inapplicable, or simply mistaken. And the text you paste becomes part of what the model reads, so content inside your own log output can influence its suggestions, and log output isn't always something you wrote. Treat a suggested command as an untrusted stranger's advice that happens to be well phrased.

## Privacy

pastebridge runs entirely on your own machines.

* No account, no sign-in, and no server belonging to anyone else.
* No usage data, analytics, crash reports, or telemetry.
* No update checks over the network.
* With sharing off, it listens only on loopback, so only the host can reach it. This is the default.
* The only network activity is sharing, which you switch on yourself and which stays on your local network.

| Recorded | Where | Retained |
| --- | --- | --- |
| Entries in both lanes | Memory only | Most recent 500 across both lanes, until cleared or the process stops |
| TLS certificate and key | `~/.pastebridge/` | Until deleted, or replaced when your addresses change |
| Anything else | — | Nothing |

### Verify It Yourself (Linux)

```bash
# 1. Nothing goes out. Expect zero outbound packets, including at startup.
sudo tcpdump -i any host not 127.0.0.1 and port not 22

# 2. Only loopback is listening until you switch sharing on.
ss -ltnp | grep pastebridge

# 3. The only things on disk are the certificate and its key.
ls -la ~/.pastebridge/

# 4. One file, nothing linked in.
ldd ./pastebridge-linux-v1_1
```

Repeat check 2 after switching sharing on, and again after switching it off: off means the listener is gone, not idle. Piping adds no listener, so check 2 looks the same either way.

The retention figures above are stated in the privacy notice shipped with the application, which is the authoritative version.

## Legal

The full legal documents (end-user license agreement, privacy notice, safety notes, publisher information, and third-party licenses) ship inside the application. Open the **About** panel or the footer links to read them. Each carries a version number, effective date, and fingerprint, so a support conversation can establish which text you saw, and each can be downloaded as plain text.

**Those shipped documents are the controlling versions.** This README is a plain-language summary, and where the two differ, the shipped documents govern.

### No Warranty

pastebridge is provided without warranty, to the extent permitted by law. You are responsible for what you paste into it and for what happens when you run something you copied out of it. The program doesn't execute or validate anything and can't tell whether a command is safe in your environment. Treat everything in both lanes as text somebody typed, because it is.

Consumer protection law in your country may give you rights that can't be excluded by agreement. Nothing here is intended to remove them.

### Publisher

© 2026 PeerHold
