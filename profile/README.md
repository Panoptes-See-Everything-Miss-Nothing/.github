<p align="center">
  <img src="panoptes-logo.png" alt="Panoptes Logo" width="200">
</p>

<h1 align="center">Panoptes</h1>
<p align="center"><em>See Everything. Miss Nothing.</em></p>

<p align="center">
  <a href="https://en.cppreference.com/w/cpp/20"><img src="https://img.shields.io/badge/C%2B%2B-20-blue.svg" alt="C++20"></a>
  <a href="https://www.microsoft.com/windows"><img src="https://img.shields.io/badge/platform-Windows%2010%2B-0078d4.svg" alt="Platform"></a>
  <a href="#build"><img src="https://img.shields.io/badge/arch-x64%20%7C%20x86-green.svg" alt="Architecture"></a>
  <a href="#license"><img src="https://img.shields.io/badge/license-GPLv3-lightgrey.svg" alt="License"></a>
</p>

---

# Introduction

**Panoptes** is a community-driven vulnerability management platform built to eliminate blind spots that traditional scanners leave behind.

If you've ever asked:

> *"How did this vulnerability exist on the machine when the scanner said it was clean?"*

Panoptes exists because of that question.

---


## Project Status

- **Spectra Windows sensor:** Production-capable and actively maintained  
- **Iris backend:** Under active development  
- **Linux/macOS sensors:** Planned  

---

# Table of Contents

- [Why Panoptes?](#why-panoptes)
- [Architecture Overview](#architecture-overview)
- [Features](#features)
- [System Requirements](#system-requirements)
- [Build](#build)
- [Usage](#usage)
- [Core Contributors](#core-contributors)
- [Contributing](#contributing)
- [License](LICENSE)

---

# Architecture Overview

Panoptes is modular:

- **Spectra** → Sensor layer (endpoint inventory collection)  
- **Iris** → Backend correlation and intelligence engine  
- *(Future)* Web UI / API / Database components  

```
┌─────────────────────────────────────────────────────────┐
│                    Panoptes Platform                    │
│           See Everything. Miss Nothing.                 │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   Spectra    │  │   Spectra    │  │   Spectra    │  │
│  │   Windows    │  │    Linux     │  │    macOS     │  │
│  │  (this repo) │  │  (planned)   │  │  (planned)   │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
│         │                 │                 │          │
│         └────────┬────────┴────────┬────────┘          │
│                  │                 │                    │
│           ┌──────▼──────┐  ┌──────▼──────┐             │
│           │  Ingestion  │  │  Database   │             │
│           │  Pipeline   │──▶│  (findings) │             │
│           └─────────────┘  └──────┬──────┘             │
│                                   │                    │
│                            ┌───────▼───────┐             │
│                            │    Iris     │             │
│                            │  (backend)  │             │
│                            │  NVD match  │             │
│                            └───────┬───────┘             │
│                                   │                    │
│                            ┌───────▼───────┐             │
│                            │  Dashboard  │             │
│                            │  (frontend) │             │
│                            └─────────────┘             │
│                                                         │
└─────────────────────────────────────────────────────────┘

**Spectra Sensor for Windows**

┌─────────────────────────────────────────────────────┐
│                  Main.cpp (Entry Point)             │
│   /install · /upgrade · /uninstall · /console · SCM │
└────────┬────────────────────────────┬───────────────┘
         │                            │
   ┌─────▼──────┐            ┌───────▼────────┐
   │  Service    │            │  Console Mode  │
   │  Framework  │            │  (one-shot)    │
   └─────┬──────┘            └───────┬────────┘
         │                            │
   ┌─────▼────────────────────────────▼───────┐
   │          Data Collection Engine           │
   │  ┌────────────┐  ┌───────────────────┐   │
   │  │ Win32Apps   │  │ WinAppXPackages   │   │
   │  │ MsiApps     │  │ AppXPackages      │   │
   │  │ InstalledUpdates │  │ WindowsServices   │   │
   │  │ OsVersion   │  │ ProcessTracker    │   │
   │  │ MachineInfo ... │  │ UserProfiles ...     │   │
   │  └────────────┘  └───────────────────┘   │
   └──────────────────┬───────────────────────┘
                      │
              ┌───────▼────────────┐
              │  JSON Output        │
              │ inventory.json      │
              │ processes.json      │
              │ mspt_inventory.json │
              └─────────────────────┘
```
---

### Spectra Sensor for Windows

#### Features

- Deep system inventory (Win32/MSI/AppX, processes, services, updates)  
- Artefact collection even when patch data is missing  
- ETW-based process tracking  
- JSON output for ingestion into SIEM, data lakes, or analytics pipelines  
- Supports querying and CVE correlation (once Iris backend is ready)
- The application is a single native C++ executable (`Panoptes-Spectra.exe`) that can run as a **Windows Service** (periodic collection) or in **console mode** (one-shot collection).
- Uses Volume Shadow Copy (VSS) to safely mount and inspect offline registry hives to perform full per-user inventory even for inactive or logged-out accounts.

#### Using Spectra Today

> **Note:** Iris is currently under active development. In the meantime, the structured JSON inventory produced by Spectra can be ingested into your existing SIEM, data lake, CMDB, or analytics pipeline.  
> This allows you to immediately query your environment for affected versions, analyse CVE exposure, and build custom correlation logic — even before the full Panoptes backend is deployed.

---


### 4. Artefact Collection Even When Patch Data Is Missing

If patch information cannot be determined:

Spectra still collects artefacts answering:

- On which systems is this application present?  
- How many instances exist?  
- Where is it running?  
- What signals are available?  

These artefacts can be used in multiple ways:

- Community members (or in-house teams) can create reusable detection rules based on them.  
- Security teams can run their own queries against inventory data — either within existing data sources (by ingesting Spectra JSON) or, once Iris backend is available, via Iris.

---

# Iris (Backend Correlation Engine)

**Iris** correlates Spectra inventory data against:

- NVD  
- Vendor advisories  
- Patch Tuesday releases  
- Other vulnerability sources  

Instead of signature-per-CVE, Iris:

- Correlates versions automatically  
- Maps inventory against vulnerability ranges  
- Reduces rule-per-CVE detection  

> One intelligent rule per application, not one rule per CVE.

---

# Core Contributors

## Vaibhav Kakade
- 💼 [![LinkedIn](https://img.shields.io/badge/LinkedIn-Vaibhav%20Kakade-0A66C2?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/vgkakade/)
- 𝕏 [![X](https://img.shields.io/badge/X-@vk_appledore-000000?logo=x&logoColor=white)](https://x.com/vk_appledore)
- 🧑‍💻 [![GitHub](https://img.shields.io/badge/GitHub-vkappledore-181717?logo=github&logoColor=white)](https://github.com/vkappledore/)

## Sanoop Thomas
- 💼 [![LinkedIn](https://img.shields.io/badge/LinkedIn-s4n7h0-0A66C2?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/s4n7h0/)
- 𝕏 [![X](https://img.shields.io/badge/X-@s4n7h0-000000?logo=x&logoColor=white)](https://x.com/s4n7h0)
- 🧑‍💻 [![GitHub](https://img.shields.io/badge/GitHub-s4n7h0-181717?logo=github&logoColor=white)](https://github.com/s4n7h0/)

## Narendra Shinde
- 💼 [![LinkedIn](https://img.shields.io/badge/LinkedIn-narendrashinde-0A66C2?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/narendrashinde/)
- 𝕏 [![X](https://img.shields.io/badge/X-@nushinde-000000?logo=x&logoColor=white)](https://x.com/nushinde)
- 🧑‍💻 [![GitHub](https://img.shields.io/badge/GitHub-Nushinde-181717?logo=github&logoColor=white)](https://github.com/Nushinde)

## Kapil Khot
- 💼 [![LinkedIn](https://img.shields.io/badge/LinkedIn-Kapil%20Khot-0A66C2?logo=linkedin&logoColor=white)](https://www.linkedin.com/in/kapil-khot-50466952/)
- 𝕏 [![X](https://img.shields.io/badge/X-@kapil_khot-000000?logo=x&logoColor=white)](https://x.com/kapil_khot)
- 🧑‍💻 [![GitHub](https://img.shields.io/badge/GitHub-SlidingWindow-181717?logo=github&logoColor=white)](https://github.com/SlidingWindow)
---

# Contributing

Community contributions are welcome.

If you have:

- Detection artefacts  
- Version mapping improvements  
- Edge-case installation samples  
- Performance optimisations  
- API improvements
- Test bed and/or test cases

Open an issue or submit a pull request.

Let’s build something that actually sees everything.

## System Requirements

| Requirement | Details |
|---|---|
| **OS** | Windows 10 / Windows Server 2016 or later |
| **Architecture** | x64 (primary) or x86 (32-bit on 32-bit OS only) |
| **Runtime privileges** | `LocalSystem` (NT AUTHORITY\SYSTEM) — required for `SE_BACKUP_NAME`, `SE_RESTORE_NAME`, and kernel ETW sessions |
| **Installation privileges** | Local Administrator |
| **Disk space** | ~50 MB for the application plus variable space for output data |
| **Dependencies** | None — all functionality is implemented using native Windows APIs |

---
