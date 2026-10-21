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
  * `LANTransportConfig` / `LANTransportSettings`: feature flag
    (`bitchat.lan.enabled`, **default off**), Bonjour service type
    `_bitchat._tcp`, service-name ↔ peer-ID mapping, permission state
    machine.
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
* **Permissions.** `NSLocalNetworkUsageDescription` and
  `NSBonjourServices` (`_bitchat._tcp`) added to the shared `Info.plist`
  (iOS + macOS). The macOS sandbox entitlements already carried
  `com.apple.security.network.client` and `.network.server`.
* **Docs.** `docs/WIFI_TRANSPORT.md` (wire format, handshake, permissions,
  security limits, out-of-scope list, Mac verification steps).
* **Tests.** `bitchatTests/LANFrameCodecTests.swift`,
  `LANPeerRegistryTests.swift`, `LANTransportSelectorTests.swift`,
  `LANTransportConfigTests.swift` (swift-testing, in the existing test
  target).

## What is untested / not verified here

* **No local build or test run.** This workstation is Linux (no Swift
  toolchain, no Xcode, no simulator) and the VPS is Linux too. The only
  executable check is the macOS CI on the feature branch.
* **No physical-device run.** Two iPhones on the same Wi-Fi are needed to
  confirm the Bonjour advertisement, the Local Network permission prompt
  and the end-to-end handshake on real hardware.
* **No simulator run of the LAN transport.** A simulator has no second
  device on the LAN; the transport's runtime behaviour is untested.
* **Runtime settings toggle** for the feature flag is not implemented: the
  flag is read at launch.
* **The LAN transport is not wired into the panic-wipe flow** (a panic
  rotates the keychain identity, and the transport keeps a stale in-memory
  copy). It is not started across a panic reset.
* **UI**: there is no transport badge in the peer list and no LAN state in
  the connectivity indicator yet.

## Next three steps

1. **Build and test on a Mac** — `swift test --filter LAN`, then
   `xcodebuild` for the iOS and macOS targets; fix whatever the compiler
   reports (the code was written without a local Swift toolchain).
2. **Run on two physical iPhones on the same Wi-Fi** with
   `bitchat.lan.enabled = true`: check the Local Network prompt, the peer
   appearing in the unified peer list with the same fingerprint as BLE, and
   a private message crossing the LAN link.
3. **Finish the UX and the remaining DoD items**: the settings toggle with
   the permission states, the LAN badge/state in the peer list and the
   connectivity indicator, and the panic-reset handling of the transport.
