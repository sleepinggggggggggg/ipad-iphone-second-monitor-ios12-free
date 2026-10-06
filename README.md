# Turn an iPad or iPhone stuck on iOS 12 into a second display for your Mac

**Free, self-hosted, Xcode-free sideloading.**

LegacyPadDisplay is an iOS 12 client for [OpenDisplay](https://github.com/peetzweg/opendisplay), allowing older iPads and iPhones that cannot run the modern OpenDisplay client to work as a second display for a Mac.

**Built and running for real.**

The client has been successfully tested on:

- iPad mini 2
- iOS 12.5.8
- Lightning USB connection

The following features currently work:

- H.264 video
- Touch input
- Two-finger scrolling
- Mouse cursor
- Cursor position and shape
- Reconnection when the iOS app returns to the foreground

The Mac side uses the upstream **OpenDisplay** application without modification. OpenDisplay creates the virtual display, captures it, encodes the video as H.264, and sends it to the iOS client over USB.

The iOS application is a clean-room implementation of the same protocol and is licensed under MIT.

Unlike Sidecar, this project is intended for devices that are too old to run Apple's modern Sidecar stack or the current OpenDisplay iOS client.

> **Important:** You do **not** need Xcode to install the iOS application on a real device.
>
> The recommended installation method is to use a prebuilt unsigned IPA and **Legacy iOS Kit + Plumesign** to sign and sideload it with your Apple ID.

---

## Requirements

### Mac

- A Mac capable of running the OpenDisplay Mac application
- macOS 26 or later recommended
- A Lightning cable

### iOS device

- iOS **12.0 or later**
- An iPad or iPhone with Lightning
- The device must be able to install applications signed with your Apple ID

The project has been tested on an **iPad mini 2 running iOS 12.5.8**.

An iPad is recommended because its larger display is much more useful as a second monitor.

---

# Installation

## Step 1 — Clone the repository

```bash
git clone https://github.com/cuongpham1/ipad-iphone-second-monitor-ios12-free.git
cd ipad-iphone-second-monitor-ios12-free
```

If the repository contains the OpenDisplay Git submodule and it was not initialized automatically:

```bash
git submodule update --init --recursive
```

---

# Step 2 — Get the LegacyPadDisplay IPA

You do **not** need to open the project in Xcode.

You need the prebuilt unsigned IPA:

```text
LegacyPadDisplay-unsigned.ipa
```

Place the IPA somewhere convenient, for example:

```text
~/Downloads/LegacyPadDisplay-unsigned.ipa
```

The IPA is intentionally unsigned. It will be signed for your device during the sideload process.

---

# Step 3 — Install Legacy iOS Kit

Legacy iOS Kit is used for the sideloading process.

Clone it:

```bash
cd ~/Downloads
git clone --filter=blob:none https://github.com/LukeZGD/Legacy-iOS-Kit.git
cd Legacy-iOS-Kit
```

Start Legacy iOS Kit:

```bash
./restore.sh
```

You do **not** need to use Xcode or build LegacyPadDisplay from source.

---

# Step 4 — Select "Sideload IPA"

Inside Legacy iOS Kit, select:

```text
Sideload IPA
```

Then select:

```text
LegacyPadDisplay-unsigned.ipa
```

Legacy iOS Kit will prepare the application for signing.

---

# Step 5 — Sign with Plumesign

When Legacy iOS Kit asks for the signing method, select:

```text
Plumesign
```

You will then be asked to authenticate with your Apple ID.

Use the Apple ID that you want to use to sign the application.

Depending on your Apple account configuration, Apple may require additional verification.

The signing flow is:

```text
LegacyPadDisplay-unsigned.ipa
        ↓
Legacy iOS Kit
        ↓
Plumesign
        ↓
Apple ID signing
        ↓
Signed IPA
```

No Xcode project build is required.

---

# Step 6 — Connect the iPad

Connect the iPad to the Mac using a Lightning cable.

For the first connection, unlock the iPad and accept the **Trust This Computer** prompt if it appears.

Keep the device connected during the sideloading process.

Legacy iOS Kit will install the signed application onto the connected device.

---

# Step 7 — Trust the developer on the iPad

After installation, open:

**Settings → General → VPN & Device Management**

Find the developer profile associated with the Apple ID used to sign the application.

Tap it and trust the developer.

Then return to the Home Screen and launch:

```text
LegacyPadDisplay
```

The application should open to its waiting screen.

You should see:

```text
Listening on :9000
```

This means the iOS client is running and waiting for the Mac.

---

# Step 8 — Start OpenDisplay on the Mac

The Mac side of this project is the upstream OpenDisplay application.

Download the latest OpenDisplay release:

https://github.com/peetzweg/opendisplay/releases/latest

Install:

```text
OpenDisplay.app
```

into:

```text
/Applications
```

On the first launch, grant OpenDisplay:

- **Screen Recording**
- **Accessibility**

in:

**System Settings → Privacy & Security**

After granting the permissions, completely quit OpenDisplay with:

```text
⌘Q
```

and launch it again.

---

# Step 9 — Connect the iPad

Connect the iPad to the Mac using the Lightning cable.

OpenDisplay discovers the iOS client through:

```text
usbmuxd
```

and communicates with it on port:

```text
9000
```

USB / Lightning is currently the **only connection method documented and tested by this project**.

---

# Step 10 — Use the iPad as a second display

Once the connection succeeds, the black waiting screen on the iPad should be replaced by the Mac desktop.

A mouse cursor should also appear.

On the Mac, open:

**System Settings → Displays**

A new virtual display should be visible.

You can then arrange the display and drag windows onto the iPad.

---

# Reinstalling after the signing period expires

Applications installed using personal Apple ID signing are subject to Apple's signing limitations.

If the application stops launching after the signing period expires, repeat the sideload process:

```text
LegacyPadDisplay-unsigned.ipa
        ↓
Legacy iOS Kit
        ↓
Plumesign
        ↓
Apple ID
        ↓
Sideload IPA
```

You do **not** need to rebuild the application with Xcode.

You do **not** need to modify the source code.

You do **not** need to jailbreak the iPad.

---

# Troubleshooting

## The iPad does not appear during installation

Check that:

1. The iPad is unlocked.
2. The Lightning cable is connected.
3. The iPad trusts the Mac.
4. The USB connection is working.
5. Legacy iOS Kit is running.

Try unplugging and reconnecting the Lightning cable if necessary.

---

## The application installs but will not open

Open:

**Settings → General → VPN & Device Management**

and trust the developer profile associated with the Apple ID used for signing.

Then try launching LegacyPadDisplay again.

---

## The iPad stays on "Listening on :9000"

This means the iOS client is running but has not received a connection from OpenDisplay.

Check:

- The Lightning connection.
- That the Mac can see the device through `usbmuxd`.
- That OpenDisplay is running.

---

## No new display appears on the Mac

Check OpenDisplay's:

- Screen Recording permission
- Accessibility permission

After changing either permission, completely quit OpenDisplay with:

```text
⌘Q
```

and launch it again.

---

## The screen briefly goes black when returning to the app

This is the reconnect process.

The iOS listener stops while the application is in the background.

When the application returns to the foreground, `ensureListening()` starts the listener again.

A short flicker during this process is expected.

---

## `updateRequired` appears

This means the OpenDisplay protocol version has changed.

The current LegacyPadDisplay client intentionally omits the `pv` field from its handshake, causing the Mac to treat it as protocol 1.

Current OpenDisplay releases still accept this protocol.

If a future OpenDisplay release changes that behavior, the client may need to be updated.

---

# What works

Currently tested functionality:

- H.264 video
- USB / Lightning connection
- Touch input
- Touch click
- Touch drag
- Two-finger scrolling
- Mouse cursor
- Cursor position
- Cursor shape
- Reconnection after returning to the foreground

Tested hardware:

```text
iPad mini 2
iOS 12.5.8
Lightning
```

The complete path from the Mac to the physical iPad has been successfully tested.

---

# What is currently missing

- External keyboard support
- Automatic rotation to match the virtual display
- Latency measurement
- Wi-Fi connection

> Wi-Fi has not been tested by the project maintainers and is therefore intentionally not documented as a supported connection method.

---

# License

- `iOS/` — this client, MIT License. See `LICENSE`.
- `Mac/` — Git submodule of `peetzweg/opendisplay`, GPL-3.0. Copyright belongs to its respective authors.

The Mac component is not modified or vendored into this repository.

---

*Keywords: iPad iOS 12 second monitor Mac, old iPad external display, Sidecar alternative iOS 12, Duet Display free alternative, Luna Display free alternative, OpenDisplay iOS 12 client, Lightning USB second screen, legacy iPad second monitor, LegacyPadDisplay, iPad mini 2 second monitor, Xcode-free iOS 12 sideload.*
