# Unpeel for iPhone / iPad

The iOS app — a **remote controller** for Unpeel Hosts (Mac app or headless
TUI), live on TestFlight (`unpeel.com/ios`). It pairs with a Host (QR/paste),
then lists sessions, streams live terminals, sends input/verbs, and receives
attention push notifications. Off-LAN traffic rides the Link relay with a
forward-secret E2E handshake; the relay never sees content.

Product framing (see AGENTS.md): fleet glanceability + attention
notifications + control verbs rank above terminal polish, and the detail
screen stays a live terminal — never a semantic chat UI. Two hard terminal
rules: keyboard focus must never resize the remote grid, and agent-drawn
select menus are detected from rendered viewport text (they fire no hooks).

## Layout

- `UnpeelIOS/` — the Xcode project (SwiftUI app, Ghostty-based terminal
  rendering, `RemotePreviewStore` for previews — mock data is preview-only,
  never runtime)
- `UnpeelIOS/Tools/dev_bridge.py` — dev bridge for driving the simulator
  against a local desktop instance
- `PRD.md` — original product framing

Shared protocol/crypto (pairing, remote control, Relay E2E) lives in
`clients/shared/UnpeelShared`, not here.

## Build & test

Built/deployed via `xcodebuild` (see `docs/agents/clients/dev-builds.md` for signing
and device gotchas). Package tests: `swift test` is broken on macOS for this
package — use the xcodebuild package-scheme recipe in that doc.

## Private TestFlight builds (this fork)

`main` tracks upstream; `private-testflight` adds distribution settings without
changing the application code, branding, or Link endpoints. These builds are
signed by Mehmetcan Budak, not official UX Themes AS releases. Branding and
service use remain subject to upstream's `TRADEMARK.md` and any author permission.

- App Store Connect: **Unpeel Private**, app ID `6824027713`.
- Bundle ID: `com.mehmetcan.unpeel.ios.remote`, signing team `476T8NQCSX`.
- One universal app supports iPhone and iPad, preserving the official app as a
  separate installation. Pair this copy with the Host after installing.
- `PrivateTestFlight.xcconfig` supplies signing overrides; upstream project
  settings and entitlements stay intact.
- `ExportOptionsPrivateTestFlight.plist` uploads **internal-only** builds.
  The **Private** internal group automatically receives uploads; tester access is
  managed in App Store Connect.

Export uses the local **Apple Distribution** identity and the **Unpeel Private
App Store** provisioning profile, not cloud-managed signing. The profile must
include production Push Notifications and be installed in Xcode's provisioning
profile directory. The profile provisioned for this account expires June 2, 2027;
renew it with the matching distribution certificate before expiration.

Run from the repository root. Keep the App Store Connect private key outside the
checkout. Set `ASC_KEY_PATH`, `ASC_KEY_ID`, and `ASC_ISSUER_ID` in your shell from
your existing Apple API-key configuration; never commit credentials.

```sh
git switch private-testflight
git fetch upstream
git merge upstream/main

xcodebuild -project clients/ios/UnpeelIOS/UnpeelIOS.xcodeproj \
  -scheme UnpeelIOSApp -configuration Release \
  -destination 'generic/platform=iOS' \
  -xcconfig clients/ios/UnpeelIOS/PrivateTestFlight.xcconfig \
  -archivePath /tmp/unpeel-private-testflight/Unpeel.xcarchive \
  -derivedDataPath /tmp/unpeel-private-testflight/DerivedData \
  -allowProvisioningUpdates \
  -authenticationKeyPath "${ASC_KEY_PATH:?}" \
  -authenticationKeyID "${ASC_KEY_ID:?}" \
  -authenticationKeyIssuerID "${ASC_ISSUER_ID:?}" archive

xcodebuild -exportArchive \
  -archivePath /tmp/unpeel-private-testflight/Unpeel.xcarchive \
  -exportOptionsPlist clients/ios/UnpeelIOS/ExportOptionsPrivateTestFlight.plist \
  -exportPath /tmp/unpeel-private-testflight/export \
  -allowProvisioningUpdates \
  -authenticationKeyPath "${ASC_KEY_PATH:?}" \
  -authenticationKeyID "${ASC_KEY_ID:?}" \
  -authenticationKeyIssuerID "${ASC_ISSUER_ID:?}"
```

The upload configuration lets Xcode manage the build number. Apple processing and
tester eligibility still determine when a build becomes installable. Pulling
upstream changes does not itself publish an update; archive and upload again.
Preserving Link endpoints does not prove push delivery for this separate signing
identity: the operated relay needs APNs support for the new team and bundle ID.
Verify pairing, remote connectivity, and push notifications on hardware.
