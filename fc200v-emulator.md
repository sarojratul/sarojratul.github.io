---
layout: project
title: FC-200V Calculator Emulator
permalink: /fc200v-emulator/
series: fc200v-emulator
description: A native Android emulator of the CASIO FC-200V financial calculator, built with Claude Code from the official manual. Project overview and build log.
lede: The CASIO FC-200V I ordered for my Engineering Economics exams was due to arrive the night before the midterm. To practise in the meantime, I built a native Android emulator of it with Claude Code, using the calculator's manual as the specification, and tested it against the manual's worked examples. It is now being prepared for Google Play, with a public goal to cover the developer fee and then drop ads for everyone.
hero: /images/fc200v_themes.jpg
hero_alt: The emulator in its dark, light and classic CASIO themes
hero_position: center 30%
facts:
  - label: Platform
    value: Android 7.0+ (API 24)
  - label: Language
    value: Java, custom views
  - label: Spec
    value: CASIO FC-200V manual
  - label: Tests
    value: 19 JUnit, all pass
  - label: Backend
    value: Firebase Firestore
  - label: Stage
    value: Waiting on Google verification
---

## Where it stands

*As of 1 November 2026.* The emulator covers every FC-200V mode and matches the manual's examples. The Google Play version is code-complete: a live fee goal, ads that switch off at the goal, a rewarded break from the banner, the EU consent form and a CA$1.49 Premium purchase. Publishing is paused while Google verifies my developer, AdMob and payments accounts.

| Work | Status |
|---|---|
| Calculator engine and UI, all modes | Done, [Part 0](/fc200v-emulator/part-0-why-and-the-native-rebuild/#native-rebuild-from-the-manual) |
| Tests against the manual's examples | Done, 17 then 19 tests, [Part 0](/fc200v-emulator/part-0-why-and-the-native-rebuild/#testing-against-the-manual) |
| Public fee goal and master ad switch | Done, [Part 1](/fc200v-emulator/part-1-fee-goal-ads-and-billing/#firestore-goal-and-master-ad-switch) |
| Banner and 5-minute rewarded break | Done, tested with test ads, [Part 1](/fc200v-emulator/part-1-fee-goal-ads-and-billing/#banner-and-the-rewarded-break) |
| EU consent and Play Billing | Code done; purchase untested, [Part 1](/fc200v-emulator/part-1-fee-goal-ads-and-billing/#consent-and-play-billing) |
| Google account verification | **Waiting** |
| Signing key, internal testing, store listing | Not started, [list in Part 1](/fc200v-emulator/part-1-fee-goal-ads-and-billing/#where-it-stopped) |
| Side-by-side check with the physical FC-200V | Not started |

## Build log

{% include build-log.html series="fc200v-emulator" %}

## How it is built

| | |
|---|---|
| **App** | Plain Java, custom-drawn LCD and keys, no UI framework beyond the Android SDK; 427 KB release APK before the Play features |
| **Build** | Gradle 8.11.1, Android Gradle Plugin 8.7.3, JDK 21, minimum Android 7.0 |
| **Services** | Firebase Firestore (fee goal), Google Mobile Ads with the consent SDK, Google Play Billing |
| **Testing** | JUnit on the manual's examples; `adb` scripts that press keys on the Android emulator and screenshot the result |
| **Tooling** | Claude Code for all code, builds and emulator runs; a Gemini chat for console steps Claude Code cannot do |

## Fee goal

Google Play's developer registration is US$25, about CA$35. Ad revenue, PayPal donations and CA$1.49 Premium purchases all count toward it, and the app shows the live total. When it reaches CA$35.00, ads switch off for every user, permanently.
