# Xray-core Integration Design Document

## Overview

This document proposes a phased integration of **Xray-core** (`v26.10.31`) as an alternative core for **VLESS** and **Trojan** profiles in NekoBox+, alongside the existing sing-box core. The user will be able to choose per-protocol which core to use (sing-box vs Xray) from a new "Core" section in the main settings screen.

## Scope & Constraints

- **In scope (phase 1):** VLESS and Trojan protocols only.
- **Out of scope (phase 1):** VMess, Shadowsocks, Hysteria, and other protocols remain sing-box only.
- **Build target:** Android (arm64-v8a, armeabi-v7a, x86_64, i686) via gomobile.
- **UI:** New "Core" section in `GlobalSettingsScreen.kt` with per-protocol core selector.
- **Persistence:** DataStore key per protocol (`coreForVless`, `coreForTrojan`).

## Architecture Changes

### 1. Go / libcore Layer

#### New files
- `libcore/xray_instance.go` — implements `XrayInstance` wrapping `github.com/xtls/xray-core/core` + `app`.
- `libcore/xray_config.go` — converts sing-box-compatible config to Xray JSON config.

#### Existing files to modify
- `libcore/go.mod` — add `github.com/xtls/xray-core v1.8.x` (or v26.10.31 module path).
- `libcore/libcore.go` — add `NewXrayInstance(config, resolver)` constructor and `XrayInstance` interface.
- `libcore/build.sh` — add Xray version ldflag, update gomobile tags.

#### libcore API additions
```go
type XrayInstance interface {
    Start() error
    CloseTimeout(timeout int64) error
    Sleep()
    Wake()
    // ... mirror BoxInstance interface
}
func NewXrayInstance(config string, resolver DnsResolver) (XrayInstance, error)
```

### 2. Android Kotlin Layer

#### New files
- `bg/proto/XrayInstance.kt` — Kotlin wrapper mirroring `BoxInstance.kt`.
- `fmt/xray/XrayConfigBuilder.kt` — builds Xray JSON from `StandardV2RayBean`/`TrojanBean`.

#### Existing files to modify
- `bg/proto/ProxyInstance.kt` — select core based on `DataStore.coreForVless` / `coreForTrojan`.
- `bg/BaseService.kt` — instantiate correct core type.
- `database/DataStore.kt` — add `coreForVless`, `coreForTrojan` keys.

### 3. UI Layer

#### New section in GlobalSettingsScreen.kt
```kotlin
group("core", R.string.core_section_title) {
    list("coreForVless", R.drawable.ic_..., R.string.core_for_vless, 
         entries = arrayOf("sing-box", "xray"), values = arrayOf("singbox", "xray"))
    list("coreForTrojan", R.drawable.ic_..., R.string.core_for_trojan,
         entries = arrayOf("sing-box", "xray"), values = arrayOf("singbox", "xray"))
}
```

### 4. Build System

#### Workflow changes
- Add Xray-core clone step in `build-patched.yml` (similar to sing-box).
- Add Xray version env var.

#### go.mod changes
```go
require github.com/xtls/xray-core v1.8.24  // or v26.10.31
```

## Implementation Phases

### Phase 0: Preparation (1-2 days)
- [ ] Fork/clone Xray-core at v26.10.31
- [ ] Verify Xray builds for Android (gomobile bind)
- [ ] Study Xray's `core` and `app` packages for instance lifecycle

### Phase 1: Go libcore Xray wrapper (3-5 days)
- [ ] Create `libcore/xray_instance.go`
- [ ] Implement `Start()`, `Close()`, `Sleep()`, `Wake()`
- [ ] Add `NewXrayInstance()` to libcore API
- [ ] Update `go.mod`
- [ ] Test gomobile bind

### Phase 2: Config generation (2-3 days)
- [ ] Create `XrayConfigBuilder.kt`
- [ ] VLESS config: outbound with VLESS protocol, TLS/Reality, transport (ws/grpc/xhttp)
- [ ] Trojan config: outbound with Trojan protocol, TLS
- [ ] Inbound: tun/socks for Android VPN

### Phase 3: Android integration (2-3 days)
- [ ] Create `XrayInstance.kt` Kotlin wrapper
- [ ] Modify `ProxyInstance.kt` to select core
- [ ] Modify `BaseService.kt` instance creation
- [ ] Add DataStore keys

### Phase 4: UI (1 day)
- [ ] Add "Core" section to `GlobalSettingsScreen.kt`
- [ ] Add strings (en, ru)
- [ ] Add icons

### Phase 5: Testing & Polish (2-3 days)
- [ ] Build & test on AVD
- [ ] Test VLESS with Xray core
- [ ] Test Trojan with Xray core
- [ ] Test switching cores
- [ ] Fix any issues

## Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Xray-core API differs significantly from sing-box | Study Xray docs first; create adapter layer |
| gomobile bind fails with Xray deps | Test early in Phase 0 |
| Config format incompatibility | Create dedicated Xray config builder |
| Performance regression | Benchmark both cores |
| Binary size increase | Xray is larger; accept or use build tags |

## Dependencies

- Xray-core v26.10.31 (https://github.com/XTLS/Xray-core/releases/tag/v26.10.31)
- Go 1.27.1 (already in use)
- gomobile (already in use)

## Estimated Timeline

- **Total:** 10-14 days
- **Phase 0-1:** 4-7 days (Go layer)
- **Phase 2-3:** 4-6 days (Android layer)
- **Phase 4-5:** 3-4 days (UI + testing)

## Notes

- This is a significant undertaking. Consider starting with a proof-of-concept (PoC) that only supports VLESS over TCP with no advanced features.
- Xray-core's Go module path is `github.com/xtls/xray-core`. Check if it supports gomobile bind (some Xray packages may not be mobile-compatible).
- The existing sing-box integration in `libcore` provides a good template for the Xray wrapper.

---

*Created: 2026-11-02*
*Status: Draft — awaiting approval*
