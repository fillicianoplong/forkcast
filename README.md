# Forkcast

Forkcast is an Android-first nutrition app that logs food by weight. Put food on a Bluetooth kitchen scale, pick what it is, and Forkcast scales the nutrition facts to the grams and shows your progress against daily goals. No scale? Type grams or servings.

**Status:** In design. Requirements and architecture version 1.1 (October 23, 2026); implementation starting.

## Key features

- **Food lookup:** USDA FoodData Central search and an offline My foods list.
- **Custom foods:** enter a US Nutrition Facts label; unknown values stay unknown, and a 0 is a real zero.
- **Grams or servings:** log by weight, typed grams, or servings (for example 2 slices). Nothing is saved until you tap Log.
- **Bluetooth scales:** pair one or more BLE kitchen scales, starting with the Etekcity ESN00, through one adapter per scale model.
- **Live weighing:** grams and nutrients update as the weight changes; Log turns on once the reading is stable.
- **Today screen:** one day at a time, entries by meal slot, and a summary of every tracked nutrient against its goal.
- **Goals:** FDA Daily Values by default, limits shown as stay-under, and custom goals, with a partial-data flag for unknown values.
- **Planned (Should):** weighing with the app closed (a notification opens the weighing screen; nothing is saved in the background), barcode scanning, Meals and Recipes, pinned goals, restaurant foods, Google Drive backup, and first-run onboarding.
- **Private by design:** no account and no server. Data stays in an encrypted database on the phone, with export to a backup file.

## Tech stack

| Area | Choice |
|---|---|
| Platform | Android 12 and later (minSdk 31, targetSdk 36); iOS later from the same code |
| App | React Native 0.85, Expo SDK 56 (development build), expo-router, TypeScript |
| Bluetooth | react-native-ble-plx, with one scale adapter per scale model |
| Background weighing | Custom Expo native module (Kotlin): Companion Device Manager, a connectedDevice foreground service, and a placement detector per scale model |
| Storage | expo-sqlite with SQLCipher; key in the Android Keystore through expo-secure-store; excluded from Android Auto Backup; no server |
| Food data | USDA FoodData Central API, with an optional personal API key |
| Backup | Google Drive app data folder through react-native-cloud-storage; file export and import |
| Barcode scanning | expo-camera |
| Notifications | expo-notifications; the app-closed notification is posted from Kotlin |
| Testing and linting | Jest with captured-packet fixtures, the TypeScript typecheck, and ESLint, run in CI |

Forkcast will be released on Google Play, Android first.

## Documentation

- [Software Requirements Specification](docs/SRS.md) (version 1.1; also as [Word](docs/Plong_Forkcast_SRS.docx) and [PDF](docs/out/Plong_Forkcast_SRS.pdf))
- [Architecture (design document)](docs/ARCHITECTURE.md) (version 1.1; also as [Word](docs/Plong_Forkcast_Architecture.docx) and [PDF](docs/out/Plong_Forkcast_Architecture.pdf))
- [Diagrams](docs/diagrams/) (PlantUML sources, rendered in [docs/images](docs/images/))

## License

© 2026 Filliciano Plong. All rights reserved. This source code is public for viewing only; no license is granted to copy, modify, or distribute it.
