---
title: "Setting up remote development with tailscale and ssh"
layout: post
---

This fall, I have a class called operating sytems that requires me to dual boot ubuntu w/ my mac. I already have a thinkpad that has ubuntu installed but I decided I could just ssh into the computer from my mac. I think this was a good way for me to understand ssh a little better. 

This guide documents the exact steps to set up a Mac (client) to SSH into a ThinkPad running Ubuntu (server) securely and reliably, even across different networks. It also covers keeping the Ubuntu laptop running with the lid closed and ensuring services survive reboots.

## 1. Install SSH on Ubuntu

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
```

SSH server will now auto-start on every boot.

---

## 2. Install Tailscale for Remote Access

Tailscale creates a private VPN mesh so both laptops can connect as if they’re on the same LAN.

**On Ubuntu:**

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo systemctl enable tailscaled
sudo systemctl start tailscaled
sudo tailscale up
```

Log in using your account (Google/Microsoft/GitHub/etc). Authentication persists across reboots.

**On macOS:**

* Download from [https://tailscale.com/download](https://tailscale.com/download) and install.
* Sign in with the same account.

**Verify connectivity:**

```bash
tailscale ip
```

Note the `100.x.y.z` IP.

Test SSH:

```bash
ssh ubuntu-username@100.x.y.z
```

---

## 3. Configure SSH Alias on macOS

Create/edit `~/.ssh/config`:

```bash
mkdir -p ~/.ssh
nano ~/.ssh/config
```

Add:

```
Host thinkpad
    HostName 100.x.y.z
    User ubuntu-username
```

Now connect simply with:

```bash
ssh thinkpad
```

---

## 4. VS Code Remote SSH Setup

1. Install VS Code on macOS.
2. Install extension: `Remote - SSH`.
3. Cmd+Shift+P → `Remote-SSH: Connect to Host` → select `thinkpad`.
4. VS Code will auto-deploy its server to the ThinkPad.

---

## 5. Prevent Sleep When Lid is Closed

By default Ubuntu suspends on lid close. Disable this behavior:

```bash
sudo nano /etc/systemd/logind.conf
```

Set/uncomment:

```
HandleLidSwitch=ignore
HandleLidSwitchDocked=ignore
```

Restart (safer than reloading logind directly):

```bash
sudo reboot
```

The laptop will now keep running with the lid closed.

⚠️ Note: With lid closed, cooling is reduced. Heavy loads may cause overheating.

---

## 6. Ensure Auto-Restart of Services After Reboot

Both SSH and Tailscale are enabled via `systemctl`. After reboot, they start automatically.

Test:

```bash
sudo reboot
```

Wait 1–2 minutes, then SSH from macOS:

```bash
ssh thinkpad
```

---

## 7. Handling Power Loss / Shutdown

If the ThinkPad is fully powered off, you can’t reach it until it boots. Options:

* **BIOS/UEFI** → Enable *Power on AC attach* (auto-boot when power is restored).
* **Wake on LAN (WoL)** → Only reliable on Ethernet, rarely works on Wi-Fi. Allows remote power-on with a magic packet.

---

## Summary

* SSH + Tailscale makes remote dev possible across any network.
* Lid close won’t suspend the Ubuntu server.
* SSH and Tailscale auto-start after reboot.
* Full shutdown still requires physical power-on unless BIOS/WoL is configured.

This setup makes the ThinkPad behave like a headless server, accessible from VS Code or plain SSH anywhere.

That's about it. I used 'thinkpad' since it was the one that I used but you can go ahead and name your server whatever you want.
