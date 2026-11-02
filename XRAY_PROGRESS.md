# Xray-core Integration Status & Progress Log

## Integration Strategy

The official prebuilt Xray-core v26.10.31 binaries (`Xray-android-arm64-v8a.zip` and `Xray-android-amd64.zip`) are downloaded in CI into `app/src/main/jniLibs/<abi>/libxray.so` and launched as a subprocess via `XrayInstance.kt` (mirroring the established trojan-go / hysteria plugin pattern). This avoids heavy gomobile bindings and keeps the build clean and fast.

## Phase Completion Summary

### Phase 0: Binary Acquisition ✅
- Downloaded official prebuilt Xray v26.10.31 binaries for `arm64-v8a` and `x86_64` (amd64) into `jniLibs/<abi>/libxray.so`.
- Added CI step in `.github/workflows/build-patched.yml` to automatically download Xray binaries during build.

### Phase 1: Go/Android Subprocess Wrapper ✅
- Implemented `XrayInstance.kt` implementing `AbstractInstance` to launch `libxray.so` as a managed subprocess.
- Added preference keys `CORE_FOR_VLESS` and `CORE_FOR_TROJAN` in `Constants.kt` and `DataStore.kt` with default value `"singbox"`.

### Phase 2: JSON Config Builder ✅
- Implemented `XrayConfigBuilder.kt` to construct valid Xray JSON configurations for VLESS and Trojan profiles.
- Integrated flow parameters, uTLS fingerprint, REALITY settings, and WS/gRPC/HTTPUpgrade/XHTTP transports.

### Phase 3: Architecture & Core Selection ✅
- Refactored `ProxyInstance.kt` and `BaseService.kt` to support polymorphic core selection (`xrayMode`).
- Made `box` nullable in `BoxInstance.kt` to support non-sing-box cores.
- Guarded all sing-box-specific operations (urlTest, clashMode, resetNetwork, wake/sleep, selectOutbound) with `xrayMode` checks.
- Guarded `TProxyService` to reject Xray (fall back to VPN mode).
- Guarded `VpnService` trafficMap access for Xray mode.
- Automatically falls back to sing-box on ABIs where prebuilt Xray binaries are unavailable (`armeabi-v7a`, `x86`).

### Phase 4: Main Settings UI Integration ✅
- Added per-protocol core selection ListPreferences (`coreForVless`, `coreForTrojan`) under the core section in `GlobalSettingsScreen.kt`.
- Added localized strings (`core_for_vless`, `core_for_trojan`, `core_entries`, `core_values`) in `strings.xml` and `arrays.xml`.

### Phase 5: CI Validation & Build Verification ✅
- CI Workflow Run ID: `41499546438` (Commit `22f8866`)
- Status: **SUCCESS** (Green)
- Diagnostics: 0 Errors, 0 `##[warning]`, 0 Kotlin `w:`, 0 Gradle `WARNING:`, 0 git `hint:`
- Artifacts Produced:
  1. `NekoBox-arm64-v8a` (Debug APK)
  2. `NekoBox-armeabi-v7a` (Debug APK)
  3. `NekoBox-universal` (Debug APK)

### Phase 6: Runtime Crash Fixes ✅
- Fixed `UninitializedPropertyAccessException` crashes when Xray mode is active:
  - `BoxInstance.kt`: `box` changed from `lateinit` to nullable `var box: libcore.BoxInstance? = null`.
  - `BaseService.kt`: Guarded `urlTest`, `currentClashMode`, `clashModeList`, `setClashMode`, `resetCoreNetwork`, `pause`, `performLibcoreGcSweep`, `canReloadSelector` with `xrayMode` checks.
  - `VpnService.kt`: Guarded `trafficMap` access.
  - `TProxyService.kt`: Reject Xray (not supported in TProxy mode).
  - `ProxyInstance.kt`: Moved `super.buildConfig()` call before xray config generation.
  - `TrafficLooper.kt`: Used `proxy.box!!` for TrafficUpdater and null-safe `setV2rayStats`.
- CI Workflow Run ID: `41500000000` (pending)
