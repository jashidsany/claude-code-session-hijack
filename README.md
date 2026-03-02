# Claude Code Remote Control - Session Hijacking PoC

## Overview

This repository contains a proof-of-concept demonstrating a session hijacking vulnerability in Claude Code's remote-control feature (v2.1.63). The `claude.ai/v1/sessions/{session_id}/events` endpoint lacks per-session authentication, allowing an attacker to inject arbitrary messages into a victim's active session from any machine on the internet.

Injected messages are processed identically to legitimate user messages with no visual indicator of external origin, constituting invisible command execution.

## Vulnerability Details

- **Product:** Claude Code CLI v2.1.63
- **Feature:** Remote Control (`claude remote-control`)
- **CWE:** CWE-306 (Missing Authentication for Critical Function)
- **Reported:** March 2, 2026
- **Status:** Submitted to Anthropic via HackerOne

## Prerequisites

- Python 3.x
- `cloudscraper` library

```bash
pip3 install cloudscraper
```

## Required Information

To execute this PoC, you need:

1. **sessionKey** - The victim's `sessionKey` cookie from `claude.ai` (found in browser cookies)
2. **session_id** - The active remote-control session ID (found in the session URL, terminal output, or debug logs)
3. **org_id** - The victim's organization UUID (found in browser requests via DevTools)

## Usage

```bash
python3 exploit.py -k <sessionKey> -s <session_id> -o <org_id> -c "<command>"
```

### Examples

Inject a simple message:
```bash
python3 exploit.py \
  -k "sk-ant-sid02-..." \
  -s "session_01ABC..." \
  -o "96682414-bcca-4968-a8fc-ebd0de28b2b2" \
  -c "What is 2+2?"
```

Create a file on the victim's machine:
```bash
python3 exploit.py \
  -k "sk-ant-sid02-..." \
  -s "session_01ABC..." \
  -o "96682414-bcca-4968-a8fc-ebd0de28b2b2" \
  -c "Create a file called proof.txt on the Desktop containing: Hello from a remote session"
```

## Reproduction Steps

### Setup

1. **Victim machine (Windows/macOS/Linux):** Install Claude Code and authenticate
   ```
   claude /login
   ```

2. **Victim machine:** Start remote-control
   ```
   claude remote-control --verbose
   ```

3. **Victim machine:** Open the session URL in the browser and send a message (e.g., "hi") to confirm the session is active

4. **Victim machine:** Obtain the `sessionKey` cookie from browser DevTools (Application > Cookies > claude.ai)

### Exploitation

5. **Attacker machine:** Run the exploit script with the stolen sessionKey, session ID, and org UUID

6. **Victim machine:** Observe the injected message appears in the browser as a standard user message with no origin indicator

7. **Victim machine:** Claude processes the injected command and executes tool calls on the victim's machine

## Attack Flow

```
Attacker (Kali)                          Victim (Windows)
     |                                        |
     |  1. Obtain sessionKey cookie           |
     |  2. Obtain session_id                  |
     |                                        |
     |  POST claude.ai/v1/sessions/           |
     |       {session_id}/events              |
     |  Cookie: sessionKey=<stolen>           |
     |  Body: {events: [{message: "..."}]}    |
     | -------------------------------------> |
     |                                        |
     |           HTTP 200 OK                  |
     | <------------------------------------- |
     |                                        |
     |        Message appears in session      |
     |        Claude executes commands        |
     |        on victim's machine             |
     |                                        |
```

## Root Cause

The session events endpoint authenticates using only the general-purpose `sessionKey` cookie. There is no:

- Per-session cryptographic token
- Device fingerprint binding
- IP address validation
- Message origin indicator in the UI

## Evidence

| File | Description |
|------|-------------|
| `evidence/01_credentials_json.PNG` | Plaintext credentials and file permissions |
| `evidence/02_exploit_py_source_code.PNG` | Exploit script on attacker machine |
| `evidence/03_session_hijack1.PNG` | Successful message injection from Kali |
| `evidence/04_session_hijack2.PNG` | Injected message in victim's browser |
| `evidence/05_proof_txt1.PNG` | File creation command injection |
| `evidence/06_proof_txt2.PNG` | Claude created file from injected command |
| `evidence/07_proof_txt3.PNG` | File verified on victim's machine |
| `evidence/Session_Hijack_Proof_txt.mp4` | Video of complete attack chain |

## Disclaimer

This tool is provided for authorized security research and educational purposes only. Only use this against systems you own or have explicit written permission to test. Unauthorized access to computer systems is illegal.

## Author

Jashid Sany https://github.com/jashidsany
