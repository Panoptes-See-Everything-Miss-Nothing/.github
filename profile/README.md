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
- [Installation](#installation)
- [Configuration](#configuration)
- [Output](#output)
- [Security](#security)
- [Project Structure](#project-structure)
- [Documentation](#documentation)
- [Core Contributors](#core-contributors)
- [Contributing](#contributing)
- [License](#license)

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

