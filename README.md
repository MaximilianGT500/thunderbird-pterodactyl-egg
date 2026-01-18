![Picture](https://pics.re/raw/pwSewC.png)
# Thunderbird WebVNC (noVNC)

## Overview

This Pterodactyl Egg runs **Mozilla Thunderbird** as a GUI desktop application inside a container and exposes it via **WebVNC (noVNC)**.
You can use Thunderbird directly in your browser, without installing a native VNC client.

The Egg provides:

* an **internal VNC server** (TigerVNC / Xvnc) listening on **127.0.0.1:5901** (not publicly exposed)
* a **web gateway** (noVNC + websockify) listening on your **Pterodactyl allocation port**
* no directory listing and an automatic redirect from `/` to the correct noVNC page

---

## What This Egg Does

* Starts a lightweight desktop session (Openbox)
* Launches **Thunderbird** automatically
* Serves **noVNC** on the server’s allocation port (HTTP + WebSockets)
* Protects access using a **VNC password**
* Redirects `http://<host>:<port>/` to the correct noVNC URL automatically

---

## Browser Access

Open in your browser:

`http://<HOST>:<PORT>/`

This will redirect to the noVNC client and autoconnect.
When prompted, enter your **VNC password** to access the desktop.

If you want the direct URL (normally not required), it is:

`http://<HOST>:<PORT>/vnc.html?autoconnect=true&resize=remote&path=websockify`

---

## Requirements & Recommendations

Thunderbird is a full desktop mail client. Allocate enough resources:

* **Recommended:** 1 vCPU, **1–2 GB RAM**
* **Minimum workable:** ~768 MB RAM (depends on usage)
* Disk: depends on mail cache; **2–5 GB** is a good baseline

---

## How Networking Works

* **Public WebVNC (noVNC/websockify):** `0.0.0.0:<SERVER_PORT>`
  This is your **Pterodactyl allocation port**.
* **Internal VNC server:** `127.0.0.1:5901`
  Only used internally; websockify proxies to it.

---

## Environment Variables

### Required

**VNC_PASSWORD**

* Password for VNC authentication
* noVNC will ask for this password when connecting

### Optional

**VNC_RESOLUTION**

* Desktop resolution (strongly impacts CPU/RAM usage)
* Suggested values:

  * `1280x720` (good default)
  * `1024x768` (lowest resource usage)
  * `1366x768` (common laptop size)

**PUBLIC_HOST**

* Used only to print a clean access URL in the server console
* Example: `45.145.40.189` or `node.example.com`

**PUBLIC_PROTO**

* Used only for console output
* Default: `http`
* Use `https` if you access it through a reverse proxy with TLS

---

## Console Output (What You’ll See)

After startup, the server console prints a status message similar to:

* `WEBVNC READY`
* `[OK] noVNC: http://<PUBLIC_HOST>:<PORT>/`
* `[OK] Internal VNC: 127.0.0.1:5901  Display=:1  Resolution=<RES>`

This helps you immediately see the correct URL/port.

---

## Security Notes (Important)

* By default, noVNC is served over **plain HTTP** (no TLS).
* If you expose this publicly, secure it using one of:

  * VPN access
  * SSH tunneling
  * a reverse proxy with **HTTPS** (recommended)

Always use a strong VNC password.

---

## Common Troubleshooting

### noVNC loads but won’t connect

* Verify `VNC_PASSWORD` is set
* Check the console for:

  * internal VNC running on `127.0.0.1:5901`
  * websockify proxying to `127.0.0.1:5901`

### Slow performance / high resource usage

* Lower `VNC_RESOLUTION` (try `1024x768`)
* Increase RAM allocation
* Avoid syncing very large mail folders all at once

### 404 messages in logs

Occasional 404 entries can happen if a browser requests `/favicon.ico` or similar assets. If the noVNC page works, this is not an issue.

---

## Startup Flow (High Level)

1. Internal VNC server (Xvnc) starts on `127.0.0.1:5901`
2. Openbox session starts
3. Thunderbird starts automatically
4. noVNC/websockify starts on your allocation port
5. Root URL `/` redirects to the noVNC client
6. Console prints the final access URL and status
