# eRemote by Celeratec - MSP Setup Guide

This guide will help you set up eRemote (based on RustDesk) as a ScreenConnect replacement for your MSP.

**Repository:** https://github.com/Celeratec/eremote-server

## Overview

eRemote consists of two main components:
1. **eRemote Server** - Handles ID registration and relay connections
   - `hbbs` - ID/Rendezvous server (handles device registration and connection brokering)
   - `hbbr` - Relay server (relays traffic when direct P2P connection isn't possible)
2. **eRemote Client** - The remote desktop application installed on endpoints

## Quick Start: Server Setup

### Prerequisites
- A server with Docker and Docker Compose installed
- Public IP address or domain name
- Open ports: 21115-21119 (TCP) and 21116 (UDP)

### Step 1: Start the Server

```bash
# Clone your eRemote server
git clone https://github.com/Celeratec/eremote-server.git
cd eremote-server

# Edit docker-compose.yml and replace YOUR_SERVER_IP_OR_DOMAIN with your actual IP/domain
# Then start the services:
docker-compose up -d
```

### Step 2: Get Your Public Key

After starting the server, a keypair is automatically generated:

```bash
# View your public key (you'll need this for clients)
cat data/id_ed25519.pub
```

**Important:** Save this key! Clients will need it to connect securely to your server.

### Firewall Ports Required

| Port | Protocol | Service | Description |
|------|----------|---------|-------------|
| 21115 | TCP | hbbs | NAT type test |
| 21116 | TCP/UDP | hbbs | ID registration and heartbeat |
| 21117 | TCP | hbbr | Relay |
| 21118 | TCP | hbbs | WebSocket (for web client) |
| 21119 | TCP | hbbr | WebSocket relay |

### Step 3: Configure Clients

Download eRemote client from https://github.com/Celeratec/eRemote/releases (or use the standard RustDesk client from https://github.com/rustdesk/rustdesk/releases)

Configure each client with:
1. **ID Server**: `your-server-ip:21116`
2. **Relay Server**: `your-server-ip:21117`  
3. **Key**: Your public key from `data/id_ed25519.pub`

---

## eRemote Branding & Customization

### Option 1: Custom Client via Filename (Quick Method)

Rename the RustDesk executable to embed your server configuration:

```
eremote-host=your-server.example.com,key=YOUR_PUBLIC_KEY,.exe
```

Example:
```
eremote-host=remote.celeratec.com,key=5Qbwsde3unUcJBtrx9ZkvUmwFNoExHzpryHuPUdqlWM=,.exe
```

### Option 2: Full eRemote Branding (Build from Source)

For complete Celeratec/eRemote branding:

#### Key Files to Modify in rustdesk client repo:

1. **App Name** - `libs/hbb_common/src/config.rs`
   ```rust
   // Line 61 - Change "RustDesk" to eRemote
   pub static ref APP_NAME: RwLock<String> = RwLock::new("eRemote".to_owned());
   ```

2. **Organization** - `libs/hbb_common/src/config.rs`
   ```rust
   // Line 46 - Change organization identifier
   pub static ref ORG: RwLock<String> = RwLock::new("com.celeratec".to_owned());
   ```

3. **Flutter App Name** - Multiple locations:
   - `flutter/pubspec.yaml` - Change `name: flutter_hbb` to `eremote` and update description
   - `flutter/android/app/src/main/res/values/strings.xml` - Change `app_name` to `eRemote`
   - `flutter/macos/Runner/Configs/AppInfo.xcconfig` - Change `PRODUCT_NAME` to `eRemote`
   - `flutter/ios/Runner/Info.plist` - Update bundle identifiers to `com.celeratec.eremote`

4. **Icons & Logos** (replace with Celeratec branding):
   - `res/icon.png` - Main app icon (256x256 recommended)
   - `res/icon.ico` - Windows icon (multi-resolution)
   - `res/logo.svg` - Logo SVG
   - `res/mac-icon.png` - macOS icon
   - `flutter/assets/` - Flutter app assets
   - `res/tray-icon.ico` - System tray icon

5. **Default Server Configuration** - Hardcode your eRemote server:
   - Modify `src/custom_server.rs` or embed in executable name
   - Set `PROD_RENDEZVOUS_SERVER` in `libs/hbb_common/src/config.rs`

### Building Custom Client

#### Prerequisites
- Rust toolchain
- vcpkg with dependencies (libvpx, libyuv, opus, aom)
- Flutter SDK (for Flutter builds)

#### Build Commands

```bash
# Desktop (Sciter UI - simpler)
cd rustdesk
cargo build --release

# Desktop (Flutter UI - recommended)
python3 build.py --flutter --release

# Android
cd flutter
flutter build apk --release

# iOS
cd flutter
flutter build ios --release
```

---

## ScreenConnect Feature Comparison

| Feature | ScreenConnect | eRemote | Notes |
|---------|--------------|--------------|-------|
| Remote Desktop | ✅ | ✅ | Full support |
| File Transfer | ✅ | ✅ | Full support |
| Unattended Access | ✅ | ✅ | Service mode |
| Session Recording | ✅ | ❌ | Not in OSS |
| Multi-Monitor | ✅ | ✅ | Full support |
| Clipboard Sync | ✅ | ✅ | Full support |
| Chat | ✅ | ✅ | Basic support |
| Audio | ✅ | ✅ | Full support |
| TCP Tunneling | ✅ | ✅ | Port forwarding |
| Web Client | ✅ | ✅ | Via WebSocket |
| Address Book | ✅ | ✅ | Local + sync |
| Groups/Folders | ✅ | ✅ | Organization |
| User Management | ✅ | ❌* | *Pro version only |
| 2FA | ✅ | ✅ | TOTP support |
| Audit Logs | ✅ | ❌* | *Pro version only |
| Custom Branding | ✅ | ✅ | Build from source |
| Wake-on-LAN | ✅ | ✅ | Built-in |
| Remote Printing | ✅ | ✅ | Windows |

### Missing Features to Consider Adding:

1. **Session Recording** - Could be added via FFmpeg integration
2. **Centralized Device Management** - Build a web portal using the API
3. **Technician Assignment** - Role-based access (Pro feature)
4. **Audit/Compliance Logs** - Database logging enhancement
5. **Custom Installer Generator** - Web-based MSI/EXE generator

---

## RustDesk Server Pro (Commercial Option)

If you need enterprise features without building them yourself:
- https://rustdesk.com/pricing.html
- Includes: User management, audit logs, LDAP, web console

---

## Architecture Diagram

```
                    Internet
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
    ┌─────────┐  ┌─────────┐  ┌─────────┐
    │eRemote  │  │eRemote  │  │eRemote  │
    │ (Tech)  │  │ (User)  │  │ (User)  │
    └────┬────┘  └────┬────┘  └────┬────┘
         │            │            │
         └────────────┼────────────┘
                      │
                      ▼
              ┌───────────────┐
              │ eRemote Server│
              │   (hbbs +     │
              │    hbbr)      │
              └───────────────┘
                      │
         ┌────────────┼────────────┐
         │            │            │
         ▼            ▼            ▼
   ┌──────────┐ ┌──────────┐ ┌──────────┐
   │ Managed  │ │ Managed  │ │ Managed  │
   │ Endpoint │ │ Endpoint │ │ Endpoint │
   └──────────┘ └──────────┘ └──────────┘
```

---

## Next Steps for Celeratec eRemote

1. [ ] Deploy eRemote server with Docker Compose
2. [ ] Configure firewall rules (ports 21115-21119)
3. [ ] Test with eRemote client pointing to your server
4. [ ] Build eRemote client from https://github.com/Celeratec/eRemote
5. [ ] Apply eRemote branding (name, icons, default server)
6. [ ] Build branded clients for Windows, macOS, Linux, Android, iOS
7. [ ] Create deployment scripts/GPO for client installation
8. [ ] Document procedures for Celeratec technicians
9. [ ] Consider building a web portal for device management

---

## Useful Resources

- [eRemote Server (Celeratec Fork)](https://github.com/Celeratec/eremote-server)
- [RustDesk Documentation](https://rustdesk.com/docs/en/)
- [Self-Hosting Guide](https://rustdesk.com/docs/en/self-host/)
- [RustDesk GitHub Issues](https://github.com/rustdesk/rustdesk/issues)
- [RustDesk Discord Community](https://discord.gg/nDceKgxnkV)
