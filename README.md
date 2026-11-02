<p align="center" style="background-colour: #CCCCCC;">
  <a href="https://www.reactvision.xyz/">
    <img src="https://avatars.githubusercontent.com/u/74572641?s=200&v=4" alt="ReactVision logo" width="120px" height="120px">
  </a>
</p>

<p align="center">
  <a href="https://discord.gg/yqqEGUjK">
    <img src="https://img.shields.io/discord/774471080713781259?label=Discord" alt="Discord">
  </a>
</p>

# ViroCore, By ReactVision

ViroCore is the high-performance 3D rendering engine behind [ViroReact](https://github.com/ReactVision/viro) — a cross-platform C++ renderer (OpenGL ES on Android, Metal on iOS/visionOS) with a descriptive Java API for building 3D, AR, and VR experiences. It handles scene rendering, PBR lighting, animation, physics, and AR integration so you can focus on the experience rather than low-level graphics.

ViroCore is MIT licensed and free forever.

> Most developers should build with **[`@reactvision/react-viro`](https://www.npmjs.com/package/@reactvision/react-viro)**, which wraps this engine behind a React Native API. This repository is the native renderer that ViroReact ships as a prebuilt library.

## Fork changes

This is a fork of [ReactVision/virocore](https://github.com/ReactVision/virocore). It tracks upstream and carries one behavioural change, plus an ARCore bump (both below). Branch `arcore-1.57` is based on upstream `v2.57.5`; `android/viroar` is unchanged between `v2.57.5` and `v2.58.1`.

### ARCore 1.57.0 (Android)

Upstream's Android build mixes ARCore versions (the vendored C library `libarcore_sdk_c.so` is 1.51 while the shipped Java client AAR is 1.43). This branch moves the ARCore C API header (`android/libs/arcore/include/arcore_c_api.h`, from `google-ar/arcore-android-sdk` `v1.57.0`), the vendored `android/viroar/src/main/jniLibs/*/libarcore_sdk_c.so` (from `com.google.ar:core:1.57.0`) and the `com.google.ar:core` dependency in `sharedCode`/`virocore`/`viroreact` to **1.57.0**, to match the iOS ARCore pods. The C API is additive between 1.51 and 1.57 (no removed symbols). Build the lib with `cd android && ./gradlew :viroar:assembleRelease` (NDK 27.1.12297006); output lands in `android/viroar/build/natives/jni/`.

### Geospatial VPS camera-config selection (Android)

**Problem.** Upstream's `SessionNative` constructor (`android/viroar/src/main/cpp/arcore/ARCore_Native.cpp`) forces the **highest-resolution ARCore CPU camera config** (typically `1920×1080`) for every AR session, to give **Cloud Anchors** more feature points for matching. That resolution starves and delays the camera frame pipeline, which prevents ARCore's **Geospatial API / VPS** from visually localizing on Android: the geospatial camera pose never converges past GNSS accuracy (several metres, with a large orientation-yaw error), so geospatially-anchored content is placed in the wrong position/orientation. VPS localization is tuned for ARCore's default CPU image (~`640×480`).

**Symptom.** `Earth`/geospatial pose reports `horizontalAccuracy` of several metres and a high `orientationYawAccuracy` even where VPS coverage exists and a stock ARCore Geospatial app localizes to ~1 m at the same spot. Confirmed with a same-location A/B: `640×480` → ~1 m lock (yaw ~2°); `1920×1080` → ~5 m GNSS (yaw ~7–20°).

**Change.** The camera-config selection now picks the config **closest to `640×480`** instead of the largest, so Geospatial VPS localizes reliably.

**Trade-off.** If your app depends on **Cloud Anchors** (which benefit from the higher-resolution image) rather than the Geospatial API, this change is not for you — revisit the selection in `SessionNative::SessionNative`.

## Documentation

- ViroReact docs and guides: <https://viro-community.readme.io/docs/overview>
- `ViroObjectDetector` (on-device detection): <https://github.com/ReactVision/viro/blob/main/docs/ViroObjectDetector.md>

## Community

Discord is the best place to find the team and other developers building with ViroReact and ViroCore:

<a href="https://discord.gg/A6TaFNqwVc">
  <img src="https://discordapp.com/api/guilds/774471080713781259/widget.png?style=banner2" />
</a>

Found a bug? Please file it in this repository.

## Enterprise Support and Partners

For teams adopting XR at scale, the ReactVision team works directly with you on architecture, migrations from 8th Wall or Unity, performance audits, and shipping production experiences. <https://reactvision.xyz/contact>

For independent agencies and consultants vetted by the ReactVision team: <https://reactvision.xyz/partners>

## Powered by

[![JetBrains logo](https://resources.jetbrains.com/storage/products/company/brand/logos/jetbrains.svg)](https://jb.gg/OpenSource)

JetBrains supports ViroReact and ViroCore development through their [Open Source program](https://jb.gg/OpenSource).

## Supporters

A huge thank you to everyone who supports ongoing development by sponsoring us on GitHub: <https://github.com/sponsors/ReactVision>

## Find Out More

- Website: <https://reactvision.xyz>
- ViroReact: <https://reactvision.xyz/viro-react>
- ReactVision Studio: <https://studio.reactvision.xyz>
- Blog: <https://updates.reactvision.xyz>

## A little history…

ViroCore was originally developed by Viro Media and open-sourced in 2019. In late 2020 the Viro Community formed to maintain and modernise the project. In January 2025 Morrow Digital acquired it, and in late 2025 ReactVision was spun out as an independent company with a full-time team building the next generation of XR tooling — with ViroCore as the rendering engine at its core.

---

MIT licensed. © ReactVision, Inc.
