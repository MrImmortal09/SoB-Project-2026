# Project: Native Android (Kotlin) & Swift Bindings for Fedimint SDK with Multi-Wallet Support

## Problem Statement

Today, the Fedimint SDK provides support for **web/browser** and **React Native** environments. However, developers building **native Android (Kotlin)** or **native iOS (Swift)** apps currently have no published, idiomatic SDK to work with. They must either:

1. Use the React Native bridge (adding unnecessary complexity for native apps), or
2. Manually integrate raw Rust FFI, which is error-prone and undocumented.

This limits Fedimint's reach in the mobile ecosystem, where native apps remain the dominant approach for production-grade Bitcoin/Lightning wallets.

Additionally, with the recently merged `fedimint-client-rpc` ([PR #7499](https://github.com/fedimint/fedimint/pull/7499)), the Fedimint client now supports **multiple wallet instances** — but this capability is not yet surfaced in the SDK.

---

## Proposed Solution

Build and publish **native Android (Kotlin) and Swift bindings** for the Fedimint client SDK, following the proven architecture used by [bitcoindevkit/bdk-ffi](https://github.com/bitcoindevkit/bdk-ffi):

### Architecture (Inspired by bdk-ffi)

```
┌─────────────────────────────────────────────┐
│              fedimint-ffi (Rust)             │
│  Wrapper around fedimint-client exposing     │
│  APIs via mozilla/uniffi-rs                  │
├─────────────────────────────────────────────┤
│          UniFFI Binding Generation           │
├──────────────────┬──────────────────────────┤
│  fedimint-android│     fedimint-swift        │
│  (Kotlin/.aar)   │   (Swift/XCFramework)     │
│                  │                           │
│  • Gradle build  │   • SPM package           │
│  • Maven Central │   • GitHub releases       │
│  • Android NDK   │   • iOS + macOS + Sim     │
│    cross-compile │     cross-compile         │
└──────────────────┴──────────────────────────┘
```

### Key reference: bdk-ffi structure

The `bdk-ffi` project demonstrates this exact pattern successfully:

| Component | bdk-ffi (Reference) | fedimint-ffi (Proposed) |
|-----------|---------------------|-------------------------|
| Rust FFI wrapper | `bdk-ffi/` | `fedimint-ffi/` |
| Android bindings | `bdk-android/` (.aar, Gradle, Maven Central) | `fedimint-android/` |
| Swift bindings | `bdk-swift/` (XCFramework, SPM) | `fedimint-swift/` |
| Binding generator | mozilla/uniffi-rs | mozilla/uniffi-rs |
| Build tooling | `just` + shell scripts | Nix flake + `just` + shell scripts |

---

## Deliverables

### Phase 1: Native Bindings Foundation (Weeks 1–5)
- [ ] Create `fedimint-ffi` Rust crate wrapping the Fedimint client for UniFFI export
- [ ] Build Android bindings (`fedimint-android`):
  - Cross-compile for `aarch64-linux-android`, `x86_64-linux-android`
  - Package as `.aar` with Gradle build
  - Publish to Maven Central (or snapshot repo)
- [ ] Build Swift bindings (`fedimint-swift`):
  - Cross-compile for iOS (aarch64), iOS Simulator (aarch64 + x86_64), macOS
  - Package as XCFramework
  - Publish via Swift Package Manager

### Phase 2: Multi-Wallet Support (Weeks 6–8)
- [ ] Expose `fedimint-client-rpc` multi-wallet capabilities through the native bindings
  - Create/manage multiple wallet instances per app
  - Independent federation joins per wallet
  - Wallet-scoped operations (send, receive, balance)
- [ ] Build example apps demonstrating multi-wallet on Android and iOS

### Phase 3: Documentation, Testing & Polish (Weeks 9–12)
- [ ] Comprehensive API documentation (KDoc for Kotlin, DocC for Swift)
- [ ] Integration test suites (Android instrumented tests, Swift XCTest)
- [ ] Example wallet apps for both platforms
- [ ] CI/CD pipelines for automated builds and releases
- [ ] Migration guide for developers moving from React Native to native

---

## Prior Art & Experience

### 1. `fedimint-client-rpc` ([PR #7499](https://github.com/fedimint/fedimint/pull/7499))
- **Status:** ✅ Merged (July 15, 2025)
- Introduced the RPC layer with `fedimint-cursed-redb` storage
- Enables multiple wallet instances — the backend foundation for multi-wallet mobile support
- 648 additions, 604 deletions across 7 files
- 38 review comments, thoroughly reviewed by maintainers

### 2. `@fedimint/react-native` ([PR #239](https://github.com/fedimint/fedimint-sdk/pull/239))
- **Status:** 🔄 Open / In Review
- Already uses UniFFI bindings to compile the Fedimint client to native libraries (iOS & Android)
- Implements transport layer, device persistence (`dbPath`), and TypeScript interface
- 19,202 additions across 122 files — substantial cross-platform build infrastructure
- Much of the UniFFI and cross-compilation work here is **directly reusable** for standalone native bindings

### 3. Reference: `bitcoindevkit/bdk-ffi`
- Mature, production-grade example of the exact pattern proposed
- Supports Kotlin (Android), Swift (iOS/macOS), with additional JVM and Python bindings
- 118 stars, actively maintained, published on Maven Central and SPM
- Proves the UniFFI-based approach works well for Bitcoin ecosystem Rust libraries

---

## Why This Project Matters

1. **Native-first mobile apps** remain the standard for production Bitcoin wallets (e.g., Mutiny, Zeus, BlueWallet all ship native)
2. **Multi-wallet support** is a differentiator — users want to manage multiple federations from one app
3. **Fedimint needs parity** with projects like BDK that already offer polished native SDKs
4. **The groundwork is done** — UniFFI bindings, cross-compilation, and multi-wallet RPC are all built; this project packages and polishes them

---

## Technical Approach

- **UniFFI (mozilla/uniffi-rs):** Generate idiomatic Kotlin and Swift from annotated Rust code
- **Build system:** Extend existing Nix flake from fedimint-sdk with `just` task runner (following bdk-ffi conventions)
- **Android targets:** `aarch64-linux-android`, `armv7-linux-androideabi`, `x86_64-linux-android` via Android NDK
- **Apple targets:** `aarch64-apple-ios`, `aarch64-apple-ios-sim`, `x86_64-apple-ios`, `aarch64-apple-darwin`, `x86_64-apple-darwin`
- **CI/CD:** GitHub Actions for automated builds, tests, and artifact publishing
- **Testing:** Android instrumented tests (emulator), Swift XCTest, integration tests against local Fedimint federation

---

## Timeline

| Week | Milestone |
|------|-----------|
| 1–2 | Set up `fedimint-ffi` crate, UniFFI annotations, basic Kotlin/Swift generation |
| 3–4 | Android cross-compilation, `.aar` packaging, Gradle setup |
| 5 | Swift XCFramework, SPM packaging, iOS build scripts |
| 6–7 | Multi-wallet API surface in native bindings |
| 8 | Example apps (Android + iOS) with multi-wallet demo |
| 9–10 | Testing infrastructure, CI/CD pipelines |
| 11–12 | Documentation, polish, release prep |


## Author
**MrImmortal09** — Previous SoB intern with Fedimint  
- Shipped [`fedimint-client-rpc`](https://github.com/fedimint/fedimint/pull/7499) (merged July 2025), enabling multi-wallet support in the Fedimint client
- Built the [`@fedimint/react-native`](https://github.com/fedimint/fedimint-sdk/pull/239) package, bringing UniFFI-based native bindings to React Native

---
