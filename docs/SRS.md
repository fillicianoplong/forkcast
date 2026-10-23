# Software Requirements Specification: Forkcast

Prepared by: Filliciano Plong · Date: October 23, 2026 · Version: 1.1

## Contents

- [Revision history](#revision-history)
- [1. Introduction](#1-introduction)
- [2. Project scope](#2-project-scope)
- [3. User types](#3-user-types)
- [4. Functional requirements](#4-functional-requirements)
- [5. Non-functional requirements](#5-non-functional-requirements)
- [6. Use cases](#6-use-cases)
- [7. Assumptions and decisions](#7-assumptions-and-decisions)
- [8. Constraints](#8-constraints)
- [9. External interfaces](#9-external-interfaces)
- [10. Acceptance criteria](#10-acceptance-criteria)
- [11. Future enhancements](#11-future-enhancements)
- [12. Interface sketch](#12-interface-sketch)
- [Appendix A. Traceability](#appendix-a-traceability)

## Revision history

| Version | Date | Changes |
|---|---|---|
| 1.0 | October 22, 2026 | Initial release. |
| 1.1 | October 23, 2026 | Added contents page and Navigation Pane headings. |

## 1. Introduction

### 1.1 Purpose

This document defines what Forkcast must do. Forkcast is a phone app that logs food by weight. A Bluetooth scale sends the weight, the user names the food, and the app scales its nutrients and tracks daily goals.

### 1.2 Intended audience

This document is written for Filliciano Plong, the solo developer, to plan, build, and test Forkcast.

### 1.3 Problem statement

Weighing food is accurate but slow: typing grams into a calorie app and checking protein or sodium is enough effort that it often does not happen. Forkcast takes the weight from the scale and shows each goal at a glance.

### 1.4 Product overview

Forkcast is an Android app, later on Google Play. The user pairs Bluetooth Low Energy (BLE) kitchen scales, starting with the Etekcity ESN00. Placing food on a scale opens the live weighing screen; the user picks the food from USDA FoodData Central, My foods, or a barcode, and taps Log. Only Log saves a weight. Typed grams or servings work with no scale.

With the app closed, weighing posts a notification that opens the live weighing screen; nothing is saved in the background. Several items can be weighed in one container and saved as a Meal or Recipe.

The Today screen shows one day: pinned goals, entries by meal slot, and a nutrient summary. Goals default to FDA Daily Values, limits show as stay under, and custom goals are allowed.

All data lives in an encrypted database on the phone. There is no account and no server. The user backs up to Google Drive or exports a file.

Starting point: the repository fillicianoplong/forkcast holds an MVP (React Native 0.85, Expo SDK 56, expo-router, TypeScript) with Logs, Food, and Profile tabs, an Add Food label form, and AsyncStorage. Forkcast replaces Logs with Today and the Profile placeholder with Profile and Settings.

### 1.5 Definitions

| Term | Meaning |
|---|---|
| Paired scale | A BLE kitchen scale the user has added in Forkcast and that a scale adapter supports. |
| Active scale | The paired scale Forkcast is reading from right now. |
| Scale adapter | The part of the app that understands one kind of scale and turns its data into readings in grams. |
| Stable reading | A weight the scale marks as settled, or one that stays inside the stability tolerance for the stability window. |
| Placement | Putting something on the scale: the reading reaches the minimum weight (2 g by default) and holds for the stability window (section 9.4). |
| Serving | A food's named portion (for example 1 bar, 2 slices, or 1 restaurant item), converted to grams by the food's serving size when it has one. |
| Daily Value (DV) | The FDA reference amount for a nutrient on a 2,000-calorie diet, the same basis as the %DV on a US label (section 9.8). |
| Limit nutrient | A nutrient whose Daily Value is a maximum: saturated fat, cholesterol, sodium, and added sugars. Shown as stay under. |
| Goal | A daily target for one nutrient: the default DV, or a custom reach or stay-under target with an optional minimum and maximum. |
| Pinned goal | A goal the user chose to show as a progress bar at the top of the Today screen. |
| Unknown value | A nutrient the food's label or data does not give. Stored as null, not 0. |
| Partial data | A goal whose total for the day includes at least one unknown value. |
| Day | The phone's local calendar date when an entry is logged. The user can change an entry's date. |
| Meal slot | Breakfast, Lunch, Dinner, or Snacks: where an entry sits on the Today screen. |
| My foods | The user's food history: foods logged, looked up, or created before, plus Meals and Recipes. |
| Session | Weighing several items in one container with Weigh several. Each item's grams are the increase since the last item (tare and add). |
| Meal / Recipe | Meal: saved foods and amounts, logged as one entry each. Recipe: a food made from ingredients and a final weight. |
| FDC | USDA FoodData Central, the app's food database. |
| CDM | Android Companion Device Manager, which tells the app when a paired scale is nearby and powered on. |
| Backup file | One file holding all user data (section 9.7), written to Google Drive or exported. |

## 2. Project scope

The system will allow the user to:

- Pair one or more Bluetooth scales (the Etekcity ESN00 first), and weigh food with the weight and nutrition updating live
- Get a notification when weighing with the app closed or the phone locked, and open the weighing screen from it
- Log food by weight, typed grams, or servings, from FDC search, My foods, or a barcode
- Create custom foods from a US Nutrition Facts label, including restaurant and fast-food items
- Weigh several items in one container, save them as a Meal or a Recipe, and log them again without weighing
- See each day on the Today screen, with pinned goals, entries by meal slot, and a full nutrient summary
- Track daily goals: FDA Daily Values by default, limits as stay under, and custom goals
- Keep a simple profile and settings, with all data on the phone and backups to Google Drive or an exported file

The system will not include:

- Medical advice, diet plans, or diagnosis
- Calorie or energy estimation, activity levels, or weight tracking over time
- Saving weights in the background, or weights with no food
- Accounts, sign-in, or cloud sync (section 11)
- A restaurant menu database; restaurant foods are entered by hand
- An iOS release at first (section 11), social features, a web version, or Health Connect and Apple Health sync

## 3. User types

Forkcast has one user per phone; for now that is Filliciano. The two roles below are usage patterns the use cases refer to.

| User | Role | Experience | Tasks |
|---|---|---|---|
| Daily tracker | Weighs most food at home to meet protein and stay under sodium. Eats out sometimes. | Uses a smartphone daily and has used a calorie app. | Weighs oats, logs 2 slices of bread, pins a 120 g protein goal, adds a fast-food burger. |
| Meal prepper | Cooks in batches and eats the same lunches during the week. | Comfortable with a kitchen scale and recipes. | Builds a 2,400 g chili recipe, weighs 350 g portions, saves a usual lunch as a Meal. |

## 4. Functional requirements

Each requirement has one MoSCoW priority for the first Google Play release: Must is required, Should is planned and first to cut, and Could is built only if time remains. Won't items are the exclusions in sections 2 and 11. GitHub issues map Must, Should, and Could to priority:high, priority:medium, and priority:low. Within a section, Must comes first; IDs run in document order. Appendix A traces each requirement to use cases and acceptance criteria.

### 4.1 App shell

- **FR-01 (Must):** The system shall run as an Android app on Android 12 (API level 31) or later.
- **FR-02 (Must):** The system shall provide Today, Foods, and Profile tabs.
- **FR-03 (Must):** The system shall open Settings from Profile, and Goals from Settings and from the pinned goals.
- **FR-04 (Must):** The system shall not require an account or sign-in for any feature.
- **FR-05 (Must):** The system shall explain each Android permission in section 9.6 before asking, and keep typed grams, search, and typed barcodes working if one is denied.

### 4.2 First-run onboarding

- **FR-06 (Should):** The system shall, on first launch, show a short introduction: what Forkcast does, that data stays on the phone, and that no account is needed.
- **FR-07 (Should):** The system shall offer, in order, to pair a scale, choose units, fill in the profile, and pin goals, each skippable.
- **FR-08 (Should):** The system shall explain that default goals are FDA Daily Values for a 2,000-calorie diet, not personal advice, and can be customized anytime.
- **FR-09 (Could):** The system shall let the user show the introduction again from Settings > About.

### 4.3 Food lookup

- **FR-10 (Must):** The system shall search FDC Foundation, SR Legacy, and Branded Foods by name and brand, labeling each result's data type, with nutrients per serving and per 100 g.
- **FR-11 (Must):** The system shall search My foods without a network, ranked by recent and frequent use.
- **FR-12 (Must):** The system shall save each viewed or logged FDC food to My foods, with its values, FDC ID, and data date, for offline use.
- **FR-13 (Must):** The system shall send an FDC search after 400 ms without typing, show My foods results first, and show a try-again message on HTTP 429.
- **FR-14 (Should):** The system shall use the built-in FDC key unless the user enters a personal data.gov key in Settings > Food data, which it offers when FDC rejects a key (section 9.9).
- **FR-15 (Should):** The system shall scan UPC-A, UPC-E, EAN-13, and EAN-8 barcodes, checking My foods first and then FDC Branded Foods.
- **FR-16 (Should):** The system shall offer manual entry of a custom food with the barcode attached when FDC has no match.
- **FR-17 (Should):** The system shall let the user type the barcode digits when the camera permission is denied or no camera is available.

### 4.4 Custom foods and nutrient values

- **FR-18 (Must):** The system shall create a custom food from the section 9.8 label fields, requiring name and energy.
- **FR-19 (Must):** The system shall accept custom food values per serving with serving grams, per 100 g, or per serving only.
- **FR-20 (Must):** The system shall store a nutrient the label or FDC does not give as unknown (null), and a label value of 0 as a real 0.
- **FR-21 (Must):** The system shall snapshot a food's nutrients on each entry, so later food edits do not change past days, except as FR-22 allows.
- **FR-22 (Should):** The system shall let the user fill in a food's unknown values later and, after confirmation, copy them into past entries where still unknown. Known past values never change.
- **FR-23 (Should):** The system shall let the user mark a custom food as a restaurant item, with an optional restaurant name and source link shown on the food.
- **FR-24 (Should):** The system shall let the user attach a barcode to a custom food, edit it, and archive it. An archived food leaves search but stays on past entries.

### 4.5 Logging

- **FR-25 (Must):** The system shall save an entry only on Log, writing food, amount, meal slot, date and time, and nutrient snapshot in one transaction.
- **FR-26 (Must):** The system shall let the user type grams instead of weighing, whether or not a scale is connected.
- **FR-27 (Must):** The system shall let the user log by servings (for example 2 slices), converting with the food's serving size. Foods with no serving grams log by servings only.
- **FR-28 (Must):** The system shall compute each entry nutrient as per-gram value times grams, or per-serving value times servings.
- **FR-29 (Must):** The system shall file each entry under the local date and the meal slot (from Settings meal times) at logging. A later time zone change does not move it.

### 4.6 Scales and adapters

- **FR-30 (Must):** The system shall read every scale through a scale adapter that yields grams and a stable flag, starting with an Etekcity ESN00 adapter (section 9.3).
- **FR-31 (Must):** The system shall scan for BLE scales only on the Add scale screen, listing each scale an adapter recognizes.
- **FR-32 (Must):** The system shall pair several scales, each with an editable name and connection state, and forget one without changing past entries.
- **FR-33 (Must):** The system shall convert every reading to grams whatever the scale's unit, and tare on the scale if its adapter supports it, otherwise in the app.
- **FR-34 (Should):** The system shall use the paired scale that is on as the active scale, or the most recently used if several are.
- **FR-35 (Could):** The system shall ship with an adapter for the standard Bluetooth Weight Scale Service (0x181D, section 9.2).
- **FR-36 (Could):** The system shall show a paired scale's battery level when the scale reports it.

### 4.7 Live weighing screen

- **FR-37 (Must):** The system shall open the live weighing screen when the app is open and a placement starts (section 9.4), unless Auto-open is off.
- **FR-38 (Must):** The system shall show live grams, a stable indicator in text and color, the active scale with a quick switch, and Tare, Search, Scan barcode, My foods, and Weigh several.
- **FR-39 (Must):** The system shall show energy, fat, carbohydrate, and protein for the selected food at the current weight, updating live.
- **FR-40 (Must):** The system shall enable Log only when the reading is stable or the user has typed grams. Typed grams replace the live weight for that entry.
- **FR-41 (Must):** The system shall discard the weight when the user closes the screen without logging. If a stable weight was showing, it shall ask first.

### 4.8 Weighing with the app closed

- **FR-42 (Must):** The system shall not save any weight or entry while the app is closed. An ignored notification saves nothing.
- **FR-43 (Should):** The system shall use the Companion Device Manager to notice a paired scale turning on, and run a connectedDevice foreground service only while a scale is on and connected.
- **FR-44 (Should):** The system shall detect a placement with the app closed even after Android has stopped the app's process, without starting the app's screens (section 9.5).
- **FR-45 (Should):** The system shall post one notification per placement, saying something is being weighed, when the app is closed or the phone is locked.
- **FR-46 (Should):** The system shall open the live weighing screen with the live weight when that notification is tapped (after unlocking).
- **FR-47 (Should):** The system shall show in Settings when background weighing is unavailable (for example notifications denied or Bluetooth off), and keep working with the app open.

### 4.9 Meals and recipes

- **FR-48 (Should):** The system shall start a session, with an empty list and the scale tared, when the user taps Weigh several on the live weighing or Today screen.
- **FR-49 (Should):** The system shall add each item by tare and add, recording the weight increase as its grams. Per-serving-only foods are added by servings.
- **FR-50 (Should):** The system shall show each item's amount and running totals, and let the user correct or remove items before finishing.
- **FR-51 (Should):** The system shall log a finished session as one entry per item in the chosen meal slot, whether or not it is saved.
- **FR-52 (Should):** The system shall save a finished session as a Meal (each food with its grams or servings).
- **FR-53 (Should):** The system shall log a saved Meal in one step without weighing, as one entry per item.
- **FR-54 (Should):** The system shall scale a Meal by a factor or to a new total weight when logging it.
- **FR-55 (Should):** The system shall save a finished session as a Recipe food in My foods, asking for its final total weight.
- **FR-56 (Should):** The system shall compute a Recipe's nutrients per gram from its ingredients and its final total weight.
- **FR-57 (Should):** The system shall log a portion of a Recipe like any other food.
- **FR-58 (Should):** The system shall accept only gram-based foods as Recipe ingredients.
- **FR-59 (Should):** The system shall list Meals and Recipes in My foods and let the user rename, edit, duplicate, and delete them. Recipe edits affect new entries only.

### 4.10 Today screen

- **FR-60 (Must):** The system shall show one day at a time, changed by swiping or by the previous and next buttons.
- **FR-61 (Must):** The system shall show the active scale and a Weigh button on the Today screen.
- **FR-62 (Must):** The system shall group entries by meal slot (Breakfast, Lunch, Dinner, Snacks), showing food, amount, and calories.
- **FR-63 (Must):** The system shall edit an entry's amount, food, meal slot, date, and time on tap, and delete it on swipe, with Undo.
- **FR-64 (Must):** The system shall show an expandable summary of every tracked nutrient (section 9.8) against its goal.
- **FR-65 (Must):** The system shall show each nutrient gray when under, green when met, and red when over a limit, with the status also in text.
- **FR-66 (Should):** The system shall jump to any date from a calendar button.
- **FR-67 (Should):** The system shall show pinned goals at the top as bars or rings with the day's intake and target.

### 4.11 Goals

- **FR-68 (Must):** The system shall default each nutrient's goal to its FDA Daily Value for a 2,000-calorie diet (section 9.8), with no profile needed. Energy has no default.
- **FR-69 (Must):** The system shall treat limit nutrients (section 9.8) as stay under: met at or below the limit, over above it.
- **FR-70 (Must):** The system shall let the user set a custom reach or stay-under goal, with optional minimum and maximum, for any tracked nutrient, replacing its default.
- **FR-71 (Must):** The system shall keep several goals active at the same time and check each one for every day.
- **FR-72 (Must):** The system shall count unknown values as 0 in daily totals and mark a goal partial when any entry that day has its nutrient unknown.
- **FR-73 (Should):** The system shall let the user pin, unpin, and reorder the goals shown on the Today screen.

### 4.12 Profile

- **FR-74 (Could):** The system shall offer an optional, on-phone profile: name, age, height, current weight, and gender.
- **FR-75 (Could):** The system shall keep gender as a simple field, with no sex-for-targets label and no pregnancy or breastfeeding options.
- **FR-76 (Could):** The system shall keep only the current weight, with no history or chart.
- **FR-77 (Could):** The system shall use profile values for display only, not for goals or energy estimates (section 11).

### 4.13 Settings

- **FR-78 (Must):** The system shall offer food weight units (g, oz, or lb and oz) and profile units (kg or lb; cm or ft and in).
- **FR-79 (Must):** The system shall show food weight to 1 g or 0.1 oz.
- **FR-80 (Must):** The system shall store all values in metric units, rounding only for display.
- **FR-81 (Must):** The system shall list paired scales, with Add scale, rename, forget, and the active scale switch.
- **FR-82 (Must):** The system shall offer backup destination and schedule, last backup time, Back up now, Restore, Export, Import, and Delete all data.
- **FR-83 (Must):** The system shall show About with the app version, privacy policy link, Show introduction again, and the FDC citation.
- **FR-84 (Should):** The system shall offer theme (system, light, or dark) and meal times.
- **FR-85 (Should):** The system shall offer Notify when weighing, Auto-open, stability window and tolerance, and minimum weight (section 9.4).

### 4.14 Data and backup

- **FR-86 (Must):** The system shall keep all user data in an encrypted on-device SQLite database (expo-sqlite with SQLCipher) as the source of truth.
- **FR-87 (Must):** The system shall export all data to one backup file (section 9.7) via the share sheet or file picker, and import one.
- **FR-88 (Must):** The system shall check a backup file (section 9.7) and confirm before replacing data.
- **FR-89 (Must):** The system shall change nothing when a backup file is damaged, has a newer format, or has the wrong passphrase.
- **FR-90 (Must):** The system shall detect at startup a database its key cannot open, and offer Restore, Import, or Start fresh instead of crashing.
- **FR-91 (Must):** The system shall, after confirmation, delete all data and the Keystore database key and sign out of Drive, first offering to delete the Drive backups.
- **FR-92 (Should):** The system shall back up to and restore from the app's private Google Drive folder, showing the last backup time and an overdue reminder.
- **FR-93 (Could):** The system shall offer a best-effort daily or weekly backup that runs the next time the app is open after the scheduled time.
- **FR-94 (Could):** The system shall move data from the earlier MVP's AsyncStorage into the database on first launch.

## 5. Non-functional requirements

Numbers below are targets for the first release, checked by the acceptance criteria in section 10.

### 5.1 Reliability

- **NFR-01 (Must):** The system shall write a logged entry in one transaction within 1 second of Log, before leaving the screen.
- **NFR-02 (Must):** The system shall leave the database unchanged by weighing with the app closed: the same row counts and no new write (FR-42).
- **NFR-03 (Must):** The system shall log from My foods and edit entries offline. A failed lookup keeps the weight on screen for another choice.
- **NFR-04 (Must):** The system shall keep saved data across app restarts, phone restarts, and app updates, and shall migrate the database when its format changes.
- **NFR-05 (Must):** The system shall write each backup as a new file before replacing the old, so a failure keeps the last good one.
- **NFR-06 (Must):** The system shall exclude the database and key from Auto Backup and device transfer (section 9.7), so no restore brings back a keyless database.

### 5.2 Security and privacy

- **NFR-07 (Must):** The system shall run no server holding user data. Data leaves the phone only in user-started or scheduled backups and exports, never to analytics or ads.
- **NFR-08 (Must):** The system shall use TLS (HTTPS) for every network request.
- **NFR-09 (Must):** The system shall encrypt the on-device database with SQLCipher, keep its key in the Android Keystore, and confirm encryption is active at startup.
- **NFR-10 (Must):** The system shall send only the search text or barcode, with the API key, to FDC, never the user's log or profile.
- **NFR-11 (Must):** The system shall keep the built-in FDC key and other secrets out of the repository, and a personal key only in Keystore-backed storage.
- **NFR-12 (Should):** The system shall back up only to the app-private Drive folder and offer passphrase encryption (section 9.7), warning when a file is unencrypted.

### 5.3 Performance

- **NFR-13 (Must):** The system shall show a new reading on the live weighing screen within 200 ms of receiving it. How often readings arrive depends on the scale.
- **NFR-14 (Must):** The system shall return My foods results within 300 ms, and FDC results within 3 seconds on a 10 Mbps, 100 ms test network.
- **NFR-15 (Must):** The system shall connect to a paired scale within 10 seconds of it turning on while the app is open.
- **NFR-16 (Should):** The system shall open the Today screen within 3 seconds of a cold start on the test phone (section 8).
- **NFR-17 (Should):** The system shall show a day with 40 entries within 500 ms, with a year of daily use (about 4,000 entries) in the database.
- **NFR-18 (Should):** The system shall post the app-closed notification within 5 seconds of a placement (where Android reports the scale), and show the live weight within 5 seconds of opening it.
- **NFR-19 (Should):** The system shall stop the foreground service and remove its notification within 30 seconds of the scale turning off or disconnecting.

### 5.4 Usability

- **NFR-20 (Should):** The system shall let the user log a stable weight with a food from My foods in three taps or fewer from the live weighing screen.
- **NFR-21 (Should):** The system shall let the user finish or skip onboarding in under 2 minutes.

### 5.5 Accessibility

- **NFR-22 (Must):** The system shall label every control for TalkBack, announce the weight only when stable and at most every 2 seconds, and read each goal's intake, target, and status.
- **NFR-23 (Must):** The system shall support system font sizes up to 200% without cutting off weights, goals, or buttons.
- **NFR-24 (Must):** The system shall not rely on color alone: states also read Under, Met, or Over, and contrast meets WCAG 2.1 AA (4.5:1 text, 3:1 bars).
- **NFR-25 (Must):** The system shall make touch targets at least 48 by 48 dp, and shall offer a button for every swipe action (change day, delete entry).
- **NFR-26 (Should):** The system shall reduce animation when the system's remove animations setting is on.

### 5.6 Error handling

- **NFR-27 (Must):** The system shall, when FDC is unreachable, slower than 10 seconds, or rejects the key, say so in one line and keep local foods and typed entry working.
- **NFR-28 (Must):** The system shall retry a failed write once, then show the error and keep the grams and food. On full storage it says so and writes nothing partial.
- **NFR-29 (Must):** The system shall copy the database file before a migration and, if the migration fails, return to that copy and offer Export.
- **NFR-30 (Should):** The system shall, when Google sign-in or a Drive backup or restore fails, show the reason, keep current data unchanged, and offer Export instead.

### 5.7 Google Play release

- **NFR-31 (Must):** The system shall target the API level Google Play requires at submission (Android 16, API level 36, from August 31, 2026; section 9.11).
- **NFR-32 (Must):** The system shall pass Google Play review with the section 9.11 privacy policy and declarations, each matching the app's behavior.

### 5.8 Maintainability

- **NFR-33 (Must):** The system shall add support for a new scale model as a new scale adapter, without changing the weighing screen, goals, or logging code.
- **NFR-34 (Must):** The system shall keep nutrient scaling, totals, goal checks, and packet parsing in plain functions unit-tested without Bluetooth or UI.
- **NFR-35 (Must):** The system shall test each background detector on the same captured packets as its scale adapter.
- **NFR-36 (Should):** The system shall reach stored data only through repositories, and give every record a UUID, timestamps, and a soft-delete flag, so sync can be added later.
- **NFR-37 (Should):** The system shall put the food source behind one interface, so new sources need no logging changes.

## 6. Use cases

### UC-01: Set up Forkcast on first launch

**Primary user:** Daily tracker (section 3)

**Goal:** Get from install to a usable app quickly.

**Requirements:** FR-01, FR-02, FR-03, FR-04, FR-05, FR-06, FR-07, FR-08, FR-74, FR-75, FR-78, FR-79, FR-80, FR-94, NFR-21

**Precondition:** Forkcast is installed on an Android phone and opened for the first time.

**Normal sequence:**

1. The app shows the introduction: logs by weight, data stays on the phone, no account.
2. The user taps Pair a scale and follows UC-04, or skips.
3. The user chooses grams and kg, enters an age, and leaves the other profile fields blank.
4. The app explains the default goals and the user pins Calories, Protein, and Sodium.

**Expected result:** The Today screen opens with three pinned goals.

**Alternative outcomes:**

- The user skips everything: the Today screen opens with default goals and no scale. Typed grams still work.
- The phone had the earlier MVP installed: its foods and logs appear in Forkcast.

### UC-02: Find or create a food

**Primary user:** Daily tracker (section 3)

**Goal:** Have the right food, with its nutrients, ready to log.

**Requirements:** FR-10, FR-11, FR-12, FR-13, FR-14, FR-15, FR-16, FR-17, FR-18, FR-19, FR-20, FR-21, FR-22, FR-23, FR-24, NFR-14, NFR-27

**Precondition:** The app is open.

**Normal sequence:**

1. The user types “greek yogurt” in Search. My foods results show at once; FDC results follow after the user stops typing.
2. The user opens a Branded Foods result. It is saved to My foods and works offline from then on.
3. The user scans a granola bar. FDC has no match, so the app offers a custom food with the barcode attached.
4. The user enters the label values, leaving potassium blank, and saves.

**Expected result:** Both foods are in My foods. Potassium on the granola bar is unknown, not 0.

**Alternative outcomes:**

- FDC is unreachable or rejects the key: the app says so, and My foods and custom foods still work. The user can enter a personal data.gov key in Settings > Food data.
- The user adds a fast-food burger as a restaurant item with the chain's nutrition page as its source.
- Later the user fills in potassium on the granola bar and confirms: past entries of it get the value.

### UC-03: Log a food without a scale

**Primary user:** Daily tracker (section 3)

**Goal:** Log a food by typed grams or servings.

**Requirements:** FR-25, FR-26, FR-27, FR-28, FR-29, NFR-01, NFR-03

**Precondition:** The app is open. No scale is needed.

**Normal sequence:**

1. The user taps Weigh on the Today screen and picks Oats from My foods.
2. The user types 45 g and taps Log.
3. The user picks Bread and chooses 2 slices instead of grams, then taps Log.

**Expected result:** Both entries are under Breakfast for today, with nutrients scaled to the amounts, and the pinned goals update.

**Alternative outcomes:**

- The user logs a late snack after midnight: it is filed under the new date, and the user can change its date in the entry editor (UC-09).

### UC-04: Pair a scale

**Primary user:** Daily tracker or meal prepper (section 3)

**Goal:** Connect a scale once so the app can use it from then on.

**Requirements:** FR-05, FR-30, FR-31, FR-32, FR-33, FR-34, FR-35, FR-36, FR-81, NFR-15

**Precondition:** The scale is on and in range. The app is open.

**Normal sequence:**

1. The user opens Settings > Scales > Add scale.
2. The app explains the Bluetooth and notification permissions and asks for them.
3. The app lists the ESN00. The user selects it, accepts Android's companion device prompt, and names it Kitchen.
4. The app connects and shows the live weight to confirm.

**Expected result:** Kitchen is a paired scale, and Android reports to Forkcast when it turns on.

**Alternative outcomes:**

- A second scale is paired: whichever one is on is used; if both are on, the most recent one, with a quick switch.
- The scale is not recognized, or permission is denied: the app says why and offers typed grams.

### UC-05: Weigh and log a food with the app open

**Primary user:** Daily tracker (section 3)

**Goal:** Log a food at the exact weight on the scale.

**Requirements:** FR-25, FR-37, FR-38, FR-39, FR-40, FR-41, NFR-13, NFR-20, NFR-22

**Precondition:** A paired scale is on and the app is open.

**Normal sequence:**

1. The user places a bowl and taps Tare.
2. The user adds oats. The live weighing screen opens and the grams update.
3. The user picks Oats from My foods. Energy and macros follow the weight.
4. The scale settles at 45 g, Log becomes available, and the user taps it.

**Expected result:** A 45 g oats entry is under Breakfast, and the pinned goals update.

**Alternative outcomes:**

- The user types 40 g before the weight settles: Log uses 40 g.
- The user closes the screen without logging: the app asks, and the weight is discarded.
- The scale disconnects before Log: the screen says so and accepts typed grams.

### UC-06: Weigh with the app closed or the phone locked

**Primary user:** Daily tracker (section 3)

**Goal:** Start logging from the scale without opening the app first.

**Requirements:** FR-42, FR-43, FR-44, FR-45, FR-46, FR-47, FR-85, NFR-02, NFR-18, NFR-19

**Precondition:** A paired scale is off. Notify when weighing is on. Notification and Bluetooth permissions are granted.

**Normal sequence:**

1. The user turns the scale on. Android tells Forkcast, which starts its foreground service and connects.
2. The user places food on the scale. The app posts: Something is being weighed.
3. The user taps the notification and unlocks the phone.
4. The live weighing screen opens with the live weight, and the user continues as in UC-05.

**Expected result:** The entry is saved only when the user taps Log in the app.

**Alternative outcomes:**

- The user ignores the notification: nothing is saved.
- The scale turns itself off: the service stops and its notification goes away.
- Android does not report the scale on this phone: Settings says background weighing is unavailable, and weighing works with the app open.

### UC-07: Weigh several items and save a Meal or Recipe

**Primary user:** Meal prepper (section 3)

**Goal:** Weigh several items in one container and save them together.

**Requirements:** FR-48, FR-49, FR-50, FR-51, FR-52, FR-53, FR-54, FR-55, FR-56, FR-57, FR-58, FR-59

**Precondition:** A paired scale is on and the app is open.

**Normal sequence:**

1. The user places a plate and taps Weigh several. The scale is tared.
2. The user adds rice. The app records 180 g as item 1, and the user picks the food.
3. The user adds chicken (150 g) and broccoli (85 g) the same way.
4. The user taps Finish, chooses Lunch, and saves the items as a Meal named Usual lunch.

**Expected result:** Three entries are under Lunch, and Usual lunch is in My foods.

**Alternative outcomes:**

- Next day the user logs Usual lunch from My foods in one step, without weighing.
- The user saves a pot of chili as a Recipe with a cooked weight of 2,400 g, then logs a 350 g portion by weighing it.
- A restaurant item is added to the Meal by servings; it cannot be added to a Recipe.

### UC-08: Set and pin daily goals

**Primary user:** Daily tracker (section 3)

**Goal:** Track the nutrients that matter, against the right targets.

**Requirements:** FR-02, FR-03, FR-67, FR-68, FR-69, FR-70, FR-71, FR-72, FR-73

**Precondition:** The app is open. No custom goals exist yet.

**Normal sequence:**

1. The user opens Goals. Every nutrient with a Daily Value shows it; sodium shows Stay under 2,300 mg.
2. The user types a custom Protein goal of 120 g, with a maximum of 160 g.
3. The user types a Calories goal of 2,200 kcal.
4. The user pins Calories, Protein, and Sodium, and orders them.

**Expected result:** The Today screen shows the three pinned goals with intake and target.

**Alternative outcomes:**

- A logged food has no vitamin D value: the vitamin D goal shows partial data, and the total counts it as 0.

### UC-09: Review and fix a day

**Primary user:** Daily tracker (section 3)

**Goal:** Check yesterday and correct a mistake.

**Requirements:** FR-60, FR-61, FR-62, FR-63, FR-64, FR-65, FR-66, NFR-17, NFR-25

**Precondition:** Entries exist for yesterday.

**Normal sequence:**

1. The user swipes right on the Today screen to yesterday.
2. The user taps the 140 g yogurt entry under Snacks, changes it to 170 g, and saves.
3. The user swipes the duplicate coffee entry to delete it.
4. The user expands the nutrient summary and sees sodium in red, marked Over.

**Expected result:** Yesterday's totals and goals update.

**Alternative outcomes:**

- The user deleted the wrong entry: Undo restores it.
- For an older day, the user taps the calendar button and picks the date.

### UC-10: Adjust profile and settings

**Primary user:** Daily tracker or meal prepper (section 3)

**Goal:** Make the app show units, times, and behavior the way the user wants.

**Requirements:** FR-09, FR-14, FR-74, FR-75, FR-76, FR-77, FR-78, FR-79, FR-80, FR-83, FR-84, FR-85

**Precondition:** The app is open.

**Normal sequence:**

1. The user opens Profile, enters a current weight, and leaves gender blank.
2. The user opens Settings, switches food weight to ounces, and sets Dinner to start at 5:00 PM.
3. The user turns off Auto-open and raises the minimum weight to 5 g.
4. The user opens About and reads the FDC citation.

**Expected result:** Weights show in ounces to 0.1 oz while grams are stored, and new evening entries go to Dinner. Goals do not change.

**Alternative outcomes:**

- The user shows the introduction again from About.

### UC-11: Back up, restore, and delete data

**Primary user:** Daily tracker or meal prepper (section 3)

**Goal:** Keep a copy of all data, restore it on a new phone, and remove it when wanted.

**Requirements:** FR-82, FR-86, FR-87, FR-88, FR-89, FR-90, FR-91, FR-92, FR-93, NFR-05, NFR-06, NFR-12, NFR-30

**Precondition:** The user has data in the app.

**Normal sequence:**

1. The user taps Settings > Backup > Back up now. The app writes to its private folder in Google Drive.
2. On a new phone, the user installs Forkcast, opens Settings > Backup, and taps Restore.
3. The app checks the file and asks before replacing anything.

**Expected result:** All foods, entries, meals, recipes, goals, profile, and settings are restored.

**Alternative outcomes:**

- No Google account is available: the user exports a file with a passphrase and imports it on the other phone.
- The backup is damaged or the passphrase is wrong: nothing is restored and current data stays as it was.
- The database cannot be opened at startup: the app offers Restore from backup, Import a file, or Start fresh.
- The user deletes all data: the app offers to delete the Drive backups first, then removes the data and the key and signs out of Drive.

## 7. Assumptions and decisions

### 7.1 Working assumptions

- The user has an Android phone with Android 12 or later and an Etekcity ESN00, or types grams. There is no iOS device to test on yet.
- The ESN00 behaves as the community notes in section 9.3 describe. This must be checked on Filliciano's scale with nRF Connect.
- Scales that use the standard Weight Scale Service work with the generic adapter. Other scales need their own adapter, added one model at a time.
- The user eats mostly US foods. FDC covers generic and US branded packaged foods; restaurant chains publish nutrition the user can copy into a custom food.
- Most packaged foods state the serving weight in grams. Values a label does not give are unknown, not zero.
- FDA Daily Values for a 2,000-calorie diet are a reasonable default for one adult. They are a general reference, not personal advice.
- One person uses the app on each phone.

### 7.2 Decisions made

**OD-01 Storage model. Decided: Option B, local-first.** No server cost or maintenance, less privacy exposure, and faster to ship the scale features; sync is a growth path (section 11).

**Option A:** Account with a cloud database (Google or email sign-in), with an offline copy on the phone. Not chosen.

| Pros | Cons |
|---|---|
| Data syncs to every phone the user signs in on. | A server to run and pay for, now and later. |
| Data survives a lost phone; matches the Google or email sign-in idea. | Health-adjacent data held on a server must be protected, and in-app account deletion is required. |
| A server can hold food API keys and cache lookups. | Offline edits on two phones need conflict handling. |

**Option B:** Local only, with backup to a cloud the user chooses. Chosen.

| Pros | Cons |
|---|---|
| No server and almost no running cost. | No automatic sync between phones. |
| Data stays with the user and works fully offline; a simpler privacy story. | Data is lost if the user never backs up. Each cloud needs its own integration, or a file export the user must remember. |
| No sign-in before first use. | The FDC API key ships in the app (OD-07). |

- OD-02 First scale: the Etekcity ESN00, which Filliciano owns and will test with. More scales follow through adapters.
- OD-03 Food data: USDA FoodData Central. Restaurant and fast-food items are custom foods entered by hand.
- OD-04 Background weighing: Companion Device Manager presence plus a foreground service only while a scale is on (FR-43). Best effort, to confirm on Filliciano's phone.
- OD-05 Background detection: in the Kotlin module, not JavaScript (section 9.5).
- OD-06 Recipes are foods: a Recipe is saved as a food in My foods, and a Meal logs as one entry per item, so every entry has exactly one food.
- OD-07 FDC key: built in, with an optional personal key (FR-14, section 9.9).
- OD-08 Android versions: minSdk 31 (Android 12) and targetSdk 36 (Android 16), as Google Play requires from August 31, 2026.
- OD-09 Platform: Android first, since there is no iOS device to test on. iOS stays a future option (section 11).
- OD-10 App name: Forkcast.
- OD-11 Personalized targets: not in this version. DRI targets by age and gender stay a future option (section 11).
- OD-12 Privacy policy hosting: GitHub Pages from the forkcast repository, linked in the Play listing and Settings > About.

### 7.3 Still open

- OD-13 Next scale models after the ESN00, and which ones to buy for testing. Decide once the ESN00 adapter works.

## 8. Constraints

- Solo developer, personal project, small budget, on the repository's stack: React Native 0.85, Expo SDK 56, expo-router, and TypeScript, with Kotlin for the native module.
- Native code (BLE, SQLCipher, Companion Device Manager, backup) requires an Expo development build, not Expo Go. Scale tests need Filliciano's phone (the test phone).
- minSdk 31, since Companion Device Manager presence needs Android 12; targetSdk 36 for Google Play. Both presence API forms (Android 16 changed it) are supported. Notifications need permission on Android 13+.
- A connectedDevice foreground service needs its permissions, a visible notification, and the companion device exemption to start in the background. It varies by phone maker, so it is best effort.
- FDC allows 1,000 requests per hour per IP by default and deactivates keys found in public. With no server, the built-in key ships inside the app and could be extracted (OD-07).
- If a build lacks SQLCipher, setting the key silently does nothing, so encryption must be confirmed at startup (NFR-09).
- Google Drive backup needs the user to grant access to the app's private Drive folder (drive.appdata). That is a permission, not an app account.

## 9. External interfaces

### 9.1 User interface

The app has the tabs in FR-02, plus live weighing, the session screen, onboarding, and the weighing notification. Section 12 lists each screen's parts.

### 9.2 Bluetooth scales and the Weight Scale Service

Each scale model has a scale adapter: ESN00 (section 9.3) and the standard Weight Scale Service (0x181D). That service sends weights on Weight Measurement (0x2A9D) and features on Weight Scale Feature (0x2A9E).

Few kitchen scales use it, and its 5 g resolution suits body scales better. Most kitchen scales need vendor adapters, added one model at a time and tested with captured packets (OD-13).

### 9.3 Etekcity ESN00

The ESN00 appears not to use the standard service. A community project (hertzg/metekcity, MIT, work in progress, last changed February 2022) documents vendor service 0x1910, with characteristic 0x2C12 for notifications from the scale, 0x2C11 for commands, and 0x2C10 for reads.

Packets start with the header FE EF C0 A2, then a type byte, a length byte, the payload, and a one-byte sum checksum. The measurement packet carries a sign, a 16-bit big-endian weight (grams times 10 in g mode), a unit code, and a stable flag, and the scale sends it repeatedly. The notes disagree on the measurement type code (0xD0 or 0xE0) and on which characteristic takes commands.

None of this is verified. Filliciano will check it with nRF Connect (and an HCI log if needed) before the adapter is final.

### 9.4 Weighing rules

| Rule | Definition |
|---|---|
| Stable | The adapter's settled flag, or else the reading stays inside the stability tolerance for the stability window (both in Settings). |
| Placement | Starts when the reading reaches the minimum weight (2 g default) for the stability window; ends when it stays below for the window. Smaller drift starts nothing. |
| Log | Enabled when the reading is stable or grams were typed; typed grams win (FR-40). |
| Spoken weight | TalkBack hears the weight only when it becomes stable, at most once every 2 seconds (NFR-22). |

### 9.5 Android background weighing

At pairing, Forkcast associates the scale through the Companion Device Manager. When the scale advertises, Android wakes the companion service, which starts a connectedDevice foreground service that reads the scale in Kotlin.

Each adapter has a Kotlin detector that applies the placement rule (section 9.4) to grams and stable, and posts the notification. No JavaScript runs, so it works after Android stops the app's process. On open, the service releases the scale to react-native-ble-plx; it stops when the scale turns off.

### 9.6 Android permissions

| Permission | Why | When |
|---|---|---|
| BLUETOOTH_SCAN (neverForLocation) | Find scales on the Add scale screen | Asked at Add scale |
| BLUETOOTH_CONNECT | Connect to paired scales and read weights | Asked at Add scale |
| POST_NOTIFICATIONS | The weighing notification (Android 13 and later) | Asked when Notify when weighing is turned on |
| CAMERA | Scan barcodes | Asked at the first scan |
| FOREGROUND_SERVICE, FOREGROUND_SERVICE_CONNECTED_DEVICE | Run the connectedDevice service while a scale is on | Granted at install |
| REQUEST_OBSERVE_COMPANION_DEVICE_PRESENCE | Learn when a paired scale appears and disappears | Granted at install |
| REQUEST_COMPANION_START_FOREGROUND_SERVICES_FROM_BACKGROUND | Start the service from the background when the scale appears | Granted at install |
| INTERNET | FDC lookups and Google Drive backup | Granted at install |

Forkcast asks for no location permission. Companion pairing may need Location Services switched on; verify on the test phone.

### 9.7 Storage, backup file, and Android backup

The database is expo-sqlite with SQLCipher (config plugin), keyed on first launch from the Android Keystore. It and its key are kept out of Auto Backup and device transfer (NFR-06): android:allowBackup is false and the data extraction rules exclude them, since on Android 12+ allowBackup alone may not.

| Part | Backup file |
|---|---|
| Content | JSON of every record, never the database file or key. Restore writes records into the phone's own database. |
| Header | forkcast-backup, format version 1, schema version, app version, created time. Newer format or schema is refused. |
| Integrity | A SHA-256 checksum of the content, checked before anything is restored. |
| Encryption | Optional AES-256-GCM, key from PBKDF2-HMAC-SHA256 (600,000 iterations, random 16-byte salt in the header). A wrong passphrase restores nothing. |
| File | Compressed with gzip and named Forkcast-backup-YYYY-MM-DD.forkcast. Drive keeps the newest file and the one before it. |

Cloud backup uses react-native-cloud-storage with Drive's hidden app data folder (drive.appdata). Export and import use the share sheet and file picker.

### 9.8 Tracked nutrients and Daily Values

Forkcast tracks energy and 43 nutrients. Daily Values are FDA values for ages 4 and up (21 CFR 101.9), kept as one data table. No DV means tracked with no default goal.

| Group | Nutrients and Daily Values |
|---|---|
| Energy | Energy, kcal (no DV) |
| Fats | Total fat 78 g; saturated fat 20 g, limit; trans fat, no DV; polyunsaturated fat, no DV; monounsaturated fat, no DV; cholesterol 300 mg, limit |
| Carbohydrate and protein | Total carbohydrate 275 g; dietary fiber 28 g; soluble fiber, no DV; insoluble fiber, no DV; total sugars, no DV; added sugars 50 g, limit; sugar alcohol, no DV; protein 50 g |
| Minerals | Sodium 2,300 mg, limit; potassium 4,700 mg; calcium 1,300 mg; iron 18 mg; phosphorus 1,250 mg; magnesium 420 mg; zinc 11 mg; selenium 55 mcg; copper 0.9 mg; manganese 2.3 mg; chromium 35 mcg; molybdenum 45 mcg; chloride 2,300 mg; iodine 150 mcg; fluoride, no DV |
| Vitamins | Vitamin A 900 mcg RAE; vitamin C 90 mg; vitamin D 20 mcg; vitamin E 15 mg; vitamin K 120 mcg; thiamin 1.2 mg; riboflavin 1.3 mg; niacin 16 mg NE; vitamin B6 1.7 mg; folate 400 mcg DFE; vitamin B12 2.4 mcg; biotin 30 mcg; pantothenic acid 5 mg; choline 550 mg |

### 9.9 Food data: USDA FoodData Central

FDC is the only food source: Foundation Foods and SR Legacy for generic foods, Branded Foods (with GTIN/UPC) for packaged foods. The free API allows 1,000 requests per hour per IP and blocks the key for an hour beyond that. Data is CC0 with a requested citation (FR-83). Restaurant foods are custom (FR-23).

Key: supplied at build time, never committed; on 403 or 429 the app falls back per NFR-27 and offers a personal api.data.gov key (FR-14). The risk of the built-in key being extracted and revoked is accepted.

Considered, not chosen: Nutritionix (paid per user), FatSecret (barcodes and restaurants only in Premier), and Edamam (paid). Open Food Facts (ODbL, attribution and share-alike) stays a possible future fallback for barcodes FDC lacks.

### 9.10 Camera and notifications

expo-camera reads the FR-15 formats. UPC-E is expanded to UPC-A and tried with and without leading zeros, since FDC may store either (to verify). The weighing notification is local and deep-links to live weighing.

### 9.11 Google Play release

Checked against Google Play Console Help and Google Cloud help on October 22, 2026. Items marked verify must be checked again in Play Console before release.

| Item | Requirement for Forkcast |
|---|---|
| Target API level | New apps and app updates must target Android 16 (API level 36) from August 31, 2026; an extension to November 1, 2026 could be requested. |
| Health apps declaration | Required for every app, including testing tracks. Forkcast declares Nutrition and weight management. Verify follow-up questions. |
| Data safety form | Matches the policy: no server, FDC gets search text or barcodes, backups go to the user's Drive. Review on data-handling changes. |
| Privacy policy | A public URL (OD-12) in the Play listing and Settings > About, describing on-device storage, FDC queries, and Google Drive backup. |
| Foreground service declaration | Declare the connectedDevice type in Play Console with a short description and a video showing it runs only while a scale is on. Verify the current form. |
| Google sign-in for Drive | drive.appdata is non-sensitive, so no scope verification. Brand verification shows the name and logo on consent. In testing, sign-in warns unverified and allows 100 users. |
| Testing before production | Personal accounts created after November 13, 2023 need a closed test with 12+ testers for 14 days before production. Verify the account type. |

### 9.12 Sources

- USDA FDC: fdc.nal.usda.gov/api-guide and /data-documentation. FDA, Daily Value on the Nutrition and Supplement Facts Labels (fda.gov); 21 CFR 101.9.
- ESN00 notes: github.com/hertzg/metekcity (README and packages/esn00-packet) and dev.to/hertzg/hacking-ble-kitchen-scale-55io. nRF Connect for Mobile (Nordic Semiconductor).
- Bluetooth Weight Scale Service 1.0.1 (bluetooth.com). react-native-ble-plx: github.com/dotintent/react-native-ble-plx
- Android developer guides (developer.android.com): companion device pairing, foreground service types, Bluetooth permissions, and Auto Backup.
- Google Play Console Help: target API level requirements, Health apps declaration, Data safety, foreground service declaration, and testing requirements for new personal developer accounts. Google Cloud help: OAuth scopes and app verification.
- Expo SQLite (SQLCipher) and Expo Camera: docs.expo.dev. React Native Cloud Storage: react-native-cloud-storage.oss.kuatsu.de. OWASP Password Storage Cheat Sheet (PBKDF2 iterations).
- WCAG 2.1 (w3.org). Cronometer nutrient targets (support.cronometer.com), used as a reference for goals.
- Considered food sources: nutritionix.com/api, platform.fatsecret.com/api-editions, developer.edamam.com/food-database-api. Open Food Facts: world.openfoodfacts.org/data

## 10. Acceptance criteria

Each line is a test of done. The requirement IDs in brackets are what it verifies; Appendix A gives the reverse map.

- **AC-01:** Fresh install: onboarding shows the introduction and Daily Value note, every step skips, it takes under 2 minutes, and Today opens with default goals. About shows it again. [FR-06, FR-07, FR-08, FR-09, NFR-21]
- **AC-02:** On Android 12 or later, every feature works without signing in. Settings opens from Profile, and Goals from Settings and from the pinned goals. [FR-01, FR-02, FR-03, FR-04]
- **AC-03:** Each permission in section 9.6 is explained before Android asks. With Bluetooth denied, typed grams work; with the camera denied, barcode digits can be typed. [FR-05, FR-17]
- **AC-04:** On the test network, an FDC search for rolled oats shows labeled results within 3 seconds, and My foods results show within 300 ms, also offline. [FR-10, FR-11, NFR-14]
- **AC-05:** Typing “chicken breast” sends one FDC request, 400 ms after the last key. A simulated HTTP 429 shows a try-again message. [FR-13]
- **AC-06:** A viewed FDC food is in My foods and can be logged in airplane mode. [FR-12, NFR-03]
- **AC-07:** With FDC unreachable, or the built-in key rejected, search says so in one line, and My foods and custom foods still work. Entering a personal key in Settings > Food data brings FDC search back. [FR-14, NFR-27]
- **AC-08:** Scan a US product found in FDC Branded Foods: it appears, and works offline afterwards. Scan an unknown one: manual entry with that barcode is offered. [FR-15, FR-16]
- **AC-09:** A custom food with sodium left blank and fiber 0 stores sodium as unknown and fiber as 0. A restaurant item with its source link is logged by servings. Values save per serving with grams, per 100 g, or per serving only. [FR-18, FR-19, FR-20, FR-23, FR-27]
- **AC-10:** Fill in sodium on a food logged yesterday and confirm: yesterday's sodium includes it and loses its partial mark, and no other value of yesterday changes. [FR-22]
- **AC-11:** Edit a food's protein: yesterday's totals do not change. Archive it: it leaves search and stays on past entries. A barcode attached to a custom food finds it. [FR-21, FR-24]
- **AC-12:** With no scale paired, log 45 g of oats and 2 slices of bread. Each nutrient equals the per-gram or per-serving value times the amount, and the entries are under the right date and meal slot. [FR-25, FR-26, FR-27, FR-28, FR-29]
- **AC-13:** Kill the app right after Log: the entry is there, written within 1 second. [FR-25, NFR-01]
- **AC-14:** An entry logged at 11:50 PM stays on that date after the phone's time zone changes, and editing its date moves it to the chosen day. [FR-29, FR-63]
- **AC-15:** Check the ESN00 with nRF Connect and record its services. Pair it, turn it off and on with the app open, and it connects within 10 seconds. [FR-30, FR-31, FR-32, NFR-15]
- **AC-16:** For 50 g, 250 g, and 1,000 g, ESN00 readings match the display within resolution, and stable matches settling. Ounce mode still shows grams. App Tare zeroes the scale. [FR-30, FR-33]
- **AC-17:** With two paired scales, the one on is used, or the most recent if both are; the quick switch changes it. Rename and forget leave past entries. Battery shows when reported. [FR-32, FR-34, FR-36, FR-81]
- **AC-18:** A Weight Scale Service scale, or a simulated one in nRF Connect, gives readings in grams. [FR-35]
- **AC-19:** With the app open, placing 30 g opens the live weighing screen and a 1 g bump does not. Readings show within 200 ms, and macros follow the weight. [FR-37, FR-38, FR-39, NFR-13]
- **AC-20:** Log stays disabled while the weight is unstable and enables when it settles. Typing 50 g enables Log and logs 50 g. [FR-40]
- **AC-21:** Closing the screen with a stable weight asks first, then discards it. Logging a stable weight from My foods takes three taps or fewer. [FR-41, NFR-20]
- **AC-22:** Phone locked, even after removing the app from recents: weighing posts one notification within 5 seconds, and a tap shows the live weight within 5 seconds. Ignoring it leaves the database unchanged. [FR-42, FR-43, FR-44, FR-45, FR-46, NFR-02, NFR-18]
- **AC-23:** The foreground service notification shows only while the scale is on, and is gone within 30 seconds after it turns off. [NFR-19]
- **AC-24:** With notifications denied, Settings says background weighing is unavailable and weighing works with the app open. Notify when weighing, Auto-open, and the minimum weight change behavior as set. [FR-47, FR-85]
- **AC-25:** Weigh several: rice 180 g, chicken 150 g, broccoli 85 g by tare and add, plus a restaurant item by servings. Totals run, an item can be corrected, and Finish logs one entry each. [FR-48, FR-49, FR-50, FR-51]
- **AC-26:** Save the session as Usual lunch and log it the next day in one step, scaled by 1.5, as one entry per item. [FR-52, FR-53, FR-54]
- **AC-27:** A 2,400 g chili Recipe's 350 g portion has 350/2,400 of the ingredients' nutrients. Per-serving-only foods are refused. Rename, duplicate, edit, and delete work; edits leave past entries. [FR-55, FR-56, FR-57, FR-58, FR-59]
- **AC-28:** The Today screen shows the active scale and a Weigh button. Change days by swipe and by the buttons, jump by calendar, see entries by meal slot, edit an entry, and delete one with Undo. [FR-60, FR-61, FR-62, FR-63, FR-66]
- **AC-29:** The summary lists all 44 tracked nutrients in gray, green, or red with Under, Met, or Over in text. A 40-entry day shows within 500 ms with a year of data; a cold start reaches Today within 3 seconds. [FR-64, FR-65, NFR-16, NFR-17]
- **AC-30:** Default goals match section 9.8; Energy has none. Sodium is Stay under and turns Over. A pinned, reordered 120 g protein goal (160 g maximum) updates on Today, with other goals still checked. [FR-67, FR-68, FR-69, FR-70, FR-71, FR-73]
- **AC-31:** A logged food with no vitamin D value marks the vitamin D goal partial and counts as 0. [FR-72]
- **AC-32:** Every profile field can stay blank, only the current weight is kept, and changing the profile changes no goal. Gender offers no pregnancy or breastfeeding options. [FR-74, FR-75, FR-76, FR-77]
- **AC-33:** Food weight shows to 1 g or 0.1 oz with stored grams unchanged. Theme and meal times apply. About shows the version, policy link, and FDC citation. [FR-78, FR-79, FR-80, FR-83, FR-84]
- **AC-34:** Back up, delete the app, reinstall, and restore: everything returns and the last backup time shows. Interrupt a backup: the previous one still restores. [FR-82, FR-88, FR-89, FR-92, NFR-05]
- **AC-35:** Export with a passphrase and import on a second phone. A wrong passphrase, a damaged file, or a newer format version changes nothing. An export without a passphrase shows a warning. [FR-87, FR-88, FR-89, NFR-12]
- **AC-36:** With a daily schedule, opening the app after the scheduled time runs the backup. [FR-93]
- **AC-37:** An Android backup and device transfer test leaves the database out. A database copied in without its key opens the Restore, Import, or Start fresh choice, with no crash. [FR-90, NFR-06]
- **AC-38:** Delete all data asks first, offers to delete the Drive backups, then removes the database and its Keystore key and signs out of Drive. [FR-91]
- **AC-39:** Foods and logs from the earlier MVP appear after the first launch. [FR-94]
- **AC-40:** The database reports SQLCipher active, and a copied database file cannot be read. Data survives an app update. A forced migration failure returns to the copy and offers Export. [FR-86, NFR-04, NFR-09, NFR-29]
- **AC-41:** A forced write failure retries once, then shows the error with the grams and food kept. Full storage shows a message and writes nothing partial. A failed Google sign-in shows the reason and offers Export. [NFR-28, NFR-30]
- **AC-42:** A day's proxy capture shows only HTTPS to FDC (query and key) and Drive, nothing to analytics. No key is in the repository. [NFR-07, NFR-08, NFR-10, NFR-11]
- **AC-43:** TalkBack labels every control, announces only stable weights at most every 2 seconds, and reads goal status. Main screens work at 200% font, meet contrast, use 48 dp targets and swipe buttons, and reduce animation. [NFR-22, NFR-23, NFR-24, NFR-25, NFR-26]
- **AC-44:** The release targets API 36 with minSdk 31. Before the first Play release, the policy URL is live and the section 9.11 declarations are complete. [NFR-31, NFR-32]
- **AC-45:** A test adapter and food source need no weighing, goal, or logging changes. Only repositories touch the database, and records carry UUIDs, timestamps, and soft-delete. Unit tests run without Bluetooth or UI; detectors pass their adapters' packets. [NFR-33, NFR-34, NFR-35, NFR-36, NFR-37]

## 11. Future enhancements

- iOS from the same code, with iCloud Drive backup. iOS background BLE gives about 10 seconds per wake and does not relaunch a force-quit app.
- Week view and history across days, for trends.
- Personalized targets: DRI defaults from age and gender (RDA or Adequate Intake minimums, Tolerable Upper Intake Level maximums), with no energy estimate.
- Optional accounts (Google and email sign-in) and cloud sync, planned with Supabase, built on the sync-ready data layer (NFR-36).
- Open Food Facts as a barcode fallback, more scale adapters, Health Connect, and photo recognition.

## 12. Interface sketch

Each row names the labeled parts a drawn sketch should show.

| Screen | Labeled parts |
|---|---|
| Onboarding | Welcome (on-device, no account), Pair a scale, Units, Profile, Default goals and pin, Skip on every step. |
| Today | Date (swipe, previous, next, calendar); scale chip; Weigh; Weigh several; pinned goals; meal slot lists (food, amount, kcal); nutrient summary. |
| Live weighing | Grams readout, Stable label, scale switch, Tare, grams or servings toggle, food chip, macros, Search, Scan, My foods, Log, Weigh several. |
| Weighing notification | Something is being weighed, with the scale name; a tap opens Live weighing. |
| Session | Running totals, item rows (food, grams or servings), Add next item, Finish, Save as Meal, Save as Recipe. |
| Food search | Search box, Scan, My foods and FDC tabs, rows with name, brand, and source (FDC type, Restaurant, or Recipe). |
| Goals | Nutrient list with the default DV or custom target, Stay under label for limits, min and max, Pin and reorder. |
| Profile | Name, Age, Height, Current weight, Gender (all optional), Settings button. |
| Settings | Units, Theme, Meal times, Scale behavior, Scales, Food data, Backup and data (Back up, Restore, Export, Import, Delete all), Goals, About. |

## Appendix A. Traceability

Every functional requirement maps to at least one use case and one acceptance criterion, and every non-functional requirement to an acceptance criterion. The table reads down the left half, then down the right half.

| ID | Priority | Use cases | Acceptance | ID | Priority | Use cases | Acceptance |
|---|---|---|---|---|---|---|---|
| FR-01 | Must | UC-01 | AC-02 | FR-67 | Should | UC-08 | AC-30 |
| FR-02 | Must | UC-01, UC-08 | AC-02 | FR-68 | Must | UC-08 | AC-30 |
| FR-03 | Must | UC-01, UC-08 | AC-02 | FR-69 | Must | UC-08 | AC-30 |
| FR-04 | Must | UC-01 | AC-02 | FR-70 | Must | UC-08 | AC-30 |
| FR-05 | Must | UC-01, UC-04 | AC-03 | FR-71 | Must | UC-08 | AC-30 |
| FR-06 | Should | UC-01 | AC-01 | FR-72 | Must | UC-08 | AC-31 |
| FR-07 | Should | UC-01 | AC-01 | FR-73 | Should | UC-08 | AC-30 |
| FR-08 | Should | UC-01 | AC-01 | FR-74 | Could | UC-01, UC-10 | AC-32 |
| FR-09 | Could | UC-10 | AC-01 | FR-75 | Could | UC-01, UC-10 | AC-32 |
| FR-10 | Must | UC-02 | AC-04 | FR-76 | Could | UC-10 | AC-32 |
| FR-11 | Must | UC-02 | AC-04 | FR-77 | Could | UC-10 | AC-32 |
| FR-12 | Must | UC-02 | AC-06 | FR-78 | Must | UC-01, UC-10 | AC-33 |
| FR-13 | Must | UC-02 | AC-05 | FR-79 | Must | UC-01, UC-10 | AC-33 |
| FR-14 | Should | UC-02, UC-10 | AC-07 | FR-80 | Must | UC-01, UC-10 | AC-33 |
| FR-15 | Should | UC-02 | AC-08 | FR-81 | Must | UC-04 | AC-17 |
| FR-16 | Should | UC-02 | AC-08 | FR-82 | Must | UC-11 | AC-34 |
| FR-17 | Should | UC-02 | AC-03 | FR-83 | Must | UC-10 | AC-33 |
| FR-18 | Must | UC-02 | AC-09 | FR-84 | Should | UC-10 | AC-33 |
| FR-19 | Must | UC-02 | AC-09 | FR-85 | Should | UC-06, UC-10 | AC-24 |
| FR-20 | Must | UC-02 | AC-09 | FR-86 | Must | UC-11 | AC-40 |
| FR-21 | Must | UC-02 | AC-11 | FR-87 | Must | UC-11 | AC-35 |
| FR-22 | Should | UC-02 | AC-10 | FR-88 | Must | UC-11 | AC-34, AC-35 |
| FR-23 | Should | UC-02 | AC-09 | FR-89 | Must | UC-11 | AC-34, AC-35 |
| FR-24 | Should | UC-02 | AC-11 | FR-90 | Must | UC-11 | AC-37 |
| FR-25 | Must | UC-03, UC-05 | AC-12, AC-13 | FR-91 | Must | UC-11 | AC-38 |
| FR-26 | Must | UC-03 | AC-12 | FR-92 | Should | UC-11 | AC-34 |
| FR-27 | Must | UC-03 | AC-09, AC-12 | FR-93 | Could | UC-11 | AC-36 |
| FR-28 | Must | UC-03 | AC-12 | FR-94 | Could | UC-01 | AC-39 |
| FR-29 | Must | UC-03 | AC-12, AC-14 | NFR-01 | Must | UC-03 | AC-13 |
| FR-30 | Must | UC-04 | AC-15, AC-16 | NFR-02 | Must | UC-06 | AC-22 |
| FR-31 | Must | UC-04 | AC-15 | NFR-03 | Must | UC-03 | AC-06 |
| FR-32 | Must | UC-04 | AC-15, AC-17 | NFR-04 | Must | — | AC-40 |
| FR-33 | Must | UC-04 | AC-16 | NFR-05 | Must | UC-11 | AC-34 |
| FR-34 | Should | UC-04 | AC-17 | NFR-06 | Must | UC-11 | AC-37 |
| FR-35 | Could | UC-04 | AC-18 | NFR-07 | Must | — | AC-42 |
| FR-36 | Could | UC-04 | AC-17 | NFR-08 | Must | — | AC-42 |
| FR-37 | Must | UC-05 | AC-19 | NFR-09 | Must | — | AC-40 |
| FR-38 | Must | UC-05 | AC-19 | NFR-10 | Must | — | AC-42 |
| FR-39 | Must | UC-05 | AC-19 | NFR-11 | Must | — | AC-42 |
| FR-40 | Must | UC-05 | AC-20 | NFR-12 | Should | UC-11 | AC-35 |
| FR-41 | Must | UC-05 | AC-21 | NFR-13 | Must | UC-05 | AC-19 |
| FR-42 | Must | UC-06 | AC-22 | NFR-14 | Must | UC-02 | AC-04 |
| FR-43 | Should | UC-06 | AC-22 | NFR-15 | Must | UC-04 | AC-15 |
| FR-44 | Should | UC-06 | AC-22 | NFR-16 | Should | — | AC-29 |
| FR-45 | Should | UC-06 | AC-22 | NFR-17 | Should | UC-09 | AC-29 |
| FR-46 | Should | UC-06 | AC-22 | NFR-18 | Should | UC-06 | AC-22 |
| FR-47 | Should | UC-06 | AC-24 | NFR-19 | Should | UC-06 | AC-23 |
| FR-48 | Should | UC-07 | AC-25 | NFR-20 | Should | UC-05 | AC-21 |
| FR-49 | Should | UC-07 | AC-25 | NFR-21 | Should | UC-01 | AC-01 |
| FR-50 | Should | UC-07 | AC-25 | NFR-22 | Must | UC-05 | AC-43 |
| FR-51 | Should | UC-07 | AC-25 | NFR-23 | Must | — | AC-43 |
| FR-52 | Should | UC-07 | AC-26 | NFR-24 | Must | — | AC-43 |
| FR-53 | Should | UC-07 | AC-26 | NFR-25 | Must | UC-09 | AC-43 |
| FR-54 | Should | UC-07 | AC-26 | NFR-26 | Should | — | AC-43 |
| FR-55 | Should | UC-07 | AC-27 | NFR-27 | Must | UC-02 | AC-07 |
| FR-56 | Should | UC-07 | AC-27 | NFR-28 | Must | — | AC-41 |
| FR-57 | Should | UC-07 | AC-27 | NFR-29 | Must | — | AC-40 |
| FR-58 | Should | UC-07 | AC-27 | NFR-30 | Should | UC-11 | AC-41 |
| FR-59 | Should | UC-07 | AC-27 | NFR-31 | Must | — | AC-44 |
| FR-60 | Must | UC-09 | AC-28 | NFR-32 | Must | — | AC-44 |
| FR-61 | Must | UC-09 | AC-28 | NFR-33 | Must | — | AC-45 |
| FR-62 | Must | UC-09 | AC-28 | NFR-34 | Must | — | AC-45 |
| FR-63 | Must | UC-09 | AC-14, AC-28 | NFR-35 | Must | — | AC-45 |
| FR-64 | Must | UC-09 | AC-29 | NFR-36 | Should | — | AC-45 |
| FR-65 | Must | UC-09 | AC-29 | NFR-37 | Should | — | AC-45 |
| FR-66 | Should | UC-09 | AC-28 |  |  |  |  |
