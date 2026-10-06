# Changelog

All notable changes to luci-app-trafficctl since v1.0.0.

---

## [1.22.0] - 2026-10-06

### Features
- redesign Telegram Bot settings with mode toggle, live preview, and template variables ([#2](https://github.com/nobodyman-star/luci-app-trafficctl/issues/2)) ([8d49873](https://github.com/nobodyman-star/luci-app-trafficctl/commit/8d498737de53db551648f254d005a1ecf0b5d4bc))
  Telegram Bot section:
  - Add "Notifications only" / "Full control" segmented toggle
  - Add live chat-bubble preview for notification messages
  - Add inline keyboard preview reflecting toggle states dynamically
  - Add custom notification template with 17 variables:
  {{ name }}, {{ ip }}, {{ mac }}, {{ link }}, {{ date }}, {{ time }},
- add .apk package support for OpenWrt 25.12+ ([2057729](https://github.com/nobodyman-star/luci-app-trafficctl/commit/2057729a18af0c329053d5e7c15ea8128e21a293))
  - build-apk.sh: standalone APK builder using apk-tools mkpkg
  - Lifecycle scripts: pre-upgrade (stop bot), post-upgrade (restart rpcd+bot),
  pre-deinstall (stop+disable bot)
  - build-ipk.sh: add matching preinst/prerm scripts for parity
  - compat.yml: test .apk on 25.12+/snapshot, .ipk on older versions
  - auto-release.yml: build both .ipk and .apk in every release
- add Telegram bot mock and integration tests ([d5e008e](https://github.com/nobodyman-star/luci-app-trafficctl/commit/d5e008e6f0e29c6aac46c18b79efa18c90f232b5))
  Level 2 (mock): template substitution, IP/rate validation, control guard,
  command routing, escaping, keyboard structure — 37 tests, no deps.
- sign packages with usign (IPK) and RSA (APK) ([9418116](https://github.com/nobodyman-star/luci-app-trafficctl/commit/941811643ee637a67a80a0ae5b77b55edde5a1f2))
  - Generate and store public keys in keys/ directory
  - Private keys stored as GitHub Secrets (USIGN_PRIVATE_KEY, APK_PRIVATE_KEY)
  - auto-release: passes KEY_BUILD and PRIVATE_KEY to gh-action-sdk
  - compat: same signing for consistency
  - README: add instructions for installing APK with signature verification
- add Telegram bot E2E tests ([4522c99](https://github.com/nobodyman-star/luci-app-trafficctl/commit/4522c99f0de449257fbc5284955ff57941f2ae86))
  Run real bot script with mocked externals (curl, jsonfilter, uci,
  backend scripts) and verify it processes commands (/devices, /status,
  /help) and callback actions (block, unblock, limit, shape, wblock)
  correctly. Also tests control_enabled guard, unauthorized chat_id
  rejection, and invalid IP sanitization.
- detect flow offload mode and adapt monitoring accordingly ([#5](https://github.com/nobodyman-star/luci-app-trafficctl/issues/5)) ([e567f6c](https://github.com/nobodyman-star/luci-app-trafficctl/commit/e567f6c819f6e6c94e29b173b50c3d259fe11a24))
  ## What changed
- add manual-release workflow for rebuilding existing releases ([172fbc1](https://github.com/nobodyman-star/luci-app-trafficctl/commit/172fbc1cbb6d6167bc251b01ece9b6a759bfc938))
  Cherry-picked early from #8 so we can rebuild v1.5.0's broken
  artifacts without waiting for the full PR to merge — workflow_dispatch
  requires the workflow file to exist on the default branch.
- replace HTML tables with LuCI native div tables, drop tm- CSS system ([4b9c837](https://github.com/nobodyman-star/luci-app-trafficctl/commit/4b9c837ead0189a1937bf1190edadc8cbd78640a))
  - Convert all <table>/<thead>/<tbody>/<tr>/<th>/<td> to LuCI-native
  div.table > div.tr > div.th/div.td throughout buildSummaryTable,
  buildGroupedTable, buildExtendedStatsPanel and buildExtendedStatsLegend.
  - Remove runtime JS CSS-variable injection (isDarkTheme/injectStyles/watchTheme);
  dark mode handled by CSS-only :root[data-darkmode="true"] + prefers-color-scheme.
  - Rename --tm-* CSS vars to --tc-* with LuCI-native fallback chain.
- add SW/HW flow offload toggles to settings UI ([54debcd](https://github.com/nobodyman-star/luci-app-trafficctl/commit/54debcddad8320b658699b348d1d7b602cddee29))
  Adds a "Flow Offload" collapsible section in the settings panel with
  software and hardware offload toggles. HW is disabled when SW is off.
  Changes take effect immediately via fw4 reload (backend via offload_get
  / offload_set rpcd methods).
- batch rDNS, device speed graph, fix offload toggles ([783ec0c](https://github.com/nobodyman-star/luci-app-trafficctl/commit/783ec0c4ba87860631391375973585d4e3ba4f4f))
  - Replace per-IP rdns queue with single network.rrdns.lookup batch call:
  all uncached IPs resolved in one round-trip instead of 4-at-a-time
  sequential requests; removes resolving… stuck state.
- improve flow offload UI — show mode badge and per-toggle descriptions ([a3f3271](https://github.com/nobodyman-star/luci-app-trafficctl/commit/a3f3271931b4f679499f957573b6374e67eb0e94))
  Replace bare "Software"/"Hardware" toggles with:
  - current mode badge (⊘ No offload / ◑ Software / ● Hardware) in colour
  - per-toggle description explaining what each option does and its trade-offs
  - ⚠ warning about broken speed monitoring for hw offload placed directly
  below the hardware toggle instead of in a generic footnote
  - save status now says "Applied — firewall reloading" for clarity
- LuCI native UI, offload fixes, batch rDNS, per-device graph ([f71aff9](https://github.com/nobodyman-star/luci-app-trafficctl/commit/f71aff9bce535b1f14af0546b9a46d7f215bc4f8))
  feat: LuCI native UI, offload fixes, batch rDNS, per-device graph
- discover devices across all LAN bridges and VLANs ([53ccf1b](https://github.com/nobodyman-star/luci-app-trafficctl/commit/53ccf1bc23487571210552f289867a0bb90f8fae))
  Closes #13. The backend assumed a single LAN: tctl_get_lan_device()
  returned one device and summary.sh/bytes.sh filtered conntrack to that
  one /24, so any device on a second bridge (br-lan2, guest) or a tagged
  VLAN was silently dropped — the reporter literally "can't see IP from
  this interface".
- show egress interface per connection for mwan3 ([f1a588e](https://github.com/nobodyman-star/luci-app-trafficctl/commit/f1a588e111498a1227926173ebc1da5dc753c1f2))
  Implements the "which WAN does this connection use" request in #10. For each
  connection, capture the conntrack fwmark and resolve the egress interface via
  `ip route get <dst> mark <mark>`, which honours the policy-routing ip-rules.
- monitor devices on routed/downstream subnets, not just the LAN ([b905fee](https://github.com/nobodyman-star/luci-app-trafficctl/commit/b905fee8cc42f0aaec5a84b4aed492ff13a7620e))
  Sources are now matched against connected LAN subnets plus subnets routed
  via a LAN next-hop (downstream routers), plus optional
  trafficctl.main.extra_subnets. Independently of subnets, any conntrack flow
  that this router SNAT/masqueraded (reply dst != original src) is recognized
  as a forwarded client with zero configuration.
- port forwards tab with inbound traffic control ([2265289](https://github.com/nobodyman-star/luci-app-trafficctl/commit/2265289a327e931472125070b5b6cea70a8cf518))
  New LuCI tab (Devices / Port Forwards) listing every DNAT port forward and
  router-local WAN-open port from the firewall config, with live stats from
  conntrack: active inbound connections, distinct remote clients, and bytes
  in/out per forward (LAN-direct and non-DNATed hits are excluded).
- name devices via manual aliases and private-resolver reverse DNS ([5b6d279](https://github.com/nobodyman-star/luci-app-trafficctl/commit/5b6d27934b24e1ffaf4cecd862c0c30e54cc2a21))
  Routed devices (behind a downstream router) have no DHCP lease on this
  router, so every one of them rendered as "*". Names now resolve by
  precedence: manual alias > DHCP lease > cached reverse DNS.
- optional netifyd (DPI) application labels per device ([5baae30](https://github.com/nobodyman-star/luci-app-trafficctl/commit/5baae30edf7b64b36dc12fd315693a1889492b97))
  Adds an App column showing the top application per device, sourced from
  the Netify Agent when it is installed. Entirely optional: with no netifyd
  binary, no socket, or netify_enabled=0, every entry point returns empty
  and the UI degrades to a dash — nothing else changes.
- rate-limit a subnet or the whole network, per-device or aggregate ([0e4aa44](https://github.com/nobodyman-star/luci-app-trafficctl/commit/0e4aa44d1d9fad3686c8bc5c084662e621423624))
  Limits could only target a single host, so there was no way to cap a
  downstream subnet (10.0.20.0/24 behind the MikroTik) or the network as a
  whole.
- set limits for a subnet or the whole network from the UI ([7327f9f](https://github.com/nobodyman-star/luci-app-trafficctl/commit/7327f9f0cd30d88c658eb69851a540daf33e3364))
  Selecting "All active devices" hid the throttle panel outright, so the
  CIDR and "all" targets the backend gained had no way to be used from
  LuCI. The panel now stays visible in that mode and gains a target field
  ("all" or a CIDR such as 10.0.20.0/24) plus per-device / shared chips.
- show download and upload speed separately per device ([f20c516](https://github.com/nobodyman-star/luci-app-trafficctl/commit/f20c516ccdc3e6cf3bc81a5840a1388a628b18e7))
  The backend already returned bytes_in and bytes_out and the frontend
  already derived both rates, but only the download half reached the table
  — upload was kept solely for the hover graph. On an asymmetric link that
  hid an entire direction of every device's traffic.
- Prometheus exporter, and escape client-controlled names in summary JSON ([9aeb19c](https://github.com/nobodyman-star/luci-app-trafficctl/commit/9aeb19c1fc9920e9f4a7d726e55ef4530ef40174))
  Stats were entirely ephemeral: byte counters come straight from conntrack
  and speed was differenced in the browser, so history died with the page.
  An exporter is the natural fix, and Prometheus's pull model means the
  router stores no time series at all.
- export every recorded metric, including reverse-DNS labels ([b662fce](https://github.com/nobodyman-star/luci-app-trafficctl/commit/b662fcea3be05256a25389fb64c0d1f7d734dec7))
  Extends the exporter from byte counters to everything the app actually
  records, so Grafana can chart it without reaching into the router:
- add per-device upload speed column; fix dead live-update selectors ([14b1469](https://github.com/nobodyman-star/luci-app-trafficctl/commit/14b146987a5472ed78b69d9345ff318a8722b295))
  Closes #32.
- support allow-mode (whitelist) ACLs and stop clobbering them ([40a8f1d](https://github.com/nobodyman-star/luci-app-trafficctl/commit/40a8f1dc5138e17bf36f090d8058d3d7b9c0a9a3))
  Closes #31.
- make back/forward navigate between devices ([0a9a2d4](https://github.com/nobodyman-star/luci-app-trafficctl/commit/0a9a2d4f220f4603c7692b662f8355f88e43a969))
  Addresses #26 item 2: the back gesture on mobile and mouse back buttons did
  nothing, because every state change called history.replaceState().
- optional default limit for devices seen for the first time ([#57](https://github.com/nobodyman-star/luci-app-trafficctl/issues/57)) ([12517f7](https://github.com/nobodyman-star/luci-app-trafficctl/commit/12517f70770ebba15d462896095847e5515d191c))
  Closes the second half of #28. A new Settings section ("New Device
  Defaults") applies a rate limit or a shape the first time a device
  appears on the network. Off by default, and a rate of 0 makes it inert.
- global bmon-style traffic overview ([#26](https://github.com/nobodyman-star/luci-app-trafficctl/issues/26) item 7) ([#58](https://github.com/nobodyman-star/luci-app-trafficctl/issues/58)) ([77bb925](https://github.com/nobodyman-star/luci-app-trafficctl/commit/77bb925d988d72a494d88331ff82bf58f8fdd11c))
  Adds a global overview panel above the device table: an uplink throughput graph, role-badged per-interface rows with sparklines on a shared scale, and top talkers read from the speed map pollBytes() already maintains. No new collection — kernel counters plus data already on the page — and no timer of its own, so it inherits the Poll chip, document.hidden and the existing teardown.
- cumulative Bytes/TCP/UDP columns ([#26](https://github.com/nobodyman-star/luci-app-trafficctl/issues/26)) ([#60](https://github.com/nobodyman-star/luci-app-trafficctl/issues/60)) ([d26cc2c](https://github.com/nobodyman-star/luci-app-trafficctl/commit/d26cc2c637fad9e3d22f958a77efec4795c8a060))
  The Bytes / TCP / UDP columns came straight from live conntrack, so they showed what CURRENTLY TRACKED flows had carried and collapsed the moment those flows aged out — the "2 bytes" in the report. They are now accumulated from deltas and persisted.
- wider Poll/Window choices and router-wide defaults for them ([#63](https://github.com/nobodyman-star/luci-app-trafficctl/issues/63)) ([b32b8d3](https://github.com/nobodyman-star/luci-app-trafficctl/commit/b32b8d3045abf42e233a4c1b330469664e98fb2d))
  The Poll chips offered Off/1/2/5s and Window 5/15/30/60s, both hard-coded, so
  there was no way to ask for the slower polling the request was about: on a
  smaller router a 1s poll is a full conntrack read every second for numbers
  nobody is watching that closely. Poll gains 10s and 30s, Window gains 2m and 5m.
- cut all internet access while keeping the LAN working ([#55](https://github.com/nobodyman-star/luci-app-trafficctl/issues/55)) ([#59](https://github.com/nobodyman-star/luci-app-trafficctl/issues/59)) ([9a95c61](https://github.com/nobodyman-star/luci-app-trafficctl/commit/9a95c61b5f8cc29c4b6de3faabae1ac4f256e7f4))
  A single control that cuts internet access for every device while LAN keeps
  working. Off by default, indefinite until switched off, and the engaged state
  lives in tmpfs so a reboot always restores the internet; 'keep after reboot' is
  its own opt-in rather than the global persist_rules flag, so nobody inherits a
  persistent lockout from an unrelated decision.
- aggregate rate limits per subnet / VLAN ([#71](https://github.com/nobodyman-star/luci-app-trafficctl/issues/71)) ([bea0475](https://github.com/nobodyman-star/luci-app-trafficctl/commit/bea0475473cab2288aec73c7814b5cee7c2f7d85))
  Requested in #64 for Home/IoT/Guest VLANs. The engine already supported an
  aggregate cap — trafficctl-ratelimit.sh takes a CIDR and a 'shared' mode that
  puts the whole target in one bucket — but nothing in the dashboard exposed it
  and neither README nor docs/API.md mentioned CIDR targets or the mode at all.
  That omission is why the issue existed.
- ship translations in the release artifacts ([#76](https://github.com/nobodyman-star/luci-app-trafficctl/issues/76)) ([bac5cf7](https://github.com/nobodyman-star/luci-app-trafficctl/commit/bac5cf78da14216a328db34be614610e23cd5644))
  Adding a .po to this repository had no effect on anything a user could
  install from the Releases page. The OpenWrt feed build compiles po/ with
  luci-base's po2lmo and emits one package per language; build-ipk.sh and
  build-apk.sh, which produce every released artifact, copied root/ and
  htdocs/ and nothing else. A release advertising a complete translation
  would have been English end to end for anyone not building from a feed.
- independent download and upload rate limits ([#80](https://github.com/nobodyman-star/luci-app-trafficctl/issues/80)) ([920cfb2](https://github.com/nobodyman-star/luci-app-trafficctl/commit/920cfb2f600b5ee8f6852583265b79a02f5071bc))
  Issue #66 asked for asymmetric ceilings — a camera that needs generous
  upload and almost no download, children's devices the other way round.
- set upload and download ceilings separately (UI) ([#83](https://github.com/nobodyman-star/luci-app-trafficctl/issues/83)) ([a8124e1](https://github.com/nobodyman-star/luci-app-trafficctl/commit/a8124e18317e61753320bf5aadf213577033e5ca))
  The UI half of #66. The backend already takes two rates; this gives an
  operator a way to enter the second one.

### Bug Fixes
- use realistic mock values and fix grep portability in E2E tests ([ea14796](https://github.com/nobodyman-star/luci-app-trafficctl/commit/ea14796cf66a41090397b3d4fce8f4b3d6aab700))
  Replace generic test values with realistic router data (sanitized MACs
  with locally-administered bit, RFC 5737 WAN IP, realistic uptime/load).
  Add `--` to all grep calls in assert helpers to handle patterns starting
  with dashes (fixes {{signal}} tag test on macOS ugrep).
- add debug output for CI template tag failures ([7d323a0](https://github.com/nobodyman-star/luci-app-trafficctl/commit/7d323a0a1cd6a5bc77717276b1ebf9f3ea20897f))
  Temporary: prints actual file contents on assertion failure to diagnose
  why template tags fail in Ubuntu CI but pass locally on macOS.
- add CI debug output for template tag failures ([ea26690](https://github.com/nobodyman-star/luci-app-trafficctl/commit/ea266908702bd3b6257d025ce2b218d5e2748962))
  Captures bot stderr, known.json state, PATH info, and mock outputs
  when template tags test fails — to diagnose Ubuntu CI-specific issue.
- rename awk variable 'load' to avoid gawk reserved word conflict ([006ba72](https://github.com/nobodyman-star/luci-app-trafficctl/commit/006ba727258e8e936dd76312f94f5845f3051502))
  gawk treats 'load' as a builtin function name and refuses to use it as
  a variable. Rename to 'ld' in format_new_device_msg awk template
  substitution. Also removes temporary CI debug output.
- replace OpenWrt SDK APK build with standalone apk-tools ([a824069](https://github.com/nobodyman-star/luci-app-trafficctl/commit/a824069f95d1977ee89a6fa36423fcba7d84d291))
  The gh-action-sdk can't resolve luci.mk includes because feed ordering
  isn't guaranteed. Switch to standalone build-apk.sh with Alpine's
  apk-tools-static which has mkpkg support.
- build APK without apk mkpkg dependency ([86f2481](https://github.com/nobodyman-star/luci-app-trafficctl/commit/86f248195e46b0d6ad6ade8f7d59e4ff0bd4064a))
  apk mkpkg is only in OpenWrt's fork of apk-tools, not available on
  standard Alpine/Ubuntu. Add fallback that builds APKv2 manually via
  concatenated control+data tar.gz archives.
- trigger signed release build v1.3.2 ([e2926c5](https://github.com/nobodyman-star/luci-app-trafficctl/commit/e2926c530caf9f6dc2b990e9a25796f07a492219))
- remove NO_DEFAULT_FEEDS from manual-release on main ([4212fdb](https://github.com/nobodyman-star/luci-app-trafficctl/commit/4212fdbf5077f4a00dfd39aa6a655699224ff69c))
  workflow_dispatch always uses the YAML from the default branch,
  regardless of the ref input. NO_DEFAULT_FEEDS: 1 excluded the
  packages feed (lua.h headers), breaking SDK builds of luci deps.
- remove duplicate luci EXTRA_FEEDS from manual-release ([0b2653b](https://github.com/nobodyman-star/luci-app-trafficctl/commit/0b2653bdbe6b2f250355272e8ebf3b44b1198d30))
  SDK 25.12.4 already includes luci in its default feeds.conf.
  Adding it again via EXTRA_FEEDS causes "Duplicate feed name 'luci'"
  error and exits with code 25 during feeds update.
- mkdir dist before collecting APK in manual-release ([4853bac](https://github.com/nobodyman-star/luci-app-trafficctl/commit/4853bac041969a51c662501e462bb189a4b4fc98))
  SDK action runs in Docker and may remove the dist/ directory
  created by the earlier ipk build step. Ensure it exists.
- preserve IPK files across SDK Docker step ([fdfbc55](https://github.com/nobodyman-star/luci-app-trafficctl/commit/fdfbc55bb4d5fcdfde17c103675222da93fbe504))
  SDK action may wipe the workspace dist/ directory. Save IPK files
  to /tmp/ before SDK runs, restore them when collecting APK.
- move IPK preservation into build step itself ([ee93b8b](https://github.com/nobodyman-star/luci-app-trafficctl/commit/ee93b8b0d113ef9c7772f3fd261b78e7667511e7))
  Separate step failed despite build-ipk succeeding (possible workspace
  isolation between steps). Move cp to /tmp/ into the same run block.
- install scenarios + restructure for feed install ([1237819](https://github.com/nobodyman-star/luci-app-trafficctl/commit/12378190131d684779e78da9bbad4437846507bb))
  - build-ipk.sh / build-apk.sh: copy status.css alongside status.js
  (was missing in v1.5.0, breaking all styling on install)
  - test_install.sh: install IPK via opkg (not raw tar), exercise
  postinst hooks and package-DB registration; assert status.css
  - auto-release.yml: build APK via OpenWrt SDK like compat.yml,
  drop the broken APKv2 standalone fallback (v1.5.0 APK was
- detect dark mode across Bootstrap, Argon, and OS preference ([4b48f75](https://github.com/nobodyman-star/luci-app-trafficctl/commit/4b48f758bc43e57ac4e80f8f25c79813f965525b))
  Replace static CSS-selector-only dark mode detection with a three-tier
  JS approach:
- fix three regressions from div-table conversion ([ab69fe4](https://github.com/nobodyman-star/luci-app-trafficctl/commit/ab69fe40264937cc870b1fb1c0908b528a826e8c))
  - settings panel loaded expanded despite collapsed=true: settingsBody
  was created without tc-hidden, making state and DOM out of sync from
  the first render; add tc-hidden to initial class.
  - rDNS "resolving…" stuck forever: querySelectorAll('td[data-dst=…]')
  matched the HTML <td> tag, but cells are now <div class="td"> after
  the div-table conversion; changed selector to [data-dst=…].
- correctly handle hardware flow offload — conntrack frozen on Mediatek PPE ([1783880](https://github.com/nobodyman-star/luci-app-trafficctl/commit/17838809e45709a0e8f1369cafba698e695090b5))
  On Mediatek Filogic (and likely other platforms), the nf_flow_table hardware
  offload driver does not implement the flow_offload_stats callback, so the
  flowtable `counter` flag does not actually sync byte counts back to conntrack
  for active flows. Conntrack counters are frozen until connection teardown,
  making real-time speed monitoring show near-zero values.
- use --format ustar in build-ipk.sh to prevent PaxHeader entries ([7b0e717](https://github.com/nobodyman-star/luci-app-trafficctl/commit/7b0e71774292c82387f92787444a007055b3730c))
  bsdtar on macOS defaults to PAX format which embeds PaxHeader/ entries
  in archives. BusyBox tar on OpenWrt extracts them as literal
  directories/files, corrupting the install (real files missing from
  /usr/libexec/rpcd and /www). --format ustar forces old POSIX format
  with no PaxHeader support; all our paths fit within the 100-char limit.
- remove bind-dig dependency, update stale docs ([cb18ab3](https://github.com/nobodyman-star/luci-app-trafficctl/commit/cb18ab3cce2a4e94b6848c96397ae5bfdfc537ce))
  Replace dig with ubus network.rrdns lookup (same rpcd-mod-rrdns the
  LuCI frontend uses) + BusyBox nslookup fallback in trafficctl-rdns.sh
  and trafficctl-device.sh. The device script also gains a batch ubus
  call instead of N sequential dig invocations.
- clean up temp files on signal via EXIT traps ([351ec4c](https://github.com/nobodyman-star/luci-app-trafficctl/commit/351ec4c4c18a76f674055e8ec7e772f11c3e0ad0))
  trafficctl-device.sh (rdns map) and trafficctl-shape-stats.sh (mktemp
  qdisc scratch) removed their /tmp scratch files only on the normal exit
  path. rpcd kills a backend script that exceeds its timeout, which on a
  loaded router is exactly when these run slowly — leaving the temp file
  behind and slowly filling tmpfs (RAM). Add EXIT/INT/TERM traps so the
  files are removed even when the script is killed mid-run. summary.sh
- bot ignored all commands on newer jsonfilter (APK/snapshot) ([0dca39c](https://github.com/nobodyman-star/luci-app-trafficctl/commit/0dca39c045f6cdcb1671e65f97c78389aba2761d))
  Fixes #22 (and the long-standing "sh: out of range" reports). The update
  loop sized itself with:
- telegram bot never delivered multi-line messages nor processed updates ([e9a308b](https://github.com/nobodyman-star/luci-app-trafficctl/commit/e9a308b83b57921ca8c140ad273dd0788c7799fe))
  Two fatal bugs made the bot appear dead even though the LuCI Test button
  worked:
- rate limits and shaping only applied to download, never upload ([9ceed50](https://github.com/nobodyman-star/luci-app-trafficctl/commit/9ceed50a5ddd875f81daf5238c8eba8c3ac121b7))
  A limited device still uploaded at full line rate: an Ookla run against a
  1 Mbit limit measured 0.98 Mbps down but 84 Mbps up.
- upload limiter hooked the bridge, so it matched no traffic ([b9a41e8](https://github.com/nobodyman-star/luci-app-trafficctl/commit/b9a41e8c472e28f087176b52c702ec6e68be9f02))
  The bidirectional fix added a 'ul' chain at netdev ingress on br-lan, but
  a netdev ingress hook bound to a BRIDGE never sees bridged traffic —
  packets are received on the bridge's physical ports. The rule was
  installed and counted zero, so a limited device still uploaded at full
  speed (Ookla: 0.97 Mbps down against a 1 Mbit limit, 88 Mbps up).
- shape upload pre-NAT on an IFB, not at WAN egress ([b4e804d](https://github.com/nobodyman-star/luci-app-trafficctl/commit/b4e804db3a6bba72c94e52c43634534f6051eca7))
  Shaping upload with 'match ip src <client>' on the WAN device cannot
  work behind masquerading: POSTROUTING rewrites the source to the
  router's WAN address before the packet is queued on the egress device,
  so the filter matches nothing. This affects every NATed client, and
  routed/downstream ones especially — 10.0.20.x traffic leaves the WAN as
  the router's own address.
- WAN device fell back to a bogus name, silently disabling download limits ([dbcc38c](https://github.com/nobodyman-star/luci-app-trafficctl/commit/dbcc38c6eea76c875cb7a7ef0de217c36c7686c4))
  tctl_get_wan_device's last resort returned the literal string "wan" — an
  interface NAME, not a device. nft then refused to create the netdev
  ingress chain ("device wan" does not exist), the error was discarded by
  2>/dev/null, and trafficctl-ratelimit.sh still reported ok:true. On a
  router where uci carries no network.wan.device this left the limiter with
  no download rule at all while claiming success:
- read netify telemetry from the socket sink, not the agent API socket ([53ce75c](https://github.com/nobodyman-star/luci-app-trafficctl/commit/53ce75cf9f352eb47ef12edf85fa5097803ad8a6))
  The DPI integration produced nothing on a live Agent v5 install. Its
  netifyd.sock is a request/response API — connecting streams no flow data
  at all, so every collection returned empty while status reported the
  agent installed, running and readable.
- netify sample window was shorter than the aggregator's report interval ([dc25536](https://github.com/nobodyman-star/luci-app-trafficctl/commit/dc255360d7b98b1879ae8e8c07db736922c33216))
  The aggregator publishes roughly every 15s, so the 3s default sample
  almost always returned nothing and collection reported "no data from
  netifyd socket" against a perfectly healthy agent. Default sample is now
  18s (cap 60s) and netify_interval moves to 45s so collection spans a
  full report without overlapping runs.
- download limit never matched a masqueraded client ([7835c31](https://github.com/nobodyman-star/luci-app-trafficctl/commit/7835c3190e7fef9e59aab34e240c65f70c487162))
  Download was policed at WAN ingress with "ip daddr <client>". With
  masquerading on (fw4: ip saddr 10.0.0.0/8 masquerade), a reply arriving
  there is still addressed to the router's own WAN address — conntrack
  restores the client address in prerouting, which runs AFTER the netdev
  ingress hook. So the rule could never match, and a limited client
  downloaded at full line rate (96 Mbps against a 1 Mbit limit).
- the custom rate field closed itself a few seconds after opening ([e4cf17a](https://github.com/nobodyman-star/luci-app-trafficctl/commit/e4cf17a227f0031e3584e2aa4f283704b781a749))
  Clicking "Custom" opened the input, then the next per-device poll shut it
  again: the poll re-syncs the rate panel from the device's current rate,
  and a device with no limit took the else branch, which unconditionally
  re-hid the row (and, for a device with a limit, overwrote whatever was
  being typed).
- per-device speed monitoring died when dynamic counter maps are unsupported ([40445a4](https://github.com/nobodyman-star/luci-app-trafficctl/commit/40445a48f29f357fc340401cbebc8d23b465c91a))
  Enabling flow offload switched byte accounting to the nft-map fallback,
  whose dynamic counter maps the target kernel rejects outright:
- keep the device view live instead of freezing on selection ([724bc8e](https://github.com/nobodyman-star/luci-app-trafficctl/commit/724bc8e677d8a3f943234c4571a990a6004aa946))
  Addresses #26 items 5 and 6.
- route hardcoded colours through theme variables ([b428669](https://github.com/nobodyman-star/luci-app-trafficctl/commit/b428669de9c8c5dba8708f2f0bf093a9c60858b6))
  Closes #30.
- split speed into separate sortable DL and UL columns ([ecea06b](https://github.com/nobodyman-star/luci-app-trafficctl/commit/ecea06bc89420a457a56ad75960ab227c6032887))
  After #27 merged, upload speed was shown twice: the DL Speed cell rendered a
  combined "↓ x / ↑ y" pair, and the separate UL Speed column added in #34
  rendered it again. Beyond the duplication, packing both into one cell means
  neither direction can be sorted on its own — the column sorts by _speed only.
- allocate classids instead of deriving them from the address ([b5fc1ba](https://github.com/nobodyman-star/luci-app-trafficctl/commit/b5fc1ba21fec420cef20d510039132225ec26f33))
  The classid minor came from the last two octets, which collides with the two
  values HTB reserves. Shaping 192.168.0.1 mapped onto the root class 1:1, so
  shape_attach's "tc class del ... 1:1" tore down the root and with it every
  other device's shape; x.x.255.254 mapped onto the default class 1:fffe. Any
  two addresses sharing their last two octets collided as well — 192.168.1.50
  and 10.0.1.50 both became 1:132 — which matters because the app deliberately
- identify rules by target and match their comments in full ([a1d731c](https://github.com/nobodyman-star/luci-app-trafficctl/commit/a1d731c86aaa667729cb70f50a4ef0f25aa50456))
  Two ways a control could report success while doing nothing.
- constrain the activity log path and move activity_log to write ([a0fcf74](https://github.com/nobodyman-star/luci-app-trafficctl/commit/a0fcf74c7bb4c0f50782e46716d1ed4b98ba06d0))
  logging_config_set wrote log_file and max_lines to UCI with no validation, and
  that path is used both as an append target and as a tail/mv rotation target.
  Two consequences for anyone holding the write ACL, which on OpenWrt is a
  delegable per-app grant rather than root:
- pass port ranges through to iptables instead of cutting them short ([25a363b](https://github.com/nobodyman-star/luci-app-trafficctl/commit/25a363bc863c7d7b626acec04bdc8848f4625ef3))
  valid_port accepts a lo-hi range, but the iptables branch collapsed it with
  ${port%-*} and passed only the low port. Pausing 8000-8100 on a router without
  nftables blocked traffic to port 8000 and left the other hundred ports open,
  while do_pause answered "paused 8000-8100" and do_list's port_in matched the
  whole range — so the UI showed the pause as active across a range that was
  almost entirely unprotected. iptables spells a range lo:hi, so that is what it
- keep the bot token out of process arguments ([503c82d](https://github.com/nobodyman-star/luci-app-trafficctl/commit/503c82db8c1ee09bc214807a311b8b2ce5f1c298))
  The API URL embeds the token and was passed to curl as an argument, in a loop
  that runs every few seconds — so the token sat in /proc/<pid>/cmdline for any
  local account to read, and rpcd handed it to trafficctl-telegram-test.sh as $1
  as well. The URL now goes to curl through a config file on stdin, and the test
  script prefers the token in TCTL_TG_TOKEN, keeping the positional argument only
  so an existing caller does not break.
- tear the graph popup down and bound the speed history ([4aa6f10](https://github.com/nobodyman-star/luci-app-trafficctl/commit/4aa6f10bc0f8839b01b56b530c681026e35c8ec7))
  render() appended the graph popup to document.body and started a 2s interval
  for it, and handleTeardown removed neither. Leaving the Devices tab and coming
  back left an orphan popup node behind and another live interval running against
  retained history, once per visit. Both are now owned by the view and cleared on
  teardown.
- declare the tools the scripts call and keep state across sysupgrade ([b63e2c2](https://github.com/nobodyman-star/luci-app-trafficctl/commit/b63e2c2fa2bd368ac0d9d8d423749ba8fb106dca))
  LUCI_DEPENDS listed conntrack, luci-base, rpcd and curl, but the scripts also
  invoke tc (37 call sites in the shaper), iw (16, used unconditionally for WiFi
  station detection) and hostapd_cli (8, with no command -v guard at all). On a
  clean install shaping did nothing and WiFi deauthentication silently no-opped.
  Those three are now hard dependencies. socat/nc and kmod-ifb deliberately are
  not: netify's transport sits behind a capability check and the whole feature is
- pass the section regex to awk through the environment ([53c7d8e](https://github.com/nobodyman-star/luci-app-trafficctl/commit/53c7d8e5044f00428df45ef9a25ef4a0279a1d79))
  The release-notes generator matched commits with a pattern built as
  "^${type_regex}(\\([^)]*\\))?:" and handed it to awk via -v. awk expands
  escape sequences in a -v value, and \( is not one it recognises, so it stored
  a plain ( and warned about both parens. The optional scope group
  (\(...\))? thereby became a mandatory literal one, inverting the match in
  both directions: every fix(scope): subject stopped matching, and fixup: and
- stop release notes hanging on a commit that cites an issue ([613f5c3](https://github.com/nobodyman-star/luci-app-trafficctl/commit/613f5c34d63ff1221a7b4fc6a8b5a7952f69672c))
  The auto-linking loop in the release-notes generator rewrites `subj` in
  place and re-matches from the start:
- nft byte counters reported download and upload swapped ([#54](https://github.com/nobodyman-star/luci-app-trafficctl/issues/54)) ([2a2aa86](https://github.com/nobodyman-star/luci-app-trafficctl/commit/2a2aa8618b7bb28a4fab5e4381b3b59abdb33d98))
  The nft counter backend keyed bytes_in by `ip saddr` and bytes_out by
  `ip daddr`. In the forward hook, packets whose source is a LAN device are
  that device's UPLOAD — so bytes_in accumulated upload and bytes_out
  accumulated download, the exact opposite of every consumer:
- byte counters truncated at 2 GiB by awk's 32-bit %d ([#56](https://github.com/nobodyman-star/luci-app-trafficctl/issues/56)) ([2a21938](https://github.com/nobodyman-star/luci-app-trafficctl/commit/2a219386ede2f733b1c4a8a5e156e268c6424c5c))
  busybox awk formats %d through a 32-bit int, and what it does with a larger
  value is undefined: OpenWrt's build saturates at 2147483647, Alpine's wraps to
  -2147483648. Either way a device past 2 GiB stops reporting real numbers, and
  on the saturating builds it does so silently -- the counter just stops moving.
- released packages left rpcd unaware of the plugin ([#62](https://github.com/nobodyman-star/luci-app-trafficctl/issues/62)) ([f55906a](https://github.com/nobodyman-star/luci-app-trafficctl/commit/f55906a12c4434348d15900bfe8642f199ac3bf6))
  Installing a published package and opening the dashboard gave "RPC call to
  luci.trafficctl/summary failed with error -32000: Object not found" until the
  router was rebooted. rpcd only scans for plugins when it starts, and the
  postinst never restarted it.
- Block WiFi reported success while the device stayed online ([#69](https://github.com/nobodyman-star/luci-app-trafficctl/issues/69)) ([30137d0](https://github.com/nobodyman-star/luci-app-trafficctl/commit/30137d0031d60cb2820b8a1976c8ec2222dcda2e))
  Blocking a device's WiFi from the dashboard wrote the uci maclist, reported
  success, and left the device online. Three defects behind one symptom, all
  found on a live router:
- enforce blocks and upload limits over IPv6 via MAC-keyed rules ([#70](https://github.com/nobodyman-star/luci-app-trafficctl/issues/70)) ([7f6f7e9](https://github.com/nobodyman-star/luci-app-trafficctl/commit/7f6f7e996ea67b90e8f597619939daf5acdf1f78))
  Reported in #67 with a clean reproduction: 9.96 Mbit/s over IPv4 against a
  10 Mbit/s limit, 151 Mbit/s over IPv6 to the same endpoint. The gap was
  systemic — no enforcement path in the package matched IPv6 at all — and the
  block case mattered most: a device reported as blocked had full, unmetered
  IPv6 access, which unlike a limit that under-delivers is invisible until it
  matters.
- label the software-counter offload mode in the status badge ([a22e3ad](https://github.com/nobodyman-star/luci-app-trafficctl/commit/a22e3ad777a3f8c3ce148ab6119bfd395efb65d7))
  tctl_get_offload_mode splits software offload on the flowtable counter
  flag and returns "software-counter" when it is set, but modeLabels in
  status.js only knew the four older values. Anyone on that configuration
  saw the badge fall through to its fallback and print a question mark
  next to the raw mode string, which reads like a fault rather than the
  healthy state it actually is.
- render the Telegram limit preset buttons instead of NaN ([9f40296](https://github.com/nobodyman-star/luci-app-trafficctl/commit/9f40296fc1480e69b9083faf80edb46512c38603))
  The keyboard preview built its limit buttons as
- stop two release runs from killing each other's changelog ([#86](https://github.com/nobodyman-star/luci-app-trafficctl/issues/86)) ([c601798](https://github.com/nobodyman-star/luci-app-trafficctl/commit/c601798551d1d99c30756ca49f46f04a4d9ef281))
  The release job computes a version from "what is on main that the last
  tag does not cover", then edits PKG_VERSION and CHANGELOG.md to say so.
  Both were computed from the commit the run was triggered for, and the
  edits were committed and only then rebased onto main.

### Performance
- stop re-scanning firewall/conntrack state per device ([bab7528](https://github.com/nobodyman-star/luci-app-trafficctl/commit/bab75282da0bfba266436ce8850ad2a8c41dea7b))
  trafficctl-summary.sh rebuilt its entire view by, for every active LAN
  device inside the loop, re-reading all of /proc/net/nf_conntrack and
  re-dumping nft chains/tables and tc classes — work that is identical
  regardless of which IP is being processed. On a router with many
  devices and a large conntrack table this is dozens of heavy forks every
  refresh (and the dashboard refreshes every few seconds), which users

### Other
- add release-please for automated changelog and releases ([f1c9834](https://github.com/nobodyman-star/luci-app-trafficctl/commit/f1c9834b37a19ae091b5c6b993e1309a06c74709))
  - release-please-action watches main, creates Release PR on feat/fix commits
  - release.yml now triggers on release:created (release-please creates the
  tag+release, this workflow uploads IPK assets)
  - workflow_dispatch kept as escape hatch for manual asset rebuilds
- replace release-please with auto-release on every merge ([52ffcf4](https://github.com/nobodyman-star/luci-app-trafficctl/commit/52ffcf4e0d190f550e31423b5cef21751c767183))
  - Remove release-please workflow, config, and manifest
  - Add auto-release.yml: on push to main, detect version bump from
  conventional commits, create tag + GitHub Release + build IPK
  - No manual steps needed — merge feat:/fix: to main = instant release
  - Keep workflow_dispatch as manual fallback
  - Update CLAUDE.md release documentation
- only release on feat/fix/perf commits, not ci/refactor/docs ([8ad6ba6](https://github.com/nobodyman-star/luci-app-trafficctl/commit/8ad6ba699a8d753dd21482827ff3d285ed1de7b5))
- consolidate compat tests into single matrix workflow ([2e79792](https://github.com/nobodyman-star/luci-app-trafficctl/commit/2e79792af28a54992c9ec6b22cb09ad677a66f54))
  Replace 13 individual compat workflow files with one matrix-based
  workflow covering all OpenWrt versions with available Docker images:
- add first+last point release coverage for 24.10 and 25.12 ([49f4f57](https://github.com/nobodyman-star/luci-app-trafficctl/commit/49f4f57ae174469cdde4f4b30d6ca35414c88ab2))
  Test matrix now covers both the earliest and latest point release
  of each modern branch (24.10.1 + 24.10.6, 25.12.0 + 25.12.4)
  to catch regressions across the full patch range.
- expand compat matrix with mips, arm_a9, x86-generic, i386 ([56f9495](https://github.com/nobodyman-star/luci-app-trafficctl/commit/56f94957b8c32796a31c11c550d17d36b997d3e2))
  Add all available Docker architectures to the test matrix:
  - mips_24kc (TP-Link, Netgear, most budget routers)
  - arm_cortex-a9 (Marvell, Qualcomm mid-range)
  - x86-generic (32-bit x86 VMs)
  - i386_pentium4 (legacy 32-bit)
- use openwrt/gh-action-sdk for APK builds instead of manual apk-tools ([f22ae12](https://github.com/nobodyman-star/luci-app-trafficctl/commit/f22ae1253f29303d37e8e21768a61e3abde39d57))
  - auto-release: build .apk via official OpenWrt SDK action (25.12)
  - compat: single build-packages job produces both .ipk and .apk,
  matrix tests download pre-built artifacts
  - Removes fragile meson/apk-tools compilation step
  - SDK handles format selection, signing, and correct metadata
- remove generic-named asset symlinks from releases ([69924c4](https://github.com/nobodyman-star/luci-app-trafficctl/commit/69924c4dc610cbe54ad89488edad7998e189d076))
  Only publish versioned filenames (e.g. _1.5.0-1_all.ipk).
  GitHub's /releases/latest/download/ already resolves the latest
  release, making _latest_ and unversioned copies redundant.
- V=sc verbose for SDK in manual-release + asset hardlinks ([d96aa51](https://github.com/nobodyman-star/luci-app-trafficctl/commit/d96aa519c517d0591f48c85ebaa5c56316600e8c))
  Brings the verbose-SDK and generic-named asset hardlink logic from
  fix/install-and-feed-testing-v2 to main so workflow_dispatch can run
  with the improvements while pull_request workflows are stuck.
- add feature-build workflow for shareable dev IPKs ([972915b](https://github.com/nobodyman-star/luci-app-trafficctl/commit/972915b59abb7b463d7935bc5bd7f2617070e902))
  Triggers on push to feature/** branches. Builds the IPK, smoke-tests it,
  then creates/updates a floating pre-release tagged build-<branch-slug> so
  the download URL stays stable across pushes:
  …/releases/download/build-feature-luci-native-ui/luci-app-trafficctl.ipk
- add APK build to feature-build workflow ([00ada2f](https://github.com/nobodyman-star/luci-app-trafficctl/commit/00ada2fcb46fd9f663e1bd40595256981b25d898))
  Split into three parallel jobs: build-ipk, build-apk (via openwrt/gh-action-sdk),
  and publish. APK is unsigned (no PRIVATE_KEY) since feature builds install with
  --allow-untrusted anyway. Both artifacts get stable download URLs under the same
  floating pre-release tag.
- treat warnings as errors, remove dead code ([9f73fcc](https://github.com/nobodyman-star/luci-app-trafficctl/commit/9f73fcc145edd356705ecbf4304200f2f03cd7b2))
  The ESLint job ran `npx eslint` with no `--max-warnings`, and the config
  marked several rules as "warn" — so eslint exited 0 and CI passed green
  while reporting 4 problems. These were real issues silently shipped:
  dead `mkInlinePick` (47-line unused function), unused `rateBtn` and
  `docsUrl`, and a multi-line `if` without braces.
- scan every script strictly, fix uncovered findings ([c1e253a](https://github.com/nobodyman-star/luci-app-trafficctl/commit/c1e253ae58ede04169cf3482fb9f2a298991df3c))
  The ShellCheck job only really scanned root/usr/local/bin. Its
  `additional_files` were passed as full paths, but ludeeus/action-shellcheck
  treats that input as `find -name` basename globs — so the values became
  `-name '*…/…/…'` patterns that match nothing, and the init.d script, the
  rpcd backend and the iface hotplug script were silently never linted.
  Build scripts, the dhcp hotplug script and the whole tests/ tree were not
- fix feature-build.yml YAML so the workflow can run at all ([50ebdac](https://github.com/nobodyman-star/luci-app-trafficctl/commit/50ebdac3735dcf27f23bbb185f1c184e37216c28))
  The "Create pre-release" step embeds a `cat <<NOTES … NOTES` heredoc
  inside a `run: |` block scalar. The heredoc body and the closing `NOTES`
  delimiter were written at column 0 — less indented than the block scalar's
  content (10 spaces) — so YAML terminated the scalar early and GitHub
  rejected the whole workflow at parse time ("could not find expected ':'").
  Every feature-build run therefore failed before any job started.
- make feed-install blocking and stop masking make failures ([a1865bd](https://github.com/nobodyman-star/luci-app-trafficctl/commit/a1865bd7efd82a27b5e5e5978fd193dac92c94eb))
  The feed-install job carried `continue-on-error: true`, so a real
  regression in the user-facing `scripts/feeds` workflow reported green. The
  original lua.h SDK problem is already worked around inside
  test_feed_install.sh (it disables the liblucihttp-lua/ucode deps), and the
  job passes on every supported SDK — so make it a blocking check again.
- fail loudly on silent release-workflow errors ([2c63444](https://github.com/nobodyman-star/luci-app-trafficctl/commit/2c63444e57d5eb351f7c066b7aebe5caaee37989))
  - auto-release: verify the PKG_VERSION sed actually rewrote the Makefile.
  If the substitution silently no-ops (e.g. Makefile format changes), the
  workflow would otherwise commit + tag a release with an unbumped version.
  - manual-release: the "Verify target release exists" step only printed a
  warning ("will create it") when the release was missing, but the later
  `gh release upload --clobber` cannot create one and fails late after a
- bump actions off the deprecated Node 20 runtime ([df0095a](https://github.com/nobodyman-star/luci-app-trafficctl/commit/df0095adf9ae9149b5830557e693a40512c52c76))
  The OpenWrt Compatibility run emitted ~58 warning annotations, one per
  job, all of the form "Node.js 20 is deprecated … forced to run on Node.js
  24: actions/download-artifact@v4, docker/setup-qemu-action@v3".
- trim compat matrix to x86-64 (non-amd64 rootfs images don't exist) ([4dca330](https://github.com/nobodyman-star/luci-app-trafficctl/commit/4dca3301b12a40812c55b442f013cad91f63c5fd))
  The matrix listed 52 arch×version cells but only the 8 x86-64 ones ever
  ran. For every other arch `docker pull ghcr.io/openwrt/rootfs:<arch>` failed
  with "no matching manifest" and the job green-skipped via `exit 0`, so ~44
  of 52 jobs reported pass while testing nothing.
- retry docker pull to survive transient ghcr.io blips ([f868246](https://github.com/nobodyman-star/luci-app-trafficctl/commit/f868246d3f5bf8e45c0a595aa0d73c4f3fa0e78c))
  Single-attempt docker pulls allow transient ghcr.io registry hiccups
  (e.g. "Error response from daemon: Head https://ghcr.io/...") to red
  the entire compat CI run. This change adds a 3-attempt retry loop with
  linear backoff (5s, 10s, 15s) at all four docker pull sites in the
  workflow, making the CI resilient to brief registry unavailability.
- make auto-release work on forks without tags or signing secrets ([906ecee](https://github.com/nobodyman-star/luci-app-trafficctl/commit/906ecee65f027da0ce03f31c606ac0c216d40804))
  Two failure modes on a freshly forked repo:
- read UCI config via config_load/config_get ([ee85ab3](https://github.com/nobodyman-star/luci-app-trafficctl/commit/ee85ab36f499b4c6ed653d355f9233d1a5b19ca3))
  load_config() in trafficctl-telegram.sh read 12 UCI options with
  sequential `uci -q get trafficctl.telegram.X || echo <default>` calls,
  each forking a separate `uci` process. It runs both at bot startup and
  every ~60s in the main poll loop, so that's up to 12 forks per cycle.
- serialise release jobs so they stop racing on git push ([b024f09](https://github.com/nobodyman-star/luci-app-trafficctl/commit/b024f0966b108b72d850be0dec84bf4154ccb8a5))
  Merging #40 and #41 four seconds apart started two Auto Release jobs at
  once. Both computed the same next version and both tried to push main plus
  the new tag; one won and the other died with "failed to push some refs" —
  after it had already tagged. The result was a v1.13.1 tag with no GitHub
  Release and no artifacts behind it.
- actually enforce the ES5 rule the project relies on ([a5c22bf](https://github.com/nobodyman-star/luci-app-trafficctl/commit/a5c22bffb45bf281e5c6ad94ea47ac8cf522c225))
  The frontend must stay ES5 because LuCI serves it to whatever browser the
  router is administered from, and CLAUDE.md states that as a hard constraint —
  but the lint config parsed at ecmaVersion 2020 with no syntax restrictions, so
  let, arrow functions, template literals and .includes() all passed CI cleanly.
  The parser is now ES5 and no-restricted-syntax rejects the ES6 constructs by
  name, so a slip fails the build instead of reaching a user. Verified the
- pin the signing action and fix the release state machine ([afbac56](https://github.com/nobodyman-star/luci-app-trafficctl/commit/afbac56f381b408289ac5b3897c8d767e17dc118))
  The two release workflows and the compat build all used
  openwrt/gh-action-sdk@main — a mutable branch in a repository we do not
  control — and the release paths hand it APK_PRIVATE_KEY and USIGN_PRIVATE_KEY.
  Anyone able to push there could have added one line to the entrypoint and
  walked away with the usign key whose public half ships in keys/, then signed
  packages every existing installation trusts. All three are pinned to the v11
- anchor the breaking-change footer search to the start of a line ([11edb5f](https://github.com/nobodyman-star/luci-app-trafficctl/commit/11edb5f59d535cd3dfe279fcbca6adab8052233e))
  The major-bump condition searched the full commit message for the footer name
  as free text, unanchored. Conventional Commits defines it as a footer — start
  of line, followed by ":" or " #" — so any prose that merely named it counted.
- tell "snapshot feeds are down" apart from "our package is broken" ([895a09f](https://github.com/nobodyman-star/luci-app-trafficctl/commit/895a09f236d72d3f4869ad49b00fb37bb778a917))
  The snapshot/x86-64 job has failed on every open PR for two days, blocking
  #48, #49 and #50 — none of which could affect it (two touched only README,
  one only an awk expression in a workflow). Inside the container apk fails
  with "wget: exited with error 8" then "unable to select packages": OpenWrt's
  rolling snapshot feeds are not serving. The same job passed on main on
  2026-09-04, the image tag resolves fine, and reruns a day apart did not help.
- stop running every check two and three times ([#78](https://github.com/nobodyman-star/luci-app-trafficctl/issues/78)) ([9e43bdc](https://github.com/nobodyman-star/luci-app-trafficctl/commit/9e43bdc9dde250d6fdbb3c1143b8d945b19c243f))
  The workflows nest: auto-release.yml calls ci.yml and compat.yml, and
  ci.yml in turn calls shellcheck.yml, eslint.yml and tests.yml. All five
  of those also carried their own push and pull_request triggers, so each
  one ran standalone AND as a nested job.
- match persisted records by fixed string, not by regex ([#84](https://github.com/nobodyman-star/luci-app-trafficctl/issues/84)) ([ee51ffa](https://github.com/nobodyman-star/luci-app-trafficctl/commit/ee51ffae7c587445df32ff95fa6b1f9b1353bb31))
  tctl_persist_save and tctl_persist_remove pick the record to replace by
  building an awk pattern from the target and matching it with ~, which
  makes the value a regular expression.
- cancel superseded runs on a pull request ([#87](https://github.com/nobodyman-star/luci-app-trafficctl/issues/87)) ([fb9d417](https://github.com/nobodyman-star/luci-app-trafficctl/commit/fb9d41730d1c0af15b3f8c35eb93fb673adb812f))
  Neither ci.yml nor compat.yml declared a concurrency group, so every push
  to a branch started another full OpenWrt compatibility matrix and the old
  one kept running. On 2026-09-29 that put nine compat runs across three
  branches in flight at once — four of them superseded commits on a single
  branch — 153 jobs against the hosted concurrency limit, and runs stretched
  to 34-55 minutes against a 24-27 minute uncontended baseline.
- build the release artifacts parallel to the checks, not after them ([#88](https://github.com/nobodyman-star/luci-app-trafficctl/issues/88)) ([4b4edc0](https://github.com/nobodyman-star/luci-app-trafficctl/commit/4b4edc00c8afba9cbade0d6ad5f7ffb9777455d3))
  A release took 57 minutes for about 26 minutes of distinct work, because
  the OpenWrt SDK build ran TWICE in sequence: once inside compat.yml as
  `Build packages`, and again in the release job, which could only start
  after `needs: [ci, compat]` was satisfied. Measured on run 36856850010 —
  `Build packages` 30m47s, then the release job's SDK step 25m42s, while
  every other step in that job together took 23 seconds.

---

## [1.21.4] - 2026-10-01

### Other
- build the release artifacts parallel to the checks, not after them ([#88](https://github.com/YusDyr/luci-app-trafficctl/issues/88)) ([4b4edc0](https://github.com/YusDyr/luci-app-trafficctl/commit/4b4edc00c8afba9cbade0d6ad5f7ffb9777455d3))
  A release took 57 minutes for about 26 minutes of distinct work, because
  the OpenWrt SDK build ran TWICE in sequence: once inside compat.yml as
  `Build packages`, and again in the release job, which could only start
  after `needs: [ci, compat]` was satisfied. Measured on run 36856850010 —
  `Build packages` 30m47s, then the release job's SDK step 25m42s, while
  every other step in that job together took 23 seconds.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.21.3...v1.21.4

---

## [1.21.3] - 2026-10-01

### Other
- cancel superseded runs on a pull request ([#87](https://github.com/YusDyr/luci-app-trafficctl/issues/87)) ([fb9d417](https://github.com/YusDyr/luci-app-trafficctl/commit/fb9d41730d1c0af15b3f8c35eb93fb673adb812f))
  Neither ci.yml nor compat.yml declared a concurrency group, so every push
  to a branch started another full OpenWrt compatibility matrix and the old
  one kept running. On 2026-09-29 that put nine compat runs across three
  branches in flight at once — four of them superseded commits on a single
  branch — 153 jobs against the hosted concurrency limit, and runs stretched
  to 34-55 minutes against a 24-27 minute uncontended baseline.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.21.2...v1.21.3

---

## [1.21.2] - 2026-10-01

### Other
- match persisted records by fixed string, not by regex ([#84](https://github.com/YusDyr/luci-app-trafficctl/issues/84)) ([ee51ffa](https://github.com/YusDyr/luci-app-trafficctl/commit/ee51ffae7c587445df32ff95fa6b1f9b1353bb31))
  tctl_persist_save and tctl_persist_remove pick the record to replace by
  building an awk pattern from the target and matching it with ~, which
  makes the value a regular expression.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.21.1...v1.21.2

---

## [1.21.1] - 2026-10-01

### Bug Fixes
- stop two release runs from killing each other's changelog ([#86](https://github.com/YusDyr/luci-app-trafficctl/issues/86)) ([c601798](https://github.com/YusDyr/luci-app-trafficctl/commit/c601798551d1d99c30756ca49f46f04a4d9ef281))
  The release job computes a version from "what is on main that the last
  tag does not cover", then edits PKG_VERSION and CHANGELOG.md to say so.
  Both were computed from the commit the run was triggered for, and the
  edits were committed and only then rebased onto main.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.21.0...v1.21.1

---

## [1.21.0] - 2026-09-30

### Features
- set upload and download ceilings separately (UI) ([#83](https://github.com/YusDyr/luci-app-trafficctl/issues/83)) ([a8124e1](https://github.com/YusDyr/luci-app-trafficctl/commit/a8124e18317e61753320bf5aadf213577033e5ca))
  The UI half of #66. The backend already takes two rates; this gives an
  operator a way to enter the second one.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.20.1...v1.21.0

---

## [1.20.1] - 2026-09-30

> This release also contains the asymmetric rate-limit backend below, which the
> release job failed to record at the time: a second release run was in flight
> and its bump commit conflicted on this file, so its notes were lost (#85). The
> entry is restored by hand, and the version number stays 1.20.1 rather than
> being rewritten — the tag is published and must not change meaning.

### Features
- independent download and upload rate limits ([#80](https://github.com/YusDyr/luci-app-trafficctl/issues/80)) ([920cfb2](https://github.com/YusDyr/luci-app-trafficctl/commit/920cfb2f))
  The enforcement was already two-sided — the limiter polices download at
  LAN egress and upload at LAN ingress as separate nftables rules on
  separate hooks, and the shaper builds separate HTB classes on separate
  devices. They were symmetric only because both halves took the same
  number. This threads a second number through the scripts, the rpcd
  surface, the persisted records and the reboot restore. An absent upload
  rate keeps meaning "same as download" everywhere, so every record written
  before this reads back unchanged.

### Other
- stop running every check two and three times ([#78](https://github.com/YusDyr/luci-app-trafficctl/issues/78)) ([9e43bdc](https://github.com/YusDyr/luci-app-trafficctl/commit/9e43bdc9dde250d6fdbb3c1143b8d945b19c243f))
  The workflows nest: auto-release.yml calls ci.yml and compat.yml, and
  ci.yml in turn calls shellcheck.yml, eslint.yml and tests.yml. All five
  of those also carried their own push and pull_request triggers, so each
  one ran standalone AND as a nested job.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.20.0...v1.20.1

---

## [1.20.0] - 2026-09-30

### Features
- ship translations in the release artifacts ([#76](https://github.com/YusDyr/luci-app-trafficctl/issues/76)) ([bac5cf7](https://github.com/YusDyr/luci-app-trafficctl/commit/bac5cf78da14216a328db34be614610e23cd5644))
  Adding a .po to this repository had no effect on anything a user could
  install from the Releases page. The OpenWrt feed build compiles po/ with
  luci-base's po2lmo and emits one package per language; build-ipk.sh and
  build-apk.sh, which produce every released artifact, copied root/ and
  htdocs/ and nothing else. A release advertising a complete translation
  would have been English end to end for anyone not building from a feed.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.19.1...v1.20.0

---

## [1.19.1] - 2026-09-30

### Bug Fixes
- label the software-counter offload mode in the status badge ([a22e3ad](https://github.com/YusDyr/luci-app-trafficctl/commit/a22e3ad777a3f8c3ce148ab6119bfd395efb65d7))
  tctl_get_offload_mode splits software offload on the flowtable counter
  flag and returns "software-counter" when it is set, but modeLabels in
  status.js only knew the four older values. Anyone on that configuration
  saw the badge fall through to its fallback and print a question mark
  next to the raw mode string, which reads like a fault rather than the
  healthy state it actually is.
- render the Telegram limit preset buttons instead of NaN ([9f40296](https://github.com/YusDyr/luci-app-trafficctl/commit/9f40296fc1480e69b9083faf80edb46512c38603))
  The keyboard preview built its limit buttons as

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.19.0...v1.19.1

---

## [1.19.0] - 2026-09-30

### Features
- aggregate rate limits per subnet / VLAN ([#71](https://github.com/YusDyr/luci-app-trafficctl/issues/71)) ([bea0475](https://github.com/YusDyr/luci-app-trafficctl/commit/bea0475473cab2288aec73c7814b5cee7c2f7d85))
  Requested in #64 for Home/IoT/Guest VLANs. The engine already supported an
  aggregate cap — trafficctl-ratelimit.sh takes a CIDR and a 'shared' mode that
  puts the whole target in one bucket — but nothing in the dashboard exposed it
  and neither README nor docs/API.md mentioned CIDR targets or the mode at all.
  That omission is why the issue existed.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.18.2...v1.19.0

---

## [1.18.2] - 2026-09-29

### Bug Fixes
- enforce blocks and upload limits over IPv6 via MAC-keyed rules ([#70](https://github.com/YusDyr/luci-app-trafficctl/issues/70)) ([7f6f7e9](https://github.com/YusDyr/luci-app-trafficctl/commit/7f6f7e996ea67b90e8f597619939daf5acdf1f78))
  Reported in #67 with a clean reproduction: 9.96 Mbit/s over IPv4 against a
  10 Mbit/s limit, 151 Mbit/s over IPv6 to the same endpoint. The gap was
  systemic — no enforcement path in the package matched IPv6 at all — and the
  block case mattered most: a device reported as blocked had full, unmetered
  IPv6 access, which unlike a limit that under-delivers is invisible until it
  matters.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.18.1...v1.18.2

---

## [1.18.1] - 2026-09-29

### Bug Fixes
- Block WiFi reported success while the device stayed online ([#69](https://github.com/YusDyr/luci-app-trafficctl/issues/69)) ([30137d0](https://github.com/YusDyr/luci-app-trafficctl/commit/30137d0031d60cb2820b8a1976c8ec2222dcda2e))
  Blocking a device's WiFi from the dashboard wrote the uci maclist, reported
  success, and left the device online. Three defects behind one symptom, all
  found on a live router:

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.18.0...v1.18.1

---

## [1.18.0] - 2026-09-27

### Features
- cut all internet access while keeping the LAN working ([#55](https://github.com/YusDyr/luci-app-trafficctl/issues/55)) ([#59](https://github.com/YusDyr/luci-app-trafficctl/issues/59)) ([9a95c61](https://github.com/YusDyr/luci-app-trafficctl/commit/9a95c61b5f8cc29c4b6de3faabae1ac4f256e7f4))
  A single control that cuts internet access for every device while LAN keeps
  working. Off by default, indefinite until switched off, and the engaged state
  lives in tmpfs so a reboot always restores the internet; 'keep after reboot' is
  its own opt-in rather than the global persist_rules flag, so nobody inherits a
  persistent lockout from an unrelated decision.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.17.0...v1.18.0

---

## [1.17.0] - 2026-09-25

### Features
- wider Poll/Window choices and router-wide defaults for them ([#63](https://github.com/YusDyr/luci-app-trafficctl/issues/63)) ([b32b8d3](https://github.com/YusDyr/luci-app-trafficctl/commit/b32b8d3045abf42e233a4c1b330469664e98fb2d))
  The Poll chips offered Off/1/2/5s and Window 5/15/30/60s, both hard-coded, so
  there was no way to ask for the slower polling the request was about: on a
  smaller router a 1s poll is a full conntrack read every second for numbers
  nobody is watching that closely. Poll gains 10s and 30s, Window gains 2m and 5m.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.16.1...v1.17.0

---

## [1.16.1] - 2026-09-24

### Bug Fixes
- released packages left rpcd unaware of the plugin ([#62](https://github.com/YusDyr/luci-app-trafficctl/issues/62)) ([f55906a](https://github.com/YusDyr/luci-app-trafficctl/commit/f55906a12c4434348d15900bfe8642f199ac3bf6))
  Installing a published package and opening the dashboard gave "RPC call to
  luci.trafficctl/summary failed with error -32000: Object not found" until the
  router was rebooted. rpcd only scans for plugins when it starts, and the
  postinst never restarted it.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.16.0...v1.16.1

---

## [1.16.0] - 2026-09-19

### Features
- cumulative Bytes/TCP/UDP columns ([#26](https://github.com/YusDyr/luci-app-trafficctl/issues/26)) ([#60](https://github.com/YusDyr/luci-app-trafficctl/issues/60)) ([d26cc2c](https://github.com/YusDyr/luci-app-trafficctl/commit/d26cc2c637fad9e3d22f958a77efec4795c8a060))
  The Bytes / TCP / UDP columns came straight from live conntrack, so they showed what CURRENTLY TRACKED flows had carried and collapsed the moment those flows aged out — the "2 bytes" in the report. They are now accumulated from deltas and persisted.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.15.0...v1.16.0

---

## [1.15.0] - 2026-09-19

### Features
- global bmon-style traffic overview ([#26](https://github.com/YusDyr/luci-app-trafficctl/issues/26) item 7) ([#58](https://github.com/YusDyr/luci-app-trafficctl/issues/58)) ([77bb925](https://github.com/YusDyr/luci-app-trafficctl/commit/77bb925d988d72a494d88331ff82bf58f8fdd11c))
  Adds a global overview panel above the device table: an uplink throughput graph, role-badged per-interface rows with sparklines on a shared scale, and top talkers read from the speed map pollBytes() already maintains. No new collection — kernel counters plus data already on the page — and no timer of its own, so it inherits the Poll chip, document.hidden and the existing teardown.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.14.0...v1.15.0

---

## [1.14.0] - 2026-09-18

### Features
- optional default limit for devices seen for the first time ([#57](https://github.com/YusDyr/luci-app-trafficctl/issues/57)) ([12517f7](https://github.com/YusDyr/luci-app-trafficctl/commit/12517f70770ebba15d462896095847e5515d191c))
  Closes the second half of #28. A new Settings section ("New Device
  Defaults") applies a rate limit or a shape the first time a device
  appears on the network. Off by default, and a rate of 0 makes it inert.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.13.6...v1.14.0

---

## [1.13.6] - 2026-09-18

### Bug Fixes
- byte counters truncated at 2 GiB by awk's 32-bit %d ([#56](https://github.com/YusDyr/luci-app-trafficctl/issues/56)) ([2a21938](https://github.com/YusDyr/luci-app-trafficctl/commit/2a219386ede2f733b1c4a8a5e156e268c6424c5c))
  busybox awk formats %d through a 32-bit int, and what it does with a larger
  value is undefined: OpenWrt's build saturates at 2147483647, Alpine's wraps to
  -2147483648. Either way a device past 2 GiB stops reporting real numbers, and
  on the saturating builds it does so silently -- the counter just stops moving.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.13.5...v1.13.6

---

## [1.13.5] - 2026-09-18

### Bug Fixes
- nft byte counters reported download and upload swapped ([#54](https://github.com/YusDyr/luci-app-trafficctl/issues/54)) ([2a2aa86](https://github.com/YusDyr/luci-app-trafficctl/commit/2a2aa8618b7bb28a4fab5e4381b3b59abdb33d98))
  The nft counter backend keyed bytes_in by `ip saddr` and bytes_out by
  `ip daddr`. In the forward hook, packets whose source is a LAN device are
  that device's UPLOAD — so bytes_in accumulated upload and bytes_out
  accumulated download, the exact opposite of every consumer:

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.13.4...v1.13.5

---

## [1.13.4] - 2026-09-09

### Bug Fixes
- stop release notes hanging on a commit that cites an issue ([613f5c3](https://github.com/YusDyr/luci-app-trafficctl/commit/613f5c34d63ff1221a7b4fc6a8b5a7952f69672c))
  The auto-linking loop in the release-notes generator rewrites `subj` in
  place and re-matches from the start:

### Other
- tell "snapshot feeds are down" apart from "our package is broken" ([895a09f](https://github.com/YusDyr/luci-app-trafficctl/commit/895a09f236d72d3f4869ad49b00fb37bb778a917))
  The snapshot/x86-64 job has failed on every open PR for two days, blocking
  #48, #49 and #50 — none of which could affect it (two touched only README,
  one only an awk expression in a workflow). Inside the container apk fails
  with "wget: exited with error 8" then "unable to select packages": OpenWrt's
  rolling snapshot feeds are not serving. The same job passed on main on
  2026-09-04, the image tag resolves fine, and reruns a day apart did not help.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.13.3...v1.13.4

---

## [1.13.3] - 2026-08-25

### Bug Fixes
- pass the section regex to awk through the environment ([53c7d8e](https://github.com/YusDyr/luci-app-trafficctl/commit/53c7d8e5044f00428df45ef9a25ef4a0279a1d79))
  The release-notes generator matched commits with a pattern built as
  "^${type_regex}(\\([^)]*\\))?:" and handed it to awk via -v. awk expands
  escape sequences in a -v value, and \( is not one it recognises, so it stored
  a plain ( and warned about both parens. The optional scope group
  (\(...\))? thereby became a mandatory literal one, inverting the match in
  both directions: every fix(scope): subject stopped matching, and fixup: and

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.13.2...v1.13.3

---

## [Unreleased]

### Bug Fixes

- **Release notes silently dropped every scoped commit.** `format_section()` passed its match pattern to awk with `-v`, where awk expands escape sequences in the value: `\(` became a plain `(`, turning the optional scope group into a mandatory literal one. Scoped types like `fix(shaper):` then matched nothing while `fixup:`/`fixes:` matched, so v1.13.2 shipped with no Bug Fixes section at all despite seven `fix(scope):` commits. The pattern now reaches awk through the environment, which is passed through verbatim. Only gawk is affected — the GitHub runners symlink `awk` to it — so the fault is invisible on a machine using mawk or BSD awk.

---

## [1.13.2] - 2026-08-25

The release notes published for this version were incomplete: their Bug Fixes,
Security, Documentation and Tests sections were lost to the generator bug
recorded under Unreleased above. The full contents of the release follow.

### Security

- **The activity-log path could be pointed at any file.** `log_file` was written to UCI unvalidated and then used both as an append target and as a `tail`+`mv` rotation target, so a caller with the write ACL could read any root-owned file back through `activity_log` (which was in the **read** ACL) or empty one — `max_lines=1` rounded the rotation keep-count to zero. The path is now confined to `/tmp/trafficctl/` or `/var/log/`, `max_lines` to 20–100000, and `activity_log` is a write method.
- **Private signing keys were exposed to a mutable third-party action.** All three workflows passed `APK_PRIVATE_KEY`/`USIGN_PRIVATE_KEY` into `openwrt/gh-action-sdk@main`; the action is now pinned to a commit SHA.
- The Telegram bot token no longer appears in a command line (visible via `ps` to any local account) — the API URL is fed to curl on stdin, and `telegram_test` receives the token through the environment.
- `/etc/config/trafficctl` holds the bot token and the metrics token and is now kept at mode 0600 after every commit, not only after saving Telegram settings.
- `workflow_dispatch` inputs are no longer interpolated into `run:` blocks.

### Bug Fixes

- **Shaping a device could tear down every other device's shape.** The HTB classid was derived from the last two octets of the address, so `192.168.0.1` mapped onto the reserved root class `1:1` and shaping it deleted the root; `x.x.255.254` mapped onto the default class; and addresses from different subnets sharing their last two octets collided. Minors are now allocated and persisted, with pre-upgrade shapes still removable.
- **The shaper destroyed a pre-existing QoS setup.** The first shape unconditionally replaced the root qdisc, wiping an SQM/cake configuration. A root qdisc that is not a recognised default is now left alone and shaping declines instead.
- **Unblocking one device could unblock another.** Rule removal matched the comment as a substring, so `192.168.1.1` also matched `192.168.1.10`; the iptables state check had the same flaw and made `block` report "already blocked" without installing a rule. Comments are now matched in full and addresses as whole fields.
- **Controls created in LuCI could not be removed from the Telegram bot** (and vice versa) while still reporting success: the rule comment was built from the caller's label. Comments are now derived from the target.
- **Paused port ranges were only partly paused.** On the iptables path a range was truncated to its low port — `8000-8100` blocked only 8000 — while the UI showed the whole range as paused. Ranges are now passed as `lo:hi`.
- Concurrent shape writes no longer lose an entry: the lock is an atomic `mkdir` rather than a test-then-create file, and a waiter no longer deletes a lock still held by another writer.
- **Device aliases and shaping rules are no longer lost on a firmware upgrade** — `/etc/trafficctl` is now listed in `/lib/upgrade/keep.d/`, which default sysupgrade otherwise skips.
- Undeclared runtime dependencies (`tc`, `iw`, `hostapd_cli`) are now declared, so shaping and WiFi deauthentication no longer fail silently on a clean install.
- Hotplug scripts now ship executable.
- Frontend: the graph popup and its 2-second timer are removed on view teardown instead of accumulating one per visit; per-device speed history is capped; activity-log lines are rendered as text; SVG gradient IDs are unique per graph; a failed WiFi block no longer leaves the button permanently disabled.
- Releases: a scoped breaking change (`feat(scope)!:`) and a `BREAKING CHANGE:` footer are now detected, `refactor:`/`ci:` bump the patch version as documented, the tag is created after the rebase so it cannot be orphaned, the release is gated on the compatibility matrix, and a known-broken unsigned `.apk` is no longer published.
- **A commit body merely mentioning `BREAKING CHANGE` no longer forces a major release.** The footer search was unanchored, so prose describing the footer counted as one; this branch's own history would have published v2.0.0 from `fix:`/`ci:`/`docs:`/`test:` commits. The footer is now recognised only at the start of a line followed by `:` or ` #`.

### Documentation

- `docs/API.md` documents all 31 rpcd methods (13 were missing) with their ACL level; added `CONTRIBUTING.md`, issue and PR templates.

### Tests

- Tests that redefined local copies of the functions they claimed to check now exercise the real scripts, and the previously untested rpcd backend, block/unblock, macfilter and metrics CGI are covered. Each bug above has a regression test verified to fail against the old behaviour.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.13.1...v1.13.2

---

## [1.13.1] - 2026-08-24

### Bug Fixes
- split speed into separate sortable DL and UL columns ([ecea06b](https://github.com/YusDyr/luci-app-trafficctl/commit/ecea06bc89420a457a56ad75960ab227c6032887))
  After #27 merged, upload speed was shown twice: the DL Speed cell rendered a
  combined "↓ x / ↑ y" pair, and the separate UL Speed column added in #34
  rendered it again. Beyond the duplication, packing both into one cell means
  neither direction can be sorted on its own — the column sorts by _speed only.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.13.0...v1.13.1

---

## [1.13.0] — 2026-08-24

### Features

- **Bidirectional rate limiting and shaping** — limits and shapers previously only affected download. Upload is now shaped via an IFB device fed from LAN-side ingress (a `match ip src` filter on the WAN side can't work post-NAT), and partial application is reported honestly instead of silently half-working. ([#27](https://github.com/YusDyr/luci-app-trafficctl/pull/27))
- **Port Forwards tab** — view and manage DNAT rules from the app. ([#27](https://github.com/YusDyr/luci-app-trafficctl/pull/27))
- **Routed-subnet monitoring** and improved device naming. ([#27](https://github.com/YusDyr/luci-app-trafficctl/pull/27))
- **Optional netifyd DPI labels** — per-device application names when the Netify agent is installed. Entirely optional: not a package dependency, and inert when the agent or its socket is absent. ([#27](https://github.com/YusDyr/luci-app-trafficctl/pull/27))
- **Subnet and whole-network limits** — rate-limit an entire subnet or the whole LAN from the UI, either per-device or as an aggregate. ([#27](https://github.com/YusDyr/luci-app-trafficctl/pull/27))
- **Download and upload speed shown separately per device**, each in its own sortable column.
- **Prometheus metrics endpoint** — `/cgi-bin/trafficctl-metrics`, disabled by default, with an optional shared token. ([#27](https://github.com/YusDyr/luci-app-trafficctl/pull/27))

### Bug Fixes

- Per-device speed monitoring no longer stops on kernels where dynamic nftables counter maps are unsupported.
- The custom rate field no longer closes itself a few seconds after being opened.
- Netify telemetry is read from the socket sink rather than the agent API socket, and the sample window now covers the aggregator's report interval.
- Client-controlled device names are escaped in the summary JSON.

Thanks to [@adeelahmad](https://github.com/adeelahmad) for this contribution.

---

## [1.12.1] — 2026-08-24

### Bug Fixes

- **UI controls readable on non-Bootstrap themes** — around 14 declarations bypassed the theme palette with literal colours. Tooltips were painted dark-on-white unconditionally (unreadable once the page went dark), text on accent-filled chips and buttons was hardcoded white (invisible on themes with a pale accent), the toggle knob was always white, and every shadow used a fixed black tuned for light backgrounds. All colour literals now live in the `:root`/dark blocks. ([#30](https://github.com/YusDyr/luci-app-trafficctl/issues/30))

### Internal

- Config is read via `config_load` / `config_get` with defaults instead of repeated `uci -q get` calls — the Telegram bot's `load_config()` alone dropped from 12 forks to 1, and it runs every 60s. Suggested by [@stangri](https://github.com/stangri). ([#29](https://github.com/YusDyr/luci-app-trafficctl/issues/29))

---

## [1.12.0] — 2026-08-24

### Features

- **Back/forward navigates between devices** — selecting a device now pushes a history entry, so mobile back gestures and mouse back buttons work. Option tweaks (columns, filters, intervals) still replace the entry, so the history stack doesn't fill with chip clicks. ([#26](https://github.com/YusDyr/luci-app-trafficctl/issues/26))

---

## [1.11.0] — 2026-08-24

### Features

- **WiFi allow-mode (whitelist) ACLs are supported** — the package now adapts to whichever `macfilter` policy each radio uses, instead of assuming a blacklist. ([#31](https://github.com/YusDyr/luci-app-trafficctl/issues/31))

### Security

- **Blocking a device no longer inverts a whitelist.** Blocking used to force `macfilter=deny` on every wifi-iface. On a router configured with `macfilter=allow`, that silently reinterpreted the administrator's curated allow-list as a block-list — letting in every device the whitelist existed to exclude, and banning every device it listed. The configured policy is now respected and never overwritten: on a deny radio blocking adds the MAC, on an allow radio it removes it.

---

## [1.10.1] — 2026-08-24

### Bug Fixes

- **The device view no longer freezes when a device is selected** — selecting a device stopped all polling, so speed, drop and backlog figures and the connection table stayed frozen until a manual refresh. This also made the per-device speed graph dead code, since the only thing driving it was skipped in that mode. ([#26](https://github.com/YusDyr/luci-app-trafficctl/issues/26))

---

## [1.10.0] — 2026-08-24

### Features

- **Per-device upload speed column** — upload was already being computed from `bytes_out` but never surfaced. ([#32](https://github.com/YusDyr/luci-app-trafficctl/issues/32))

### Bug Fixes

- **Live table updates work again** — six selectors still queried `td[data-…]` even though the tables became `<div class="td">` in the 1.6 LuCI-native conversion, so they matched nothing. Between full table rebuilds the speed, sparkline, drop-counter and backlog cells never refreshed, and clicking a sparkline never opened the speed-graph popup. ([#26](https://github.com/YusDyr/luci-app-trafficctl/issues/26))

---

## [1.9.0] — 2026-08-15

### Features

- **Egress interface per connection** — the connections table can show which WAN a connection actually uses, resolved from the conntrack fwmark via `ip route get <dst> mark <mark>`, so it honours mwan3's policy routing. Only populated where the router restores the connmark; left blank otherwise rather than showing a misleading main-table answer. Optional column, hidden by default. Thanks to [@the-e3n](https://github.com/the-e3n) for the diagnostics. ([#10](https://github.com/YusDyr/luci-app-trafficctl/issues/10))

---

## [1.8.1] — 2026-08-15

### Bug Fixes

- **Telegram bot responds to commands again on OpenWrt 25.12 / APK and snapshot** — newer `jsonfilter` returns an empty string for `jsonfilter -l '@.result'`, which left the update counter empty and made BusyBox ash spam `sh: out of range` while never processing `/start`, `/help`, `/devices` or any inline button. Reported and root-caused by [@lavatti](https://github.com/lavatti). ([#22](https://github.com/YusDyr/luci-app-trafficctl/issues/22))

### CI

- The install and upgrade tests now exercise a real `opkg install` instead of masking its failure with a manual tar extract — they had been validating tar extraction, not opkg.
- The compat matrix is trimmed to x86-64. The other architectures were never actually running: OpenWrt publishes its rootfs container images for `linux/amd64` only, so those jobs failed `docker pull` and green-skipped. Since the package is `Architecture: all`, version coverage is what matters.
- ESLint and ShellCheck now fail on warnings, and ShellCheck actually scans every script (its file list had been silently matching nothing for the init.d, rpcd and hotplug scripts).
- `feature-build.yml` had a YAML error that made it fail on every run; fixed.
- `docker pull` is retried, so a transient ghcr.io blip no longer reds the build.

---

## [1.8.0] — 2026-06-21

### Features

- **Devices on secondary bridges and VLANs are discovered** — device discovery enumerates every interface in the non-WAN firewall zones instead of assuming a single LAN. VPN/tunnel zones are excluded, so WireGuard/AmneziaWG peers aren't mistaken for LAN clients. ([#13](https://github.com/YusDyr/luci-app-trafficctl/issues/13))

---

## [1.7.1] — 2026-06-21

### Performance

- **The dashboard no longer re-scans firewall and conntrack state per device** — the summary rebuilt everything inside the per-device loop, re-reading all of `/proc/net/nf_conntrack` and re-dumping nft chains and tc classes for every client. With many devices that was dozens of heavy forks every few seconds, reported as the UI heavily loading the router. All shared state is now fetched once.
- The Telegram bot scans for new devices periodically rather than on every poll iteration.

### Bug Fixes

- Temp files are removed via EXIT traps, so a script killed mid-run (which is exactly what rpcd does on a loaded router) no longer leaves scratch files filling tmpfs.

---

## [1.7.0] — 2026-06-04

### Features

- **Flow-offload awareness** — the settings panel shows the current offload mode with SW/HW toggles and explains the trade-offs. With hardware offload active the kernel stops syncing byte counts back to conntrack, so speed monitoring reads near zero; the UI now warns about this instead of showing frozen values.
- **Batch reverse DNS** — all uncached addresses are resolved in one `network.rrdns.lookup` round-trip, removing the stuck "resolving…" state.
- **Per-device speed graph** below the extended stats panel.

### Bug Fixes

- Dropped the `bind-dig` dependency: reverse DNS uses the built-in `rpcd-mod-rrdns` with a BusyBox `nslookup` fallback.
- `build-ipk.sh` uses `--format ustar`, preventing macOS PaxHeader entries from corrupting installs on BusyBox tar.

---

## [1.6.6] — 2026-05-29

### Bug Fixes

- **Fix feed-based install** — `./scripts/feeds install -p trafficctl luci-app-trafficctl` was failing with `target pattern contains no '%'` because OpenWrt's `find -L … -mindepth 1` skipped the repository-root Makefile. The repository is now laid out with the package source in a `luci-app-trafficctl/` subdirectory, which is what OpenWrt's feed scanner expects. ([#7](https://github.com/YusDyr/luci-app-trafficctl/issues/7))
- **Fix `/etc/config/trafficctl` clobbering on `opkg --force-reinstall`** — runtime state files (`shapes.json`, `telegram_known.json`) were listed as conffiles, which made opkg refuse to install on a fresh device. Conffiles now contain only `/etc/config/trafficctl`.
- **Stop renaming `/etc/trafficmon/` to `/etc/trafficctl/`** — the previous migration code could collide with other `trafficmon`-named packages on the same router. Postinst no longer touches the old directory; existing installations should migrate manually if needed.

### CI

- New OpenWrt SDK feed-install regression test (3 SDK versions) reproduces the user-facing path that issue #7 was about.
- New upgrade test (×2 SDK versions) installs the previously-released package, marks the config, installs the new build, and asserts the marker survives.
- New dependency test (×3 versions) verifies that missing deps fail cleanly and that `opkg update` / `apk update` resolves them.
- APK signing migrated from RSA to NIST P-256 (EC) keys — matches what `apk-tools v3` actually requires.
- snapshot/x86-64 compat job now tolerates upstream `rpcd-mod-luci` / `rpcd-mod-ucode` post-install hook noise that doesn't affect our package.

### Installation

- Same install commands as v1.5.0+ — see the v1.5.0 entry below.

---

## [1.6.5] - 2026-05-28

### Bug Fixes
- move IPK preservation into build step itself ([ee93b8b](https://github.com/YusDyr/luci-app-trafficctl/commit/ee93b8b0d113ef9c7772f3fd261b78e7667511e7))
  Separate step failed despite build-ipk succeeding (possible workspace
  isolation between steps). Move cp to /tmp/ into the same run block.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.6.4...v1.6.5

---

## [1.6.4] - 2026-05-28

> **Broken artifacts.** This release had install issues (missing `status.css`, invalid APK format, or a signing-key mismatch) and its assets have been deleted from GitHub. Use v1.6.5 or later. The entry is kept for history.

### Bug Fixes
- preserve IPK files across SDK Docker step ([fdfbc55](https://github.com/YusDyr/luci-app-trafficctl/commit/fdfbc55bb4d5fcdfde17c103675222da93fbe504))
  SDK action may wipe the workspace dist/ directory. Save IPK files
  to /tmp/ before SDK runs, restore them when collecting APK.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.6.3...v1.6.4

---

## [1.6.3] - 2026-05-28

> **Broken artifacts.** This release had install issues (missing `status.css`, invalid APK format, or a signing-key mismatch) and its assets have been deleted from GitHub. Use v1.6.5 or later. The entry is kept for history.

### Bug Fixes
- mkdir dist before collecting APK in manual-release ([4853bac](https://github.com/YusDyr/luci-app-trafficctl/commit/4853bac041969a51c662501e462bb189a4b4fc98))
  SDK action runs in Docker and may remove the dist/ directory
  created by the earlier ipk build step. Ensure it exists.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.6.2...v1.6.3

---

## [1.6.2] - 2026-05-28

> **Broken artifacts.** This release had install issues (missing `status.css`, invalid APK format, or a signing-key mismatch) and its assets have been deleted from GitHub. Use v1.6.5 or later. The entry is kept for history.

### Bug Fixes
- remove duplicate luci EXTRA_FEEDS from manual-release ([0b2653b](https://github.com/YusDyr/luci-app-trafficctl/commit/0b2653bdbe6b2f250355272e8ebf3b44b1198d30))
  SDK 25.12.4 already includes luci in its default feeds.conf.
  Adding it again via EXTRA_FEEDS causes "Duplicate feed name 'luci'"
  error and exits with code 25 during feeds update.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.6.1...v1.6.2

---

## [1.6.1] - 2026-05-28

> **Broken artifacts.** This release had install issues (missing `status.css`, invalid APK format, or a signing-key mismatch) and its assets have been deleted from GitHub. Use v1.6.5 or later. The entry is kept for history.

### Bug Fixes
- remove NO_DEFAULT_FEEDS from manual-release on main ([4212fdb](https://github.com/YusDyr/luci-app-trafficctl/commit/4212fdbf5077f4a00dfd39aa6a655699224ff69c))
  workflow_dispatch always uses the YAML from the default branch,
  regardless of the ref input. NO_DEFAULT_FEEDS: 1 excluded the
  packages feed (lua.h headers), breaking SDK builds of luci deps.

### Other
- V=sc verbose for SDK in manual-release + asset hardlinks ([d96aa51](https://github.com/YusDyr/luci-app-trafficctl/commit/d96aa519c517d0591f48c85ebaa5c56316600e8c))
  Brings the verbose-SDK and generic-named asset hardlink logic from
  fix/install-and-feed-testing-v2 to main so workflow_dispatch can run
  with the improvements while pull_request workflows are stuck.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.6.0...v1.6.1

---

## [1.6.0] - 2026-05-28

> **Tagged but never released.** The v1.6.0 tag exists in git, but no GitHub Release was ever published for it and no artifacts were built.

### Features
- add manual-release workflow for rebuilding existing releases ([172fbc1](https://github.com/YusDyr/luci-app-trafficctl/commit/172fbc1cbb6d6167bc251b01ece9b6a759bfc938))
  Cherry-picked early from #8 so we can rebuild v1.5.0's broken
  artifacts without waiting for the full PR to merge — workflow_dispatch
  requires the workflow file to exist on the default branch.

### Other
- remove generic-named asset symlinks from releases ([69924c4](https://github.com/YusDyr/luci-app-trafficctl/commit/69924c4dc610cbe54ad89488edad7998e189d076))
  Only publish versioned filenames (e.g. _1.5.0-1_all.ipk).
  GitHub's /releases/latest/download/ already resolves the latest
  release, making _latest_ and unversioned copies redundant.

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.5.0...v1.6.0

---

## [1.5.0] — 2026-05-27

### Features

- **Auto-detect software flow offload** ([#5](https://github.com/YusDyr/luci-app-trafficctl/issues/5)) — the realtime monitor now detects whether the router is running OpenWrt's software flow offload and switches its measurement strategy accordingly:
  - **No offload** — conntrack byte counters are accurate; we read them.
  - **Offload active** — conntrack stops accounting for fast-path packets after the flow is offloaded. We instead read an nftables counter map attached at `forward priority -200` (before the offload hook), which captures every packet.
  - The choice is re-evaluated on each refresh, so toggling flow offload in OpenWrt doesn't break the speed graph.

### Installation

- `opkg install` (OpenWrt 21.02 – 24.10):
  ```sh
  opkg install https://github.com/YusDyr/luci-app-trafficctl/releases/latest/download/luci-app-trafficctl.ipk
  ```
- `apk add` (OpenWrt 25.12+):
  ```sh
  apk add --allow-untrusted https://github.com/YusDyr/luci-app-trafficctl/releases/latest/download/luci-app-trafficctl.apk
  ```
- LuCI web UI: **System → Software → Upload Package**.

---

## [1.4.0] — 2026-05-27

### Features

- **APK package format for OpenWrt 25.12+** — releases now ship both `.ipk` (21.02 – 24.10) and `.apk` (25.12+, apk-tools v3) variants. APKs are built via the OpenWrt SDK so the resulting file uses the real APKv3 format (`ADBd` magic), not a fallback APKv2 archive that `apk-tools v3` refuses.
- **Signed packages** — IPKs are signed with usign, APKs with a NIST P-256 EC key. Public keys live in `keys/`. Signatures are verified by `apk add` automatically and by `opkg` when `option check_signature` is set.
- **Telegram bot test infrastructure** — added mock + integration + end-to-end test suites for the Telegram bot under `tests/`. All run on every PR.

### Bug Fixes

- **Don't shadow `awk`'s reserved word `load`** — variable rename in `trafficctl-summary.sh` keeps gawk happy on devices that use it instead of busybox awk.
- Several CI debug-output and portability fixes for the Telegram E2E test runner.

### CI

- **Full compatibility matrix** — 52 combinations spanning OpenWrt 21.02 / 22.03 / 23.05 / 24.10.1 / 24.10.6 / 25.12.0 / 25.12.4 / snapshot × x86-64 / x86-generic / armsr / arm_a9 / arm_a15 / armvirt32 / mips_24kc / aarch64_cortex-a53.
- Releases are now produced only by `feat:` / `fix:` / `perf:` commits — `ci:`, `refactor:`, `docs:` no longer trigger a version bump.
- APK builds via `openwrt/gh-action-sdk` instead of a hand-rolled apk-tools wrapper.

---

## [1.3.1] - 2026-05-26

> **Broken artifacts.** This release had install issues (missing `status.css`, invalid APK format, or a signing-key mismatch) and its assets have been deleted from GitHub. Use v1.6.5 or later. The entry is kept for history.

### Other
- replace release-please with auto-release on every merge ([52ffcf4](https://github.com/YusDyr/luci-app-trafficctl/commit/52ffcf4e0d190f550e31423b5cef21751c767183))
  - Remove release-please workflow, config, and manifest
  - Add auto-release.yml: on push to main, detect version bump from
  conventional commits, create tag + GitHub Release + build IPK
  - No manual steps needed — merge feat:/fix: to main = instant release
  - Keep workflow_dispatch as manual fallback
  - Update CLAUDE.md release documentation

**Full Changelog**: https://github.com/YusDyr/luci-app-trafficctl/compare/v1.3.0...v1.3.1

---

## [1.3.0](https://github.com/YusDyr/luci-app-trafficctl/compare/v1.2.1...v1.3.0) (2026-05-26)


### Features

* redesign Telegram Bot settings with mode toggle, live preview, and template variables ([#2](https://github.com/YusDyr/luci-app-trafficctl/issues/2)) ([8d49873](https://github.com/YusDyr/luci-app-trafficctl/commit/8d498737de53db551648f254d005a1ecf0b5d4bc))


### CI

* add release-please for automated changelog and releases ([f1c9834](https://github.com/YusDyr/luci-app-trafficctl/commit/f1c9834b37a19ae091b5c6b993e1309a06c74709))

## [1.2.1] — 2026-05-26

### Bug Fixes

- **Fix broken IPK format** — package was built with Debian `ar` format instead of OpenWrt's gzip-tar format; `opkg` rejected it with `Malformed package file` on all devices ([#1](https://github.com/YusDyr/luci-app-trafficctl/issues/1))
- **Fix rpcd binary path** — binary was installed as `trafficctl` but rpcd expects `luci.trafficctl`
- **Fix ShellCheck SC2086** — unquoted variable in `uci` call in `trafficctl-fw.sh`
- **Fix ESLint no-redeclare** — duplicate `chipActiveStyle` declaration in `status.js`

### CI

- Per-test badges: ShellCheck, ESLint, Tests each have their own status badge
- Release is blocked from publishing if any test fails
- OpenWrt compatibility matrix: tested across 3 versions (21.02, 22.03, snapshot) × 4 architectures (x86-64, aarch64, arm\_a15, armvirt-32)
- All CI jobs moved to GitHub-hosted runners

### Installation

- Install directly on the router without `scp`:
  ```sh
  opkg install https://github.com/YusDyr/luci-app-trafficctl/releases/latest/download/luci-app-trafficctl.ipk
  ```
- Install via LuCI web UI — **System → Software → Upload Package**
- Stable download URLs: [`luci-app-trafficctl.ipk`](https://github.com/YusDyr/luci-app-trafficctl/releases/latest/download/luci-app-trafficctl.ipk), [`luci-app-trafficctl_all.ipk`](https://github.com/YusDyr/luci-app-trafficctl/releases/latest/download/luci-app-trafficctl_all.ipk), [`luci-app-trafficctl_latest_all.ipk`](https://github.com/YusDyr/luci-app-trafficctl/releases/latest/download/luci-app-trafficctl_latest_all.ipk)

---

## [1.2.0] — 2026-05-26

### New Features

- **Interactive speed graph popup** — hover any device's sparkline to see a full-size graph with download + upload dual lines, gradient fill, min/max band, rate limit overlay line, and an interactive crosshair showing precise values at any point in time. History starts from page load and is never lost.
- **Recent devices quick-access bar** — selecting a device (via table click or search) adds it to a chip bar below the search field. Up to 6 recent devices persist across page reloads (localStorage). One-click switching between frequently monitored devices.
- **Activity logging** — all mutable actions (blocks, rate limits, shapes, WiFi denials, config changes) are logged with timestamp, source IP, username, and trigger (LuCI / Telegram / CLI). Logs are viewable in the UI and optionally forwarded to syslog.
- **Reboot persistence for blocks & rate limits** — new `persist_rules` option in Settings. When enabled, internet blocks and rate limits are saved to `/etc/trafficmon/rules.json` and automatically restored on boot alongside traffic shaping rules.
- **New device detection** — instant notification when a new device joins the network. Detects via ARP, DHCP leases, and WiFi station list. DHCP hotplug trigger provides near-realtime alerts. Integrates with Telegram notifications.
- **Per-device column toggles** — show/hide individual table columns (MAC, Speed, Conns, etc.) from the Connections table settings section.
- **Settings panel collapsed by default** — cleaner look on page load; expand on demand.

### Improvements

- **WiFi blocking no longer restarts WiFi** — uses `hostapd_cli deny_acl` + `deauthenticate` to disconnect only the target client. Other WiFi clients stay connected with zero interruption.
- **Speed display in bits (not bytes)** — sparkline and graph values now show Kbit/s and Mbit/s as expected for network speeds. Clean labels: no trailing ".0" for whole numbers (e.g., "10 Mbit/s" not "10.0 Mbit/s").
- **Stable graph scale** — spike filter caps speed at 1 Gbit/s (link ceiling) to discard conntrack counter resets. Y-axis uses 98th percentile scaling so occasional spikes don't crush the useful range.
- **Nice Y-axis values** — graph ticks are multiples of 100 or 500 Kbit/s (or 1/5/10 Mbit/s for faster links) with at least 5 gridlines for readability.
- **Upload speed tracking** — graphs now show both download (solid blue) and upload (dashed green) simultaneously.
- **Compact table headers** — limiter, drop, and queue columns use icon-only headers to save horizontal space.
- **Sort by name** — device table can be sorted alphabetically by hostname.
- **Sparkline rate limit line** — a subtle horizontal line on each sparkline shows the active speed limit for that device.
- **Redesigned speed limit UI** — pill-style chip picker for rate presets + segmented toggle for shaper/limiter mode selection.

### Bug Fixes

- Fixed speed showing in bytes instead of bits.
- Fixed graph popup not showing rate limit line for shaped devices (fallback to summary data).
- Fixed initial page load sometimes showing blank table.
- Fixed WiFi capture disconnecting all clients during screenshot automation.
- Fixed rate limit removal failing to match by IP on some configurations.

---

## [1.1.0] — 2025-05-18

### New Features

- **Telegram bot** — remote control from your phone. Send `/devices` to see active devices with inline keyboard buttons for block, unblock, rate limit, shape, WiFi deny. Long polling — runs entirely on the router, no external server needed.
- **New device notifications** — Telegram alerts when an unknown device joins your network.
- **Bot configuration UI** — token, chat ID, notification toggles, and a "Test" button directly in LuCI Settings.

### Improvements

- CI pipeline with ShellCheck, ESLint, and automated tests.
- CodeQL security scanning enabled.
- System requirements documented (RAM, flash, CPU).

---

## [1.0.0] — 2025-05-10

Initial release.

- Real-time per-device traffic monitoring via conntrack.
- Internet blocking (nftables / iptables auto-detection).
- Rate limiting (nft policer with drop counters).
- Traffic shaping (tc/HTB with fq_codel, persistent across reboots).
- WiFi MAC filtering.
- Interface detection (2.4G / 5G / 6G / LAN port).
- Live speed sparklines with configurable poll interval.
- Reverse DNS lookup for destination IPs.
- Searchable device picker (command palette style).
- Dark / light theme support.
- OpenWrt 21.02–23.05 compatibility.
