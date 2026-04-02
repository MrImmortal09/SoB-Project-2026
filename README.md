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
+-------------------------------------------------------------+
|                 fedimint-client-uniffi (Rust)               |
|   Wrapper around fedimint-client exposing APIs via          |
|   mozilla/uniffi-rs                                         |
+-------------------------------------------------------------+
|                   UniFFI Binding Generation                 |
+----------------------------+--------------------------------+
|       fedimint-android     |        fedimint-swift          |
|        (Kotlin / .aar)     |      (Swift / XCFramework)     |
|                            |                                |
|  • Gradle build            |  • SPM package                 |
|  • Maven Central           |  • GitHub releases             |
|  • Android NDK             |  • iOS + macOS + Simulator     |
|    cross-compile           |    cross-compile               |
+----------------------------+--------------------------------+

```
### Key reference: bdk-ffi structure

The `bdk-ffi` project demonstrates this exact pattern successfully:

| Component | bdk-ffi (Reference) | fedimint-sdk-ffi (Proposed) |
|-----------|---------------------|-------------------------|
| Android bindings | `bdk-android/` (.aar, Gradle, Maven Central) | `fedimint-android/`(inside fedimint-sdk-ffi itself) |
| Swift bindings | `bdk-swift/` (XCFramework, SPM) | `fedimint-swift/`(inside fedimint-sdk-ffi itself) |
| Binding generator | mozilla/uniffi-rs | mozilla/uniffi-rs(fedimint-client-uniffi) |
| Build tooling | `just` + shell scripts | Nix flake + `just` + shell scripts |

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
- **Android targets:** `aarch64-linux-android`, `x86_64-linux-android` via Android NDK
- **Apple targets:** `aarch64-apple-ios`, `aarch64-apple-ios-sim`, `x86_64-apple-ios`, `aarch64-apple-darwin`, `x86_64-apple-darwin`
- **CI/CD:** GitHub Actions for automated builds, tests, and artifact publishing
- **Testing:** Android instrumented tests (emulator), Swift XCTest, integration tests against local Fedimint federation

---
