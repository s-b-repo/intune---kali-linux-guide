# Intune Portal - Kali Linux Installation Guide

Complete guide for installing Microsoft Intune on Kali Linux (Debian-based rolling release).

**Requirements:**
- Kali Linux (amd64 only)
- A Microsoft 365 organizational account with Intune license
- Internet connection

Tested on:
- Kali Linux Rolling (2026)
- HP ProBook 440 G10 / 13th Gen Intel i7-1355U

---

## Step 1: Install Prerequisites

```bash
sudo apt install curl gpg ca-certificates gnome-keyring
```

## Step 2: Add Microsoft GPG Key

```bash
curl -sSL https://packages.microsoft.com/keys/microsoft.asc -o /tmp/microsoft.asc
gpg --dearmor < /tmp/microsoft.asc > /tmp/microsoft.gpg
sudo cp /tmp/microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
sudo chmod 644 /etc/apt/trusted.gpg.d/microsoft.gpg
rm -f /tmp/microsoft.asc /tmp/microsoft.gpg
```

## Step 3: Add Microsoft Repository

**Important:** Use the Ubuntu 22.04 (jammy) repo, NOT the 20.04 (focal) one. The jammy repo has the latest intune-portal builds with current certificate pins.

```bash
echo "deb [arch=amd64] https://packages.microsoft.com/ubuntu/22.04/prod jammy main" | sudo tee /etc/apt/sources.list.d/microsoft-prod.list
```

## Step 4: Add Debian Bookworm Repos (if not already present)

Intune depends on packages from Debian Bookworm. Check if you already have them:

```bash
grep -q "deb.debian.org/debian bookworm" /etc/apt/sources.list && echo "Already present" || echo "Need to add"
```

If not present, add them:

```bash
echo "deb http://deb.debian.org/debian bookworm main contrib non-free non-free-firmware" | sudo tee -a /etc/apt/sources.list
echo "deb http://deb.debian.org/debian-security/ bookworm-security main contrib non-free non-free-firmware" | sudo tee -a /etc/apt/sources.list
```

## Step 5: Update Package Lists

```bash
sudo apt update
```

## Step 6: Install Intune Portal and Dependencies

The intune-portal package will pull in most dependencies automatically (including `microsoft-identity-broker`, `openjdk-11-jre`, `libwebkit2gtk-4.0-37`). Install the portal and the one WebView dependency that may need manual installation:

```bash
sudo apt install intune-portal libgtk-3-0t64 libwebkit2gtk-4.0-37
```

If apt reports broken dependencies, use:

```bash
sudo apt install -f
```

## Step 7: Register URL Handler

This allows the authentication flow to redirect back to the portal:

```bash
xdg-mime default intune-portal.desktop x-scheme-handler/oneauth
```

## Step 8: Enable and Start Services

Enable the Intune daemon socket (auto-starts on boot):

```bash
sudo systemctl enable --now intune-daemon.socket
sudo systemctl start intune-daemon
```

Start the Microsoft identity brokers:

```bash
sudo systemctl start microsoft-identity-device-broker
systemctl --user start microsoft-identity-broker
```

Verify all services are running:

```bash
systemctl status intune-daemon
sudo systemctl status microsoft-identity-device-broker
systemctl --user status microsoft-identity-broker
```

## Step 9: Reboot

```bash
sudo reboot
```

## Step 10: Run Intune Portal and Enroll

After reboot, launch the portal (do NOT use `--interactive`):

```bash
intune-portal
```

A WebKit login window will appear. Sign in with your organizational Microsoft account. After successful authentication, the portal will:
1. Acquire tokens silently
2. Contact your org's Intune management endpoint
3. Enroll the device

### Verify Enrollment

After enrollment completes, check the device status:

```bash
systemctl --user status intune-agent
```

---

## Troubleshooting

### "Unknown option --interactive"

This error comes from the OneAuth authentication library, not intune-portal itself. The flag is parsed correctly by the portal but passed through to oneauth which rejects it. Run without the flag:

```bash
intune-portal
```

### Certificate verification failed / SSL handshake error

```
Certificate verification failed: InvalidCertificate("Unrecognized public key at depth 1")
```

This means your intune-portal version has outdated certificate pins. Microsoft periodically rotates intermediate certificates. Fix by upgrading:

```bash
sudo apt update
sudo apt install --reinstall intune-portal
```

If still failing, ensure you have the jammy repo (not focal):

```bash
echo "deb [arch=amd64] https://packages.microsoft.com/ubuntu/22.04/prod jammy main" | sudo tee /etc/apt/sources.list.d/microsoft-prod.list
sudo apt update && sudo apt install intune-portal
```

### Daemon not running / connection refused

```bash
sudo systemctl enable --now intune-daemon.socket
sudo systemctl restart intune-daemon
```

### Login succeeds but enrollment fails

Ensure both identity brokers are running:

```bash
sudo systemctl start microsoft-identity-device-broker
systemctl --user start microsoft-identity-broker
```

### "Failed to get image from Graph" (HTTP 404)

This is harmless. It just means no profile photo is set on your Microsoft account.

### WebView / login window doesn't appear

Ensure WebKit and GTK are installed:

```bash
sudo apt install libgtk-3-0t64 libwebkit2gtk-4.0-37 glib-networking
```

Also ensure you have a running display server (Wayland or X11). The portal cannot run headless.

### gnome-keyring errors / credential storage fails

The portal stores credentials via gnome-keyring. Make sure it's running:

```bash
eval $(gnome-keyring-daemon --start --components=secrets 2>/dev/null)
```

---

## Package Versions (tested working)

| Package | Version |
|---------|---------|
| intune-portal | 1.2603.31-jammy |
| microsoft-identity-broker | 2.0.1 |
| libwebkit2gtk-4.0-37 | 2.50.6-1~deb12u1 |
| libgtk-3-0t64 | 3.24.52-1 |
| gnome-keyring | (latest from kali-rolling) |
| openjdk-11-jre | (pulled in as dependency) |
| ca-certificates | 20260223 |
