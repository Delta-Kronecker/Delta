# Phase 0+1: Xray-core integration (pragmatic approach using prebuilt binaries)

## Approach Change

Instead of gomobile-binding Xray-core (heavy, complex, breaks build), we use the
official prebuilt Xray Android binaries (libxray.so per ABI) and launch Xray as
a subprocess, mirroring how trojan-go and hysteria plugins are launched.

This is simpler, faster, and avoids Go module dependency issues.

## Phase 0: Binary acquisition ✅
- Downloaded Xray v26.10.31 prebuilt binaries for arm64-v8a and x86_64
- Placed as jniLibs/<abi>/libxray.so

## Phase 1: Go/Android wrapper ✅
- Created XrayInstance.kt (subprocess-based, mirrors trojan-go pattern)
- Added DataStore keys: coreForVless, coreForTrojan
- Added Constants: CORE_FOR_VLESS, CORE_FOR_TROJAN

## Phase 2: Config generation ✅
- Created XrayConfigBuilder.kt (VLESS/Trojan → Xray JSON)

## Phase 3: Android integration ✅
- Modified ProxyInstance.kt to select core based on DataStore

## Phase 4: UI ✅
- Added "Core" section to GlobalSettingsScreen.kt

## Phase 5: Testing
- Pending CI build verification
