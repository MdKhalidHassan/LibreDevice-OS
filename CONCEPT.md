# LibreDevice OS — সম্পূর্ণ Concept Document

**Version:** 1.0  
**Author:** Md Khalid Hassan, Bogra, Bangladesh  
**Year:** 2026  
**License:** CC BY-SA 4.0

---

## সূচিপত্র

1. [মূল দর্শন](#১-মূল-দর্শন)
2. [Android-এর মূল সমস্যা](#২-android-এর-মূল-সমস্যা)
3. [Hardware Architecture](#৩-hardware-architecture)
4. [Software Architecture](#৪-software-architecture)
5. [Permission System](#৫-permission-system)
6. [User Freedom ও Security-র Balance](#৬-user-freedom-ও-security-র-balance)
7. [Network Security](#৭-network-security)
8. [প্রচলিত নিরাপত্তা ঝুঁকি ও সমাধান](#৮-প্রচলিত-নিরাপত্তা-ঝুঁকি-ও-সমাধান)
9. [সামগ্রিক নিরাপত্তা মূল্যায়ন](#৯-সামগ্রিক-নিরাপত্তা-মূল্যায়ন)
10. [সম্ভাব্যতা বিশ্লেষণ](#১০-সম্ভাব্যতা-বিশ্লেষণ)
11. [সারসংক্ষেপ](#১১-সারসংক্ষেপ)

---

## ১. মূল দর্শন

LibreDevice OS তিনটি মৌলিক প্রশ্নের উত্তর থেকে জন্ম নিয়েছে:

- PC-তে যেমন ইচ্ছামতো OS দেওয়া যায়, Android-এ কেন সম্ভব নয়?
- Google এবং অন্যান্য কোম্পানি আমাদের device থেকে data নেয় — এটা কি ঠেকানো সম্ভব?
- Security মানে কি user-এর freedom কেড়ে নেওয়া?

### পাঁচটি মূলনীতি

| নীতি | মানে |
|------|------|
| **Transparency** | Device-এ কী হচ্ছে সবসময় user দেখতে পাবে — কোনো কিছু লুকানো থাকবে না |
| **Freedom** | User যা খুশি তাই করতে পারবে — OS কোনো বাধা দেবে না |
| **Informed Choice** | OS warn করবে, বাধা দেবে না — Risk জানাবে, সিদ্ধান্ত user-এর |
| **Privacy by Default** | Default state সবসময় secure — User চাইলে নিজে খুলে দিতে পারবে |
| **You Own Your Device** | Hardware থেকে software — সব তোমার। কোনো company-র নিয়ন্ত্রণ নেই |

> **"OS তোমার guardian নয় — তোমার সৎ বন্ধু। সে সত্যি কথা বলবে, তারপর তোমার সিদ্ধান্তকে সম্মান করবে।"**

---

## ২. Android-এর মূল সমস্যা

PC-তে Universal OS সম্ভব কারণ standardized BIOS/UEFI এবং x86 architecture।
Android-এ সমস্যার কারণ:

```
Hardware Vendor (Qualcomm/MediaTek)
        ↓ proprietary drivers
Device Manufacturer (Xiaomi/Samsung)
        ↓ heavily customized
Stock ROM (MIUI/OneUI)
        ↓ Google Play Services (system-level, always running)
User (কোনো control নেই)
```

| সমস্যা | কারণ | প্রভাব |
|--------|------|--------|
| Proprietary drivers | Qualcomm/MediaTek closed-source রাখে | Universal ROM সম্ভব নয় |
| Bootloader locked | Manufacturer নিয়ন্ত্রণ করে | Custom OS দেওয়া কঠিন |
| Google Play Services | System-level permission, সবসময় চলে | সব data Google-এ যায় |
| No user sovereignty | Company decide করে কী safe | User freedom নেই |

---

## ৩. Hardware Architecture

### ৩.১ Form Factor

LibreDevice OS-এর জন্য প্রস্তাবিত device হবে **tablet-size**।

**কারণ:**
- Modular hardware components বসানোর যথেষ্ট জায়গা
- External module connector-এর জন্য space সমস্যা নেই
- বড় battery দেওয়া যাবে
- Physical security switch-এর জন্য side panel-এ জায়গা
- Desktop-class processor ব্যবহার সম্ভব
- Passive cooling সম্ভব

### ৩.২ Modular Hardware Design

Traditional phone-এ সব কিছু একটি SoC-এ integrated — এটাই closed firmware সমস্যার মূল কারণ।
LibreDevice-এ প্রতিটি component আলাদা, replaceable module:

| Module | Standard | Driver | Open? |
|--------|----------|--------|-------|
| Modem (4G) | Osmocom-based | Open firmware | ✅ |
| Camera | UVC Standard | Driver-free | ✅ |
| GPS | Standard NMEA | Open driver | ✅ |
| WiFi/BT | ath9k/mt76 | Open driver | ✅ |
| Storage | NVMe/UFS | Standard | ✅ |
| Extra slot | User-defined | User-defined | ✅ |

প্রতিটি module আলাদাভাবে replace করা যাবে। Device কখনো সম্পূর্ণ "মরবে" না।

### ৩.৩ Physical Security Switch System

প্রতিটি sensitive hardware component-এর জন্য **dual-layer switch**:

```
Software Switch (OS level)
        AND
Hardware Switch (circuit level, physical press)
        =
Dual-layer physical security
```

> **মূলনীতি:** Hardware Switch OFF মানে circuit physically open।
> Software যাই বলুক, Kernel compromise হোক, Pegasus যাই করুক —
> hardware-এ কোনো voltage নেই, কাজ করা physically অসম্ভব।

#### Switch Type: Piano/DIP Switch

- Device-এর side বা back panel-এ **recessed position**-এ থাকবে
- Accidentally trigger হওয়া কার্যত অসম্ভব
- সব switch এক row-এ — একনজরে সব status দেখা যায়
- প্রতিটি switch-এর পাশে **LED indicator**
- Industrial-grade durability

#### AND Gate Logic

| H (Hardware) | S (Software) | A (Result) |
|-------------|-------------|-----------|
| 0 (OFF) | 0 (OFF) | ❌ বন্ধ |
| 0 (OFF) | 1 (ON) | ❌ বন্ধ — Software চাইলেও বন্ধ |
| 1 (ON) | 0 (OFF) | ❌ বন্ধ — OS permission নেই |
| 1 (ON) | 1 (ON) | ✅ চালু — দুটো সম্মতি তবেই |

#### Switch Panel Layout

```
Device side panel (recessed):

[ MIC 🔴 ]  [ CAM ⚫ ]  [ MODEM ⚫ ]  [ WiFi 🔴 ]  [ GPS ⚫ ]
   ON          OFF         OFF            ON          OFF

🔴 = LED জ্বলছে = hardware active
⚫ = LED বন্ধ = hardware physically dead
```

---

## ৪. Software Architecture

### ৪.১ Foundation — Linux-based Host OS

- Android-এর উপর নির্মিত নয় — সম্পূর্ণ **Linux kernel-based**
- Privacy-first design — কোনো proprietary telemetry নেই
- Open-source — যে কেউ verify, audit এবং contribute করতে পারবে
- **Reproducible build** — compile করলে same output আসবে (supply chain attack prevention)

### ৪.২ Microkernel Architecture

Traditional Linux monolithic kernel-এ একটি exploit মানে সব কিছু compromise।

| Traditional Monolithic Kernel | LibreDevice Microkernel |
|-------------------------------|------------------------|
| সব কিছু kernel-এ — বিশাল attack surface | Minimal core — ছোট attack surface |
| একটি exploit = সব কিছু compromise | একটি component compromise = বাকিগুলো safe |
| Driver crash = system crash | Driver service isolated — crash করলেও OS চলে |

### ৪.৩ Container-based Isolation

```
Linux Host OS
    ├── Container A: Trusted apps (Google-free, F-Droid)
    │   → সর্বোচ্চ trust, full network access
    │
    ├── Container B: Google-dependent apps (isolated)
    │   → Limited network, Container A-র data দেখতে পারে না
    │
    └── Container C: User-defined sandbox
        → Custom rules, user নিজে define করবে
```

প্রতিটি container-এর আলাদা: storage, network identity, hardware permission।

### ৪.৪ Android App Compatibility Layer

- Linux host-এর ভেতরে container হিসেবে Android compatibility layer চলবে
- **Google Play Services ছাড়া** — F-Droid বা APK দিয়ে app install
- Android apps Linux host-এর কোনো data দেখতে পারবে না
- Container-এর network আলাদাভাবে control করা যাবে

---

## ৫. Permission System

### ৫.১ Real-time Permission Model

Android-এ app install-এ permission দিলে চিরকাল ব্যবহার করে।
LibreDevice-এ **প্রতিটি action-এ real-time permission request**:

| Permission Type | User Action | Auto-proceed? |
|----------------|-------------|---------------|
| Allow Once | Optional cancel | ✅ ৩ সেকেন্ড পর |
| Allow Session | Button press | ❌ Manual |
| Allow Always | Conscious confirm + countdown | ❌ ১০ সেকেন্ড পর |
| Critical Action | "I ACCEPT" manually type | ❌ কখনো না |

### ৫.২ Sensitive Actions — সবসময় permission নেবে

- Screenshot — প্রতিটি attempt-এ, এমনকি malware trigger করলেও
- Screen recording
- Clipboard read ও write
- Location access — প্রতিটি request-এ আলাদাভাবে
- Contact read
- File access — folder-specific
- Network request — নতুন domain-এ
- Mic activation
- Camera activation
- অন্য container বা app-এর data access
- Background-এ চলার অনুমতি

### ৫.৩ Persistent Visual Indicator System

"Always" বা "Long-term" permission চালু থাকলে status bar-এ সবসময় visible:

```
🎤 (লাল) = Mic active
📷 (লাল) = Camera active
📍 (হলুদ) = Location active
🌐 (নীল) = Network (specific app)
📋 (কমলা) = Clipboard access
```

- **Tap করলে:** কে কতক্ষণ ধরে কী access করছে + সাথে সাথে বন্ধ করার option
- **Lock screen-এও** active permission দেখাবে
- **Timer দেখাবে** — কতক্ষণ ধরে permission চালু

### ৫.৪ Alert Fatigue Prevention

- Low risk → শুধু status bar-এ subtle indicator, popup নেই
- Medium risk → প্রথমবার full popup, পরেরবার শুধু indicator
- High/Critical → সবসময় full alert, কোনো shortcut নেই

---

## ৬. User Freedom ও Security-র Balance

### ৬.১ User Sovereignty Model

| Android যা করে | LibreDevice যা করবে |
|----------------|---------------------|
| "এটা করা যাবে না" — block | "এটা করলে এই হতে পারে" — warn |
| Company decide করে কী safe | User নিজে সিদ্ধান্ত নেয় |
| Security feature বন্ধ করা যায় না | Default ON, user চাইলে বন্ধ করতে পারে |
| Root করা মানে warranty void | Root access built-in option |

### ৬.২ Risk Level অনুযায়ী Alert System

**🟡 Low Risk**
```
⚠️ "এই কাজটি তোমার battery বেশি খরচ করবে"
[Continue]  [Cancel]
→ ৩ সেকেন্ড পর auto-continue
```

**🟠 Medium Risk**
```
🔶 "এই app তোমার location সবসময় দেখতে পাবে"
   "পরিণাম: তোমার movement track হতে পারে"
[বুঝলাম, Continue]  [Cancel]
→ user manually confirm করতে হবে
```

**🔴 High Risk**
```
🔴 "তুমি Root access দিতে যাচ্ছো"
পরিণাম:
• এই app তোমার সব data দেখতে পারবে
• System files বদলাতে পারবে
[আমি বুঝেছি, তবুও করবো]  [ফিরে যাও]
→ ১০ সেকেন্ড countdown তারপর button active
```

**☠️ Critical Risk**
```
☠️ "তুমি Verified Boot বন্ধ করতে যাচ্ছো"
পরিণাম:
• Device physically access পেলে যে কেউ malware install করতে পারবে
• তোমার সব data ঝুঁকিতে পড়বে
নিচের box-এ manually টাইপ করো: I ACCEPT
→ কোনো shortcut নেই
```

### ৬.৩ Security Feature ও Freedom-এর Balance

সব security feature **default ON** — কিন্তু user চাইলে Critical Alert-এর মাধ্যমে বন্ধ করতে পারবে।

> **"Security feature মানে জেল নয় — এটা default shield। User চাইলে shield নামিয়ে রাখতে পারবে — তবে সে জানবে shield ছাড়া সে দাঁড়িয়ে আছে।"**

---

## ৭. Network Security

### ৭.১ Per-Container Network Isolation

- প্রতিটি container-এর আলাদা network identity (virtual network interface)
- Container B (Google apps) → শুধু নির্দিষ্ট server-এ যেতে পারবে
- Container A → সম্পূর্ণ আলাদা network path, tracker blocked
- Host OS firewall (nftables) সব কিছু নিয়ন্ত্রণ করবে
- Per-app network control — নির্দিষ্ট app-কে internet দেওয়া বা বন্ধ করা

### ৭.২ DNS Security

- DNS-over-HTTPS বা DNS-over-TLS OS level-এ **forced**
- Per-container আলাদা DNS resolver
- DNS query VPN tunnel-এর বাইরে যাওয়া technically অসম্ভব
- DNS leak সম্পূর্ণ prevention

### ৭.৩ Traffic Protection

- Traffic shaping — predictable pattern disguise করা
- Decoy traffic — fake traffic mix করে correlation attack prevent
- IMSI Catcher detection algorithm
- 2G fallback disable — 2G-তে encryption নেই

---

## ৮. প্রচলিত নিরাপত্তা ঝুঁকি ও সমাধান

### ৮.১ Google Data Collection

**ঝুঁকি:** Google Play Services system-level permission নিয়ে background-এ সবসময় চলে। Location, contact, app usage — সব পাঠায়। Stock Android-এ ঠেকানো প্রায় অসম্ভব।

**LibreDevice সমাধান:**
- Host Linux OS-এ Google-এর কোনো existence নেই
- Google apps শুধু Container B-তে isolated
- Container B-র বাইরে Google কিছু দেখতে পারে না
- Google Play Services-কে system-level access দেওয়া হয় না

---

### ৮.২ Pegasus ও Zero-Click Spyware

**ঝুঁকি:** Pegasus শুধু message receive করলেই (zero-click) kernel-level access পায়। Camera, mic, encrypted messages — সব access। বন্ধ phone থেকেও data collect সম্ভব।

**LibreDevice সমাধান:**
- Microkernel — attack surface ব্যাপকভাবে কমানো
- Hardware kill switch — Kernel compromise হলেও circuit physically open
- Verified Boot — tampered kernel boot করবে না
- Container isolation — একটি container compromise হলে বাকিগুলো safe
- Modem physical kill switch — device invisible to cell tower

> **সৎ মূল্যায়ন:** Pegasus-কে ১০০% software দিয়ে ঠেকানো পৃথিবীর কোনো OS পারেনি। LibreDevice-এ hardware kill switch + microkernel + verified boot একসাথে থাকায় Pegasus ঢুকলেও যা করতে পারবে তা অত্যন্ত সীমিত হবে।

---

### ৮.৩ Evil Maid Attack

**ঝুঁকি:** Device রেখে গেলে কেউ physical access পেয়ে:
- Boot করে malware inject করতে পারে
- Hardware implant বসাতে পারে
- RAM freeze করে encryption key বের করতে পারে (Cold Boot)

**LibreDevice সমাধান:**
- **Verified Boot:** প্রতিটি boot-এ cryptographic signature check — tamper হলে boot করবে না
- **TPM Chip:** Unauthorized boot detect করলে encryption key destroy — data permanently inaccessible
- **Secure Enclave:** Key RAM-এ নেই — cold boot attack কাজ করবে না
- **Tamper-evident seal:** Device খোলা হলে permanently নষ্ট হয়
- **Anti-tamper sensor:** Light/voltage/accelerometer — trigger হলে TPM key destroy

> **User Freedom Note:** এই সব protection default ON। User চাইলে Critical Alert-এর মাধ্যমে বন্ধ করতে পারবে। যেমন: নিজে device repair করতে হলে "Repair Mode" enable করা যাবে।

---

### ৮.৪ Supply Chain Attack

**ঝুঁকি:** Factory থেকেই malware pre-installed। User কিছু না করলেও compromised।

**LibreDevice সমাধান:**
- সম্পূর্ণ open-source firmware — যে কেউ verify করতে পারবে
- Reproducible build — compile করলে same binary output আসবে
- Community audit সবসময় সম্ভব

---

### ৮.৫ Baseband Attack

**ঝুঁকি:** Modem-এর আলাদা processor (Baseband) main OS থেকে সম্পূর্ণ স্বাধীনভাবে চলে। Cell tower থেকে সরাসরি attack সম্ভব — OS কিছুই জানে না।

**LibreDevice সমাধান:**
- External modem module — physical kill switch দিয়ে সম্পূর্ণ power cut
- Modem isolated — main processor-এর সাথে strictly limited interface
- Osmocom-based open firmware — proprietary baseband-এর বিকল্প

---

### ৮.৬ IMSI Catcher / Stingray Attack

**ঝুঁকি:** নকল cell tower তোমার phone connect করিয়ে location track, call intercept, SMS read করতে পারে।

**LibreDevice সমাধান:**
- Modem kill switch — tower-ই খুঁজে পাবে না
- Software layer-এ IMSI catcher detection algorithm
- 2G fallback সম্পূর্ণ disable
- Suspicious tower alert

---

### ৮.৭ Clipboard Hijacking

**ঝুঁকি:** তুমি password copy করলে background app clipboard read করে নিতে পারে।

**LibreDevice সমাধান:**
- Clipboard access = explicit real-time permission — প্রতিবার
- Container A-র clipboard Container B দেখতে পারবে না
- Clipboard auto-clear — নির্দিষ্ট সময় পর automatically clear
- Clipboard access log

---

### ৮.৮ Screenshot ও Screen Recording Attack

**ঝুঁকি:** Malicious app screen content record করে banking app-এর password, OTP দেখতে পারে।

**LibreDevice সমাধান:**
- প্রতিটি screenshot attempt-এ real-time permission — malware trigger করলেও popup
- Sensitive app চলাকালীন screenshot API সম্পূর্ণ block
- Container isolation — অন্য container screen দেখতে পারবে না

---

### ৮.৯ DNS Leakage

**ঝুঁকি:** VPN চালু থাকলেও DNS query VPN-এর বাইরে যেতে পারে।

**LibreDevice সমাধান:**
- DNS-over-HTTPS বা DNS-over-TLS OS level-এ forced
- DNS query VPN tunnel-এর বাইরে যাওয়া technically অসম্ভব

---

### ৮.১০ Ultrasonic Tracking

**ঝুঁকি:** App ultrasonic signal play করে (শোনা যায় না), কাছের device mic দিয়ে cross-device tracking।

**LibreDevice সমাধান:**
- Mic hardware kill switch — সবচেয়ে effective
- Audio frequency filter — ultrasonic range (>20kHz) block

---

### ৮.১১ Side Channel Attack

**ঝুঁকি:** Power consumption pattern, CPU cache timing, electromagnetic emission থেকে encryption key বা typed content বের করা সম্ভব।

**LibreDevice সমাধান:**
- Constant-time cryptography implementation
- Power consumption randomization
- EMI shielding (hardware level)

> **সীমাবদ্ধতা:** Side channel attack আংশিকভাবে hardware-level সমস্যা। EMI shielding hardware design-এ যোগ করতে হবে।

---

### ৮.১২ Permission Creep

**ঝুঁকি:** App install-এ minimal permission, update-এ আরও permission — user খেয়াল করলো না।

**LibreDevice সমাধান:**
- Permission একবার দিলে permanent নয়
- Update-এ নতুন permission চাইলে explicit re-approval — mandatory
- Permission audit log — কে কখন কী access করলো
- Permission expiry — নির্দিষ্ট সময় পর automatically expire

---

### ৮.১৩ Traffic Correlation Attack

**ঝুঁকি:** Tor/VPN ব্যবহার করলেও traffic pattern দেখে identity বের করা সম্ভব।

**LibreDevice সমাধান:**
- Traffic shaping — pattern disguise
- Decoy traffic — fake traffic mix
- Random timing — predictable pattern ভাঙা

> **সীমাবদ্ধতা:** Traffic correlation attack সম্পূর্ণ eliminate করা এখনো research area।

---

## ৯. সামগ্রিক নিরাপত্তা মূল্যায়ন

| Attack Type | LibreDevice-এ Status | মূল Mechanism |
|-------------|---------------------|---------------|
| Google data collection | ✅ সম্পূর্ণ block | Container isolation |
| Supply Chain | ✅ সম্পূর্ণ | Open source + reproducible build |
| Baseband Attack | ✅ সম্পূর্ণ | Physical modem kill switch |
| IMSI Catcher/Stingray | ✅ সম্পূর্ণ | Kill switch + detection |
| Evil Maid | ✅ সম্পূর্ণ | Verified boot + TPM + tamper sensor |
| Clipboard Hijack | ✅ সম্পূর্ণ | Real-time permission + container isolation |
| Screen Recording | ✅ সম্পূর্ণ | Real-time permission + container isolation |
| DNS Leak | ✅ সম্পূর্ণ | OS-level DNS enforcement |
| Ultrasonic Track | ✅ সম্পূর্ণ | Mic kill switch + frequency filter |
| Permission Creep | ✅ সম্পূর্ণ | Re-approval + audit log |
| Pegasus/Zero-click | ⚠️ আংশিক | Microkernel + hardware kill switch |
| Side Channel | ⚠️ আংশিক | EMI shielding দরকার |
| Traffic Correlation | ⚠️ আংশিক | Traffic shaping, সম্পূর্ণ নয় |

---

## ১০. সম্ভাব্যতা বিশ্লেষণ

| Component | সম্ভব? | কারণ |
|-----------|--------|------|
| Linux-based secure OS | ✅ সম্পূর্ণ | Proven technology |
| Container-based isolation | ✅ সম্পূর্ণ | Linux-এর native capability |
| Android app compatibility | ✅ সম্পূর্ণ | Container approach কাজ করে |
| Modular hardware (tablet) | ✅ সম্পূর্ণ | Size constraint নেই |
| Open firmware (camera UVC) | ✅ সম্পূর্ণ | UVC standard-ই open |
| Physical security switches | ✅ সম্পূর্ণ | Simple circuit design |
| Per-container network isolation | ✅ সম্পূর্ণ | nftables/namespaces |
| Hardware permission control | ✅ সম্পূর্ণ | Kernel level possible |
| Real-time permission system | ✅ সম্পূর্ণ | Software implementation |
| Verified boot + TPM | ✅ সম্পূর্ণ | Existing technology |
| Open firmware (modem 4G) | ⚠️ আংশিক | Osmocom mature হচ্ছে |
| Open firmware (modem 5G) | ⚠️ ভবিষ্যৎ | এখনো research stage |
| Mass production | ⚠️ কঠিন | Fairphone প্রমাণ করেছে সম্ভব |
| Side channel (hardware) | ⚠️ আংশিক | EMI shielding design লাগবে |

---

## ১১. সারসংক্ষেপ

LibreDevice OS একটি কাল্পনিক কিন্তু technically sound concept। এর প্রতিটি component আলাদাভাবে বিদ্যমান প্রযুক্তিতে সম্ভব — শুধু একসাথে কেউ এভাবে build করেনি।

### প্রধান উদ্ভাবনগুলো

- **Hardware + Software dual-layer kill switch** — physics দিয়ে privacy রক্ষা
- **Container-based compartmentalization** — Google isolated, বাকি জীবন মুক্ত
- **Real-time permission + persistent visual indicator** — সম্পূর্ণ transparency
- **Modular external hardware** — closed firmware সমস্যার clean solution
- **User sovereignty model** — warn করবে, বাধা দেবে না

### এখনো যে চ্যালেঞ্জ বাকি

- 5G modem-এর open firmware — Osmocom এখনো mature হয়নি
- Side channel attack-এর hardware-level সমাধান — EMI shielding design
- Pegasus-type zero-click — সম্পূর্ণ prevention এখনো unsolved globally
- Mass production — কঠিন কিন্তু অসম্ভব নয়

---

> **"PC-এর freedom + Linux-এর security + Android-এর app ecosystem + Modular hardware + User sovereignty = প্রযুক্তি মানুষের জন্য, মানুষ প্রযুক্তির জন্য নয়।"**

---

*LibreDevice OS Concept Document | Version 1.0 | 2026*
*Conceptualized by MD khalid Hassan, Bogra, Bangladesh*
*License: CC BY-SA 4.0*
