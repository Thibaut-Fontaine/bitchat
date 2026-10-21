# Build the Wi-Fi LAN branch

This branch is source-complete. There is **no signed IPA or TestFlight build**
in this repository: producing an installable iOS app requires macOS, Xcode and
an Apple signing identity, none of which exist on the Linux workstation where
the branch was developed.

## Build on a Mac

```bash
git clone git@github.com:Thibaut-Fontaine/bitchat.git
cd bitchat
git switch feature/wifi-lan-transport

swift test --filter LAN

xcodebuild -project bitchat.xcodeproj \
  -scheme "bitchat (iOS)" \
  -destination 'generic/platform=iOS Simulator' \
  CODE_SIGNING_ALLOWED=NO build

xcodebuild -project bitchat.xcodeproj \
  -scheme "bitchat (macOS)" \
  -destination 'generic/platform=macOS' \
  CODE_SIGNING_ALLOWED=NO build
```

## Install on two iPhones

1. Open the project in Xcode and select the `bitchat (iOS)` scheme.
2. Set your development team and a unique bundle identifier for a sideload
   build.
3. Install on two iPhones connected to the **same Wi-Fi network**.
4. In **Settings → connectivity**, turn on **wi-fi direct** on both phones.
5. Accept the Local Network prompt, then verify that each peer appears once
   with the wifi badge and that a private message arrives over the LAN link.
6. Repeat with the permission denied and with Wi-Fi off; BLE/Nostr must keep
   working.

An unsigned build from CI cannot be installed on a stock iPhone without
re-signing it with a development identity (AltStore, Sideloadly, Xcode). The
official App Store build does not contain this branch.
