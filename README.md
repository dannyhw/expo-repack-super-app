# Super App Showcase on Expo + Re.Pack

An Expo version of [callstack/super-app-showcase](https://github.com/callstack/super-app-showcase),
built with Re.Pack (Rspack) through the `@callstack/repack-expo` integration
from [callstack/repack#1425](https://github.com/callstack/repack/pull/1425).

## Packages

| Package | Role | Dev server |
| --- | --- | --- |
| `packages/host` | Expo app (native shell), React Navigation + native bottom tabs, auth gate | 8081 |
| `packages/auth` | MF v2 remote: `AuthProvider`, `SignInScreen`, `AccountScreen` | 9003 |
| `packages/trading` | MF v2 remote: markets list, asset details (Skia chart), trade bottom sheet | 9001 |
| `packages/wallet` | MF v2 remote: portfolio | 9002 |
| `packages/sdk` | Shared runtime (live Kraken prices, theme) and the MF `shared` config | - |

The host was created with `create-expo-app` (blank TypeScript template, Expo
SDK 57) and set up with `repack-expo init`, then `expo prebuild`. The mini
apps are Expo projects built with `ExpoPlugin` + `ModuleFederationPluginV2`;
they have no native projects and are loaded by the host at runtime.

## Requirements

Node.js 20.19+, pnpm 10, Xcode (iOS) and/or Android Studio (Android).

## Run

```sh
pnpm install
cd packages/host && npx expo prebuild && cd ../..   # generates ios/ and android/
pnpm start                                          # mprocs: host + 3 mini app dev servers
pnpm ios                                            # or: pnpm android
```

On Android, forward the mini app ports to the emulator:

```sh
for p in 8081 9001 9002 9003; do adb reverse tcp:$p tcp:$p; done
```

## Differences from super-app-showcase

- React Native 0.86.3 (Expo SDK 57) instead of 0.87.1, and the native libraries
  at versions that support it. Versions live in the `catalog` in
  `pnpm-workspace.yaml`. Libraries also used by the Re.Pack tester apps use
  the same versions validated there.
- Rspack 2 instead of webpack (`repack-expo` is Rspack-only).
- `expo-splash-screen` instead of `react-native-bootsplash`.
- `@expo/vector-icons` instead of `@react-native-vector-icons/*`, so the
  react-native-paper icon patch is no longer needed. Android tab icons are
  rendered with `getImageSource` before the tab bar mounts.
- `getSharedDependencies` reads versions from the installed packages instead
  of a hardcoded `dependencies.json`, and also shares the Expo modules.
- Mini apps declare their own dependencies instead of resolving the host's
  `node_modules`.
- Not ported: React Compiler, code signing of the auth remote, Jest tests,
  rnx-kit align-deps.

## Integration issues found with this app

Building this app surfaced two bugs in `@callstack/repack-expo`, both fixed
before the 0.0.1 release:

- Every module failed with "Failed to import 'hermes-parser'" because Re.Pack
  looked for hermes-parser through `@react-native/babel-preset`, which Expo
  apps don't install.
- React Navigation and AsyncStorage failed with "Module not found" for their
  extensionless imports, because `ExpoPlugin` keeps ES modules and Rspack
  enforced fully-specified ESM resolution in `"type": "module"` packages.
