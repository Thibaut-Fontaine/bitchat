# STATUS.md — bitchat (Wi-Fi LAN transport)

Last update: 2026-10-21 (feature branch `feature/wifi-lan-transport`).

## What works

* **LAN transport core (pure Swift, unit-tested).**
  * `LANFrameCodec` / `LANFrameDecoder`: 4-byte big-endian length prefix
    around one `BitchatPacket` per frame; incremental decoder (partial
    writes, split headers), oversized-frame rejection, bounded payload.
  * `LANPeerRegistry`: peer state machine `discovered → handshaking →
    authenticated → disconnected / timedOut`, discovery/handshake/idle
    timeouts, duplicate-peer and re-discovery handling, bounded peer table,
    `TransportPeerSnapshot` publication.
  * `LANTransportSelector`: Wi-Fi over BLE over Nostr, with explicit
    fallback reasons (feature disabled, permission denied / unavailable /
    not granted, peer not on LAN, peer not authenticated).
  * `LANInboundPolicy`: every packet must claim the authenticated link's
    peer ID and, when addressed, be addressed to us. Crossed links use a
    deterministic arbitration rule.
  * `LANTransportConfig` / `LANTransportSettings`: feature flag
    (`bitchat.lan.enabled`, **default off**), Bonjour service type
    `_bitchat._tcp`, service-name ↔ peer-ID mapping, permission state
    machine and connection cap.
* **Network.framework shell (`LANTransport`, `LANConnection`).**
  `NWListener` advertising `_bitchat._tcp` + `NWBrowser` discovery, TCP
  links with framing, the signed hello exchange (Ed25519 signature check +
  peer-ID ↔ Noise-key check), the existing Noise XX handshake over the
  link, encrypted payload send/receive mapped to the existing typed
  transport events, peer snapshots published to the unified peer list.
* **Integration.** The LAN transport is created in
  `ChatViewModelServiceBundle`, added to the router (`[lan, ble, nostr]`)
  and to `UnifiedPeerService` (peers are merged **by peer ID**, so a peer
  reachable over BLE and Wi-Fi appears once with the same fingerprint).
  BLE and Nostr code paths are untouched apart from that seam.
* **Settings and device UI.**
  * **Settings → connectivity → wi-fi direct** runtime toggle. Enabling
    starts discovery immediately; disabling tears down the listener,
    browser and links. The first enable triggers the Local Network prompt.
  * Clear `granted`, `unknown`, `denied` and `unavailable` states, with
    *open settings* and *try again* actions.
  * Peer list shows a wifi badge for a peer with an authenticated LAN link.
  * The existing connectivity banner surfaces a denied or unavailable
    Local Network permission while the toggle is on; Bluetooth and Tor keep
    their priority.
* **Permissions.** `NSLocalNetworkUsageDescription` and
  `NSBonjourServices` (`_bitchat._tcp`) added to the shared `Info.plist`
  (iOS + macOS). The macOS sandbox entitlements already carried
  `com.apple.security.network.client` and `.network.server`.
* **Security hardening.** Single hello per link, advertised-name check,
  sender/recipient checks on every packet, Ed25519 verification for public
  messages, authenticated-only public/encrypted payloads, 64-link cap and
  timeout cleanup for unauthenticated sockets.
* **Docs.** `docs/WIFI_TRANSPORT.md` (wire format, handshake, permissions,
  security limits, out-of-scope list, Mac verification steps).
* **Tests.** `bitchatTests/LANFrameCodecTests.swift`,
  `LANPeerRegistryTests.swift`, `LANTransportSelectorTests.swift`,
  `LANTransportConfigTests.swift`, `LANInboundPolicyTests.swift`
  (swift-testing, in the existing test target). **43 of these pure tests
  were executed with Swift 6.1 on Linux in a stubbed harness and pass.**

## What is untested / not verified here

* **No local Apple build.** This workstation is Linux (no Swift/Xcode
  toolchain for UIKit/Network.framework) and the VPS is Linux too.
* **No macOS CI result.** GitHub Actions starts no job: the account is
  locked for billing. The job annotations say *"The job was not started
  because your account is locked due to a billing issue."* No paid plan or
  spending change was made.
* **No physical-device run.** Two iPhones on the same Wi-Fi are needed to
  confirm Bonjour, the Local Network prompt and the end-to-end handshake.
* **No signed downloadable app.** Building/signing an installable iOS app
  needs Xcode and an Apple signing identity on a Mac; this environment has
  neither. The branch is source-complete but not a signed IPA/TestFlight
  build.
* **Panic wipe.** The LAN transport is not restarted after a panic reset;
  its in-memory identity would be stale. It is stopped with the other
  services.

## Next three steps

1. **Build on a Mac** — `swift test --filter LAN`, then the two
   `xcodebuild` Release builds from `docs/WIFI_TRANSPORT.md`; fix any
   compiler/SwiftLint findings.
2. **Run on two physical iPhones on the same Wi-Fi** with the toggle on:
   check the Local Network prompt, the peer appearing once with the wifi
   badge, and a private message crossing the LAN link.
3. **Resolve GitHub Actions billing or run the macOS CI in the upstream
   repository**, then mark the CI todo in m5 complete and re-check the
   denied/granted/unavailable permission paths.
