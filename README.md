# LibreDevice OS

> **"OS তোমার guardian নয় — তোমার সৎ বন্ধু।**
> সে সত্যি কথা বলবে, তারপর তোমার সিদ্ধান্তকে সম্মান করবে।"

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC%20BY--SA%204.0-blue.svg)](https://creativecommons.org/licenses/by-sa/4.0/)
[![Concept Status](https://img.shields.io/badge/Status-Concept%20Paper-orange.svg)]()
[![Contributions Welcome](https://img.shields.io/badge/Contributions-Welcome-brightgreen.svg)]()

---

## এটা কী?

LibreDevice OS একটি **কাল্পনিক কিন্তু technically feasible** mobile operating system-এর concept — যেখানে:

- **PC-এর freedom** — যেকোনো hardware-এ install করা যাবে
- **Linux-এর security** — proven, open-source foundation
- **Android-এর app ecosystem** — container-এ isolated Android compatibility
- **Modular hardware** — প্রতিটি component replaceable, open firmware
- **User sovereignty** — তুমি device-এর মালিক, কোনো company নয়

---

## কেন এটা দরকার?

আজকের Android-এ:

- Google **OS-level access** পায় — ঠেকানো প্রায় অসম্ভব
- Manufacturer **bootloader lock** করে রাখে
- Closed-source driver-এর কারণে **universal custom ROM** সম্ভব নয়
- Security-র নামে **user freedom** কেড়ে নেওয়া হয়
- Pegasus-এর মতো spyware **বন্ধ phone থেকেও** data নিতে পারে

LibreDevice এই সমস্যাগুলো **root level থেকে** solve করার চেষ্টা।

---

## মূল দর্শন — পাঁচটি নীতি

| নীতি | মানে |
|------|------|
| **Transparency** | Device-এ কী হচ্ছে সবসময় user দেখতে পাবে |
| **Freedom** | User যা খুশি তাই করতে পারবে — OS বাধা দেবে না |
| **Informed Choice** | OS warn করবে, সিদ্ধান্ত user-এর |
| **Privacy by Default** | Default state সবসময় secure |
| **You Own Your Device** | Hardware থেকে software — সব তোমার |

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│              Physical Hardware Layer                │
│                                                     │
│  Piano/DIP Switches (recessed panel + LED):         │
│  [ MIC 🔴 ] [ CAM ⚫ ] [ MODEM ⚫ ] [ WiFi 🔴 ] [ GPS ⚫ ] │
│  Circuit-level cut — software bypass অসম্ভব        │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│            Modular Hardware Slots                   │
│  [Modem: Osmocom] [Camera: UVC] [GPS] [WiFi: open] │
│  প্রতিটি module আলাদাভাবে replaceable              │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│         Linux-based Host OS (Microkernel)           │
│  • Verified Boot + TPM                              │
│  • Full Disk Encryption + Secure Enclave            │
│  • Per-container Firewall (nftables)                │
│  • Real-time Permission System                      │
└──────┬───────────────┬───────────────┬──────────────┘
       │               │               │
┌──────▼─────┐  ┌──────▼─────┐  ┌─────▼──────┐
│Container A │  │Container B │  │Container C │
│ Trusted    │  │ Google/    │  │ User-      │
│ F-Droid    │  │ Isolated   │  │ defined    │
│ apps       │  │ apps       │  │ sandbox    │
└────────────┘  └────────────┘  └────────────┘
```

---

## Hardware Design

### Form Factor
Tablet-size device — phone-এর চেয়ে বড়। এই সিদ্ধান্তের কারণ:
- Modular component-এর জন্য যথেষ্ট space
- বড় battery — external modules-এর জন্য দরকার
- Physical security switch-এর জন্য side panel জায়গা
- Desktop-class processor সম্ভব

### Physical Kill Switch System — AND Gate Logic

```
Hardware Switch (H) AND Software Switch (S) = Hardware Active (A)

H=0, S=1 → A=0  ←  Software চাইলেও hardware বন্ধ
H=1, S=0 → A=0  ←  OS permission ছাড়া বন্ধ
H=1, S=1 → A=1  ←  দুটো সম্মতি তবেই চালু
```

Pegasus kernel compromise করলেও — circuit physically open — mic capture **physically impossible**.

### Modular Hardware

| Module | Standard | Open? |
|--------|----------|-------|
| Modem (4G) | Osmocom-based | ✅ |
| Camera | UVC Standard | ✅ |
| GPS | NMEA | ✅ |
| WiFi/BT | ath9k/mt76 | ✅ |
| Extra slot | User-defined | ✅ |

---

## Software Features

### Container-based Isolation
- Container A: Trusted apps (Google-free, F-Droid)
- Container B: Google-dependent apps (isolated, limited network)
- Container C: User-defined sandbox
- প্রতিটির আলাদা storage, network identity, hardware permission

### Real-time Permission System
প্রতিটি sensitive action-এ permission নেবে:
- Screenshot (malware trigger করলেও popup আসবে)
- Clipboard read/write
- Location, Camera, Mic
- Screen recording
- Background activity

### Persistent Visual Indicator
```
Status Bar সবসময়:
🎤 (লাল) = Mic active
📷 (লাল) = Camera active
📍 (হলুদ) = Location active

Tap → কে কতক্ষণ access করছে + সাথে সাথে বন্ধ করার option
```

### User Freedom Alert System

| Risk Level | Action | Behavior |
|------------|--------|----------|
| Low | Optional cancel | ৩ সেকেন্ড পর auto-proceed |
| Medium | Button press | Manual confirm |
| High | Conscious confirm | ১০ সেকেন্ড countdown |
| Critical | Type "I ACCEPT" | কোনো shortcut নেই |

---

## Security — Threat Coverage

| Attack | Status | Mechanism |
|--------|--------|-----------|
| Google data collection | ✅ Block | Container isolation |
| Supply Chain | ✅ | Open source + reproducible build |
| Baseband Attack | ✅ | Physical modem kill switch |
| IMSI Catcher/Stingray | ✅ | Kill switch + detection |
| Evil Maid | ✅ | Verified boot + TPM + tamper sensor |
| Clipboard Hijack | ✅ | Real-time permission |
| DNS Leak | ✅ | OS-level DNS enforcement |
| Ultrasonic Tracking | ✅ | Mic kill switch + frequency filter |
| Permission Creep | ✅ | Re-approval + audit log |
| Pegasus/Zero-click | ⚠️ Partial | Microkernel + hardware kill switch |
| Side Channel | ⚠️ Partial | EMI shielding দরকার |
| Traffic Correlation | ⚠️ Partial | Traffic shaping |

---

## এখনো যে চ্যালেঞ্জ বাকি

- **5G modem open firmware** — Osmocom এখনো mature হয়নি
- **Side channel (hardware)** — EMI shielding design
- **Pegasus zero-click** — সম্পূর্ণ prevention এখনো unsolved globally
- **Mass production** — কঠিন, কিন্তু Fairphone প্রমাণ করেছে সম্ভব

---

## Full Document

বিস্তারিত concept paper পড়তে:
- 📄 [CONCEPT.md](./CONCEPT.md) — সম্পূর্ণ বাংলা document
- 📥 [LibreDevice_OS_Concept.docx](./docs/LibreDevice_OS_Concept.docx) — Word format

---

## Contribute করতে চাও?

[CONTRIBUTING.md](./CONTRIBUTING.md) পড়ো।

এই project-এ সবার contribution স্বাগত — developer, security researcher, hardware engineer, বা যে কেউ যে মনে করে প্রযুক্তি মানুষের জন্য হওয়া উচিত।

---

## License

- **Concept & Documentation:** [Creative Commons BY-SA 4.0](./LICENSE-CC)
  — ব্যবহার করতে পারবে, নাম দিতে হবে, মুক্তই রাখতে হবে
- **Any derived code:** [GNU GPL v3](./LICENSE-GPL)
  — source code সবসময় মুক্ত থাকতে হবে

---

## Author

**Ojana** — Bogra, Bangladesh
*Electrical Engineering | Industrial Automation | Privacy Advocate*

> "প্রযুক্তি মানুষের জন্য, মানুষ প্রযুক্তির জন্য নয়।"
