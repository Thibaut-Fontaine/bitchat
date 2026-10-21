# CHECKS.md — human checks

`mox check` imports the open items below into the attention queue (nothing else
in this file is parsed). Add a line whenever a human must verify something the
agent cannot: a VPS service or cron, an API key/quota, a deploy, a data sample.

Format — one check per line, only the title is required:

```text
```

* `due`: `+30m`, `+2h`, `+3d`, `+1w`, `now`, or an RFC3339 timestamp. Omit for
  "whenever you get to it".
* `kind`: `vps`, `api`, `deploy`, `manual`, `data`, `security`, `other`.
* `priority`: `low`, `normal`, `high`, `critical`.
* `detail`: one extra sentence, no secrets.
* `every`: recurrence (`1d`, `12h`); marking it done reschedules it.
* `probe`: read-only VPS check. Supported forms:
  `systemctl is-active <unit>`, `disk_free > 5G`, `http 200 https://…`.
  `mox check probe` marks failures due now with high priority.

Rules:

* Never put tokens, keys, passwords or personal data in this file.
* Mark a line `- [x]` only after the human confirmed; `mox` ignores those.
* Keep titles short and actionable ("Verify the fetcher cron ran", not "VPS").
* Re-running sync refreshes an open item with the same title; done items stay
  done.

## Open items (Wi-Fi LAN transport)

- [ ] Build and unit-test the LAN transport on a Mac | due: +1d | kind: manual | priority: high | detail: run `swift test --filter LAN` and the two xcodebuild Release builds from docs/WIFI_TRANSPORT.md on a Mac; the code was written on Linux without a Swift toolchain.
- [ ] Run the LAN transport on two iPhones on one Wi-Fi | due: +2d | kind: manual | priority: high | detail: enable bitchat.lan.enabled on both devices, accept the Local Network prompt and check that the peer appears once in the peer list with the same fingerprint as BLE and that a private message crosses the Wi-Fi link.
- [ ] Confirm the Local Network permission states | due: +2d | kind: manual | priority: normal | detail: with the permission denied the transport must stay inert and BLE/Nostr must keep working; with it granted the peer list must show the LAN peer.

