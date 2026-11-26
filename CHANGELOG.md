# Changelog

## 1.2.5 — 2026-11-26

- iPhone: the app's creator field (`NSHumanReadableCopyright`) says `applesaeid`.
- Android phone: the phone app says who built it at the end of **Help**: `Developer: applesaeid`.
- iPhone: Tether sets its CallKit provider up again when iOS restarted its call service under the running app. Until
  now every call then failed until the app was force-quit: calls dialed on the iPhone ended at once ("unknown call
  provider") and incoming calls rang but CallKit never activated their audio (no microphone, no sound). Tether checks
  this whenever it comes to the front, after an audio-system reset and on a refused request, then tries once more;
  the link log says `CallKit: … provider set up again`. The Android client uses Android's Telecom and is not
  affected (#18, thanks @arashfarahani).
- Android phone, Qualcomm phones whose audio configuration wires `incall_music_uplink` without an `afe-proxy-playback`
  mixer path (seen on a Redmi Note 8 with a custom Android 16): the iPhone's voice is now sent into the call through
  the in-call music route (`telephony_tx`). The interception API's uplink went nowhere there (the HAL logged "unable
  to find path 'afe-proxy-playback afe-proxy'"), so the other side heard only what the Android phone's own microphone
  picked up, and only when it lay close. The self-test and the in-call check read the same proxy and reported the
  voice as reaching the call. Tether reads the vendor's mixer configuration once, never injects through the
  interception API there (not even when an earlier self-test chose it), and says why in the link log (#18, thanks
  @arashfarahani).
- Android phone, Qualcomm without a maker's microphone switch (Xiaomi, Redmi, Poco, OnePlus, Samsung with a Qualcomm
  chip and others): **Muted** no longer uses Android's microphone mute. Its audio system mutes the whole uplink with
  it, the iPhone's voice included, while the in-call check still heard that voice and remembered the mute as safe: the
  other side heard nothing (Redmi Note 8; perhaps also #14 on a Poco X3 Pro). The phone's microphone stays on there,
  the Android app says so in orange under **Muted**, the link log and problem report give the reason, and the
  self-test no longer tests that mute. The iPhone's Link tab already shows **In this call: Microphone on** (#18,
  thanks @arashfarahani).
- Android phone: switching between **Off during calls**, **Muted** and **On** keeps what this phone showed of the
  redirect modes. Before, every switch forgot it, so the next calls tried a mode known to fail again, each with a few
  seconds of silence and a late uplink for the rest of that call. Choosing **Off during calls** again while it is
  chosen, or the self-test, still tries them anew (#18, thanks @arashfarahani).

## 1.2.4 — 2026-11-25

- Coming from 1.1.0: uninstall the 1.1.0 Android app first (1.2 APKs are signed with the project's release key), then
  pair again.
- Link: an iPhone app suspended in the background now finds its way back when the Android phone closed the session or
  Tether restarted there (an update): it answers every wake at once and sets a new session up when the phone's
  service changed or a wake finds none, and the Android phone wakes a paired iPhone every 3 minutes while there is
  no session. Before, with Bluetooth tethering on, the app could stay unlinked until it was opened (21 minutes seen).
  Every wake is now in the link log, with why one could not be sent.
- Android phone, Samsung with a Qualcomm chip (Galaxy A73 and others): Tether no longer writes Samsung's call keys or
  switches calls into a redirect mode there. On a Galaxy A73 (Android 16) they left the phone's voice channel busy,
  so call audio and even calls on the phone itself stayed silent until a reboot (#13). Redirect modes are no longer
  tried automatically on any Samsung; vendor keys are written back at helper start only if Tether set them; a phone
  remembered as giving no call audio is tried again after a Tether update; the notice says to restart the phone once
  if its own calls have no sound.
- Android phone: real calls and texts always reach the iPhone and other clients, even while a debug test filter is on,
  and that filter switches itself off after 30 minutes (a forgotten one let a call dialed from the iPhone run silent).
- Android phone: **Help › Send a problem report** shares one `.txt` file (or copies it) with what is needed to find
  any problem: device and Android / One UI version, setup steps, link, Bluetooth and tethering state, the SIMs' network
  and signal, call-audio settings and self-test, the link log and the app's log. Phone numbers, names, message text,
  e-mail, addresses and keys are removed. It also holds call-audio diagnostics from the helper (audio mode, the
  interception API, each active recording with Android's `isClientSilenced`, routes, selected `dumpsys audio` /
  `media.audio_policy` lines), taken now and automatically when a call's path looks dead and at the end of every
  bridged call (the last three are kept), to tell a silenced recording from a phone that gives no call audio (#13).
- Tether Client: **Link › Send a problem report** with the same format (device, setup, link, the phone's status
  without numbers or names, playout and microphone state, link log, app log).
- Issue forms and docs ask for the problem reports instead of pasted logs.

- Design rules for both apps (dark first, the teal accent, palette colours only, English and Persian) in CONTRIBUTING.md,
  checked by `tools/check-guardrails.py` in CI: raw or off-palette colours fail the build, and a pull request may not
  change the palette, icons, app names, ids, signing, workflows or the checks. The new pairing screens use the app's
  own teal and dark cards.
- iPhone: a redesigned first-run Link screen (connect card, numbered steps), a scanner that explains a missing or
  refused camera, and a clearer manual code entry (#17, thanks @ashkan0033).
- iPhone: Link › **Send a problem report** builds one text (device, setup and permissions, link, call audio, the Android
  phone's last status, the link log, the app's events) and shares it as a .txt file or copies it. Phone numbers, e-mail,
  names, Bluetooth and IP addresses, IDs and keys are removed before it leaves the app.
- Contributing: pull request rules (one topic, keep the project's name and ids, never break installed apps, CI changes
  on their own); `main` now requires all CI jobs to pass and a maintainer's approval before a pull request is merged.

## 1.2.2 — 2026-11-25

- Android phone: a call answered or dialed on the Android phone itself (its screen, a headset or a car connected to it)
  stays on that phone. Before, with the iPhone linked, its audio went to the iPhone and the phone's own earpiece and
  microphone were muted, so the call had no audio where it was taken (#13). The iPhone shows such a call as on the
  Android phone, with **Move here**.
- Android phone: when its software gives Tether no call audio (the call's sound reads as silence, as on a Galaxy A73
  with Android 16, #13), the call goes back to the phone itself within seconds instead of staying silent on both ends,
  and later calls are not bridged on that firmware. The iPhone, the Android client and the phone app say so
  (`helper.audio` "blocked"); a system update or a self-test that hears the call turns call audio back on.
- iPhone: the renewal note says to refresh the app in SideStore, AltStore or Sideloadly (it named a developer script).
- Android phone: Call audio › **Link log** shows the link log with Share and Copy (headed by the app version, phone
  model and Android version), so a bug report no longer needs a computer and adb.
- Coming from 1.1.0: uninstall the 1.1.0 Android app first (1.2 APKs are signed with the project's release key), then
  pair again.

## 1.2.1 — 2026-11-25

- Android phone: a redirect audio mode whose effect on the uplink cannot be checked (the phone gives no uplink capture,
  common on custom ROMs) is no longer kept for the call, and no other untested mode is tried on later calls. Before, the
  first one or two calls on such a phone could leave the other side hearing nothing of the iPhone (Poco F3 and
  Poco X3 Pro reports, #14).
- Coming from 1.1.0: uninstall the 1.1.0 Android app first (1.2 APKs are signed with the project's release key), then
  pair again. Everything new in 1.2: see 1.2.0.

## 1.2.0 — 2026-11-25

### Call audio on the Android phone

- The phone's own microphone is kept out of bridged calls fully automatically: the default setting tries a redirect
  audio mode, then the phone maker's own microphone switch (Samsung's tx key, MediaTek's UL mute; never Android's
  general mic mute), then leaves the microphone on, each step checked during the call and remembered per phone model.
  A redirect mode whose effect on the uplink could not be seen is no longer used on later calls (#15); where the
  uplink cannot be checked at all, only Samsung's key (verified on real calls) stays on.
- The call's audio paths open only once the phone is in a call's audio mode (on outgoing calls that can take seconds of
  dialing): no fall-back to older methods for that, and no restart loop (at most one restart, never over the
  self-test's choice) (#15).
- The iPhone's voice goes into the call only once it is connected: nothing piles up while dialing.
- With Bluetooth tethering on: frequent hold-ups get a larger uplink cushion (60 ms instead of 45), and what a long
  stall delivers late is thrown away at once instead of delaying the call for a minute.
- Samsung phones with a Qualcomm chip (Galaxy A71, A73, Fold) also mute the voice-call stream, so the earpiece is silent
  even where Samsung's keys do nothing (#15). The `telephony_tx` path is checked to be routed to the call, and built
  without a low-latency track some Qualcomm phones refuse (#15).
- Hang-up no longer waits for blocked audio reads; start, stop, the mic setting and a new helper session take one lock
  (#15).
- Diagnostics for "no audio" reports (#13, #14): the Link log says when the downlink capture delivers nothing or only
  zeros, when the phone takes nothing from the uplink injection, when the injection is not routed to the call, and how
  long the paths waited; the self-test's rows say the same.

### iPhone

- iPhone: the microphone permission is asked for once the app is paired and again when a call starts; a refused or
  silent microphone is shown on the call screen, in the Link tab and the Link log, and "Your voice in the call" no
  longer says it reaches the other side then (#14).
- iPhone call audio: the microphone is sent only once a call is connected (not while an outgoing call dials); a
  stall's backlog (Bluetooth tethering) plays late and drains in the pauses instead of losing speech; interruptions
  and media-services resets bring the audio back; CallKit callbacks run in order; a speaker choice made while
  connecting is kept; the render-thread flags use a heap-allocated lock (#15).
- iPhone link: renewing a pending connect no longer leaves the link waiting on the cancelled one (#15).

### Android client

- Android client: a malformed `sync.ids` page no longer deletes local rows (#11, thanks @Shabdin-A).
- Android client: the same playout stall handling as the iPhone (a stall's backlog plays late and drains in the
  pauses, cut only beyond 0.8 s; a stall's burst no longer holds the buffer deep).
- Android client: the microphone is sent only once a call is connected (not while dialing), and a refused permission, a recorder that does not open or one that delivers only silence for 6 s is shown on the call screen and the main screen and in the Link log; the permission is asked for again when a call starts (#14).
- Android client: content no longer goes behind the navigation bar, text fields stay above the keyboard, call-screen buttons share the width evenly (#12).

### Platforms

- The phone app installs on Android 8.0 and 9 (was 10): the link runs over GATT there (no L2CAP channels; the phone
  answers the PSM read empty, so the iPhone and the Android client go straight to GATT), texts, contacts, call log and
  call control work, the default phone app is requested through Telecom, a call's direction comes from its first
  state. No call audio before Android 11: the status field `helper.audio` ("unavailable") is now shown on the
  iPhone in Link › Call audio. Hotspot and Bluetooth tethering switches are reported as unavailable before Android 11.

### Developers

- SDK guides: a "Writing a client" checklist for the 1.2 rules (two pairing-code forms, an empty PSM means GATT, the
  microphone only once a call is active, `status.helper.audio`, audio stalls); the examples show the QR link form.
- CONTRIBUTING.md: what "done" means (every doc, translation, SDK page, example and the other client app updated with
  a change) and why no personal Apple or signing account may appear in the repository; the pull request checklist
  asks for both.

### Install, pairing and first run

- Release APKs are signed with the project's own release key (fingerprint in docs/RELEASING.md). The 1.1.0 APKs were
  debug-signed: uninstall the 1.1.0 app before installing 1.2.0, then pair again.
- Install without building: an AltStore / SideStore source for the iPhone app (`releases/latest/download/altstore.json`),
  a step-by-step install guide (docs/INSTALL.md) with SideStore, AltStore, Sideloadly and TrollStore, an FAQ, a
  complete Persian guide (docs/fa/), and release notes that say which download goes on which phone.
- Pairing QR code is now a `tether://pair?c=…` link: the iPhone's Camera app offers to open it in Cellular, which asks before pairing. Old `tether1:` codes still work everywhere; the Android app can show the old code for 1.1 iPhone apps.
- First-run guides: numbered Setup steps with plain instructions on the Android phone (incl. the one-time wireless-debugging pairing), a Set up section on the iPhone's Link tab, and pairing steps in the Android client (English and Persian).

Older entries (1.1.0 and 1.0.0): [CHANGELOG-1.0-1.1.md](CHANGELOG-1.0-1.1.md)
