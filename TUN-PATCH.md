# RustDesk 1.5.3 + TUN P2P patch

Unofficial build. Upstream RustDesk 1.5.3 with one change in its WebRTC library, so that WebRTC
connects peer-to-peer on machines running a TUN proxy (Shadowrocket, Clash/mihomo, sing-box,
Surge) instead of always falling back to TURN.

**The problem.** webrtc-rs gathers server reflexive (srflx) candidates only from a socket bound
to `0.0.0.0`. That socket follows the default route, which on such a machine is the TUN. The
proxy re-sends each flow from a socket of its own, so the mapping the STUN server reports is not
the one the connectivity checks use. The host looks symmetric even behind a cone NAT, and every
session ends up relayed. libwebrtc binds one socket per network interface and does not hit this.

**The change.** `webrtc-ice` also gathers srflx candidates from a socket bound to each interface
address, skipping 198.18.0.0/15 (TUN proxies' fake-IP range) and link-local. On macOS a socket
bound to an interface address is routed by that interface (scoped routing), so it bypasses the
TUN and reports the NAT's real mapping. See the `webrtc` branch: upstream `rustdesk-org/webrtc`
@ `49c89bd8` plus one commit.

Everything else is upstream: `main` is the 1.5.3 tree with the other CI workflows removed, the
webrtc crates redirected in `Cargo.toml`, and `.github/workflows/tun-build.yml`, which follows
upstream's macOS arm64 build without signing. The dmg it produces is unsigned.

**Limits.** Linux TUN proxies that route everything into the TUN with policy rules may not be
bypassed by binding a source address; that needs `SO_BINDTODEVICE`, not done here.

License: AGPL-3.0, as upstream.
