# Awesome Data Center Technician Field Guide & Interview Preparation
### Hyperscale Hardware Triage, Optical Cabling, BMC/IPMI Diagnostics & Loop Interview Mastery

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![Amazon Live](https://img.shields.io/badge/Amazon-Paperback%20Live-orange?logo=amazon&style=for-the-badge)](https://www.amazon.com/dp/B0HP9JNGXL)
[![License](https://img.shields.io/badge/License-CC%20BY--NC%204.0-green?style=for-the-badge)](https://creativecommons.org/licenses/by-nc/4.0/)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=for-the-badge)](CONTRIBUTING.md)

An open-source technical reference for technicians, hardware engineers, and candidates preparing for **Data Center Operations Technician (DCO / DCT)** technical loop interviews at **Amazon Web Services (AWS), Google Cloud (GCP), Microsoft Azure, Meta, Equinix, and Digital Realty**.

---

## 📘 Physical Workbook Companion

[![Watch Video Preview](https://img.shields.io/badge/Video%20Preview-1080p%20Trailer-red?style=for-the-badge&logo=youtube)](./datacenter_technician_promo_1080p.mp4)

This repository provides open quick-reference summaries and cheat sheets. If you need physical bench drills with **18 full-length incident simulation labs, authentic BMC/IPMI hex telemetry logs, blank FACTS ticket forms, and complete model answer keys**, check out the official published manual:

👉 **[Data Center Operations Technician Field Workbook & Technical Practice Guide (Amazon ASIN: B0HP9JNGXL)](https://www.amazon.com/dp/B0HP9JNGXL)**  
*(8.5 x 11 inch workbench workbook, 104 pages, published by Hyperscale Infrastructure Labs)*

---

## Table of Contents
* [1. Physical Layer Cabling & Fiber Optics](#1-physical-layer-cabling--fiber-optics)
  * [1.1 TIA-598-D 12-Color Optical Fiber Code](#11-tia-598-d-12-color-optical-fiber-code)
  * [1.2 MPO/MTP Polarity Architectures (Method A vs B vs C)](#12-mpomtp-polarity-architectures)
  * [1.3 Digital Optical Monitoring (DOM) Diagnostics](#13-digital-optical-monitoring-dom-diagnostics)
  * [1.4 Fiber Endface Cleaning (IEC 61300-3-35)](#14-fiber-endface-cleaning-iec-61300-3-35)
* [2. Server Hardware Bootstrap & POST Isolation](#2-server-hardware-bootstrap--post-isolation)
  * [2.1 The 4 Bootstrap Essentials](#21-the-4-bootstrap-essentials)
  * [2.2 DIMM Memory Channel Fault Isolation](#22-dimm-memory-channel-fault-isolation)
* [3. Out-of-Band BMC, IPMI 2.0 & Redfish Reference](#3-out-of-band-bmc-ipmi-20--redfish-reference)
  * [3.1 Essential `ipmitool` CLI Commands](#31-essential-ipmitool-cli-commands)
  * [3.2 Decoding IPMI SEL Event Hex Codes](#32-decoding-ipmi-sel-event-hex-codes)
  * [3.3 Modern Redfish API Triage](#33-modern-redfish-api-triage)
* [4. AI Clusters & GPU Node Triage (NVIDIA SXM5 / PCIe)](#4-ai-clusters--gpu-node-triage)
  * [4.1 `nvidia-smi` Field Diagnostics](#41-nvidia-smi-field-diagnostics)
  * [4.2 Thermal Throttling & Air Containment](#42-thermal-throttling--air-containment)
* [5. Power Distribution & Safety Hierarchy (NFPA 70E / OSHA)](#5-power-distribution--safety-hierarchy)
  * [5.1 Strict Legal Order of Precedence](#51-strict-legal-order-of-precedence)
  * [5.2 PDU Formulas & IEC Connectors](#52-pdu-formulas--iec-connectors)
* [6. Top 5 Hyperscale Interview Scenarios & Model Answers](#6-top-5-hyperscale-interview-scenarios--model-answers)
* [7. The FACTS Incident Triage Framework](#7-the-facts-incident-triage-framework)

---

## 1. Physical Layer Cabling & Fiber Optics

### 1.1 TIA-598-D 12-Color Optical Fiber Code
The standard color sequence used across all hyperscale data halls for multi-fiber push-on (MPO) trunks, breakout harnesses, and loose-tube fiber cables:

| Pos # | Color Name | Code | Standard Mnemonic |
| :---: | :---: | :---: | :--- |
| **1** | **Blue** | `[BLU]` | **B**oys |
| **2** | **Orange** | `[ORG]` | **O**r |
| **3** | **Green** | `[GRN]` | **G**irls |
| **4** | **Brown** | `[BRN]` | **B**ring |
| **5** | **Slate (Gray)** | `[SLT]` | **S**ome |
| **6** | **White** | `[WHT]` | **W**hiskey |
| **7** | **Red** | `[RED]` | **R**ight |
| **8** | **Black** | `[BLK]` | **B**efore |
| **9** | **Yellow** | `[YEL]` | **Y**ou |
| **10** | **Violet** | `[VIO]` | **V**enture |
| **11** | **Rose (Pink)** | `[ROS]` | **R**ound |
| **12** | **Aqua** | `[AQU]` | **A**laska |

### 1.2 MPO/MTP Polarity Architectures
* **Method A (Straight-Through):** Key-Up to Key-Down. Fiber 1 connects to Fiber 1. Requires an A-to-B patch cord on one end and an A-to-A cord on the other.
* **Method B (Cross-Over / Reversed) — Hyperscale Standard:** Key-Up to Key-Up. Fiber 1 connects to Fiber 12, Fiber 2 to Fiber 11. Used in parallel optics (40GBASE-SR4, 100GBASE-SR4, 400G-DR4) because it maintains identical patch cables across all ports.
* **Method C (Pair-Wise Flip):** Key-Up to Key-Down. Pairs are flipped (1-2 becomes 2-1). Primarily used for duplex emulation across trunk backbones.

### 1.3 Digital Optical Monitoring (DOM) Diagnostics
When troubleshooting packet loss, CRC drops, or link flap on Linux:

```bash
# Query transceiver hardware, vendor, and link status
sudo ethtool -i eth0

# Read live optical Tx/Rx power levels, temperature, and supply voltage
sudo ethtool -m eth0

# Continuous monitoring during physical cable reseating
watch -n 1 "sudo ethtool -m eth0 | grep -E '(Receiver signal average optical power|Laser output power)'"
```

#### Optical Power Health Thresholds:
* **Tx Power:** Normal range: `-1.0 dBm` to `+1.0 dBm` (SR4/SR2) or `-3.0 dBm` to `+2.0 dBm` (LR/LR4).
* **Rx Power:** Healthy range: `-2.0 dBm` to `-9.0 dBm`.
* **Marginal Warning:** `-11.0 dBm` to `-14.0 dBm` (susceptible to packet drops).
* **Loss of Signal (LOS):** Below `-18.0 dBm` (link will drop).

### 1.4 Fiber Endface Cleaning (IEC 61300-3-35)
* 85% of physical layer failures in data centers stem from contaminated endfaces.
* **Procedure:** Inspect with digital optical microscope (400x) $\rightarrow$ Dry clean (One-Click cleaner, 2 clicks max) $\rightarrow$ Re-inspect.
* If residue persists: Wet-to-Dry clean with 99% optical-grade isopropyl alcohol on optical wipe. Never plug dirty fiber into a transceiver.

---

## 2. Server Hardware Bootstrap & POST Isolation

### 2.1 The 4 Bootstrap Essentials
Before a server can display video, load UEFI NVRAM, or initialize PCIe buses, the following 4 subsystems must reach hardware power-good:
1. **Auxiliary & Main Power Rails:** VRM supplying `Vcore`, `+12V`, `+5V`, `+3.3V`, and `VTT`.
2. **System Clock Generator:** Oscillator supplying baseline differential frequencies.
3. **Primary CPU (Socket 0):** Must fetch initial reset vector instructions from SPI ROM.
4. **Base System Memory (Channel 0 / Slot 0):** At least one functional DIMM in CPU 0, Slot A1.

### 2.2 DIMM Memory Channel Fault Isolation
* **Single-Bit Correctable ECC:** Handled transparently by hardware SECDED logic. Does not crash server. Replace if error rate exceeds threshold (e.g. 10/hour).
* **Multi-Bit Uncorrectable ECC:** Triggers an immediate Machine Check Exception (MCE), non-maskable interrupt (NMI), and kernel panic.
* **Triage Rule:** Use the motherboard's **Minimum Configuration for POST** (CPU 0 + Slot A1). Use the **Half-Population Method** across test boots to isolate bad memory banks.

---

## 3. Out-of-Band BMC, IPMI 2.0 & Redfish Reference

### 3.1 Essential `ipmitool` CLI Commands

```bash
# Verify BMC connectivity and firmware version
ipmitool -I lanplus -H <BMC_IP> -U <USER> -P <PASS> mc info

# View System Event Log (SEL)
ipmitool -I lanplus -H <BMC_IP> -U <USER> -P <PASS> sel list

# Extended SEL with hex sensor data
ipmitool -I lanplus -H <BMC_IP> -U <USER> -P <PASS> sel elist

# Clear SEL log after completing hardware swap
ipmitool -I lanplus -H <BMC_IP> -U <USER> -P <PASS> sel clear

# Read all physical sensors (temperatures, fans, voltages)
ipmitool -I lanplus -H <BMC_IP> -U <USER> -P <PASS> sdr elist

# Chassis Power Control
ipmitool -I lanplus -H <BMC_IP> -U <USER> -P <PASS> chassis power status
ipmitool -I lanplus -H <BMC_IP> -U <USER> -P <PASS> chassis power cycle
ipmitool -I lanplus -H <BMC_IP> -U <USER> -P <PASS> chassis power reset

# Activate Serial-over-LAN (SoL) Crash Cart Console
ipmitool -I lanplus -H <BMC_IP> -U <USER> -P <PASS> sol activate
```

### 3.2 Modern Redfish API Triage
```bash
# Query system health status via cURL and jq
curl -k -u <USER>:<PASS> -X GET https://<BMC_IP>/redfish/v1/Systems/1 | jq .Status

# Inspect PSU redundancy and power draw
curl -k -u <USER>:<PASS> -X GET https://<BMC_IP>/redfish/v1/Chassis/1/Power | jq .PowerSupplies
```

---

## 4. AI Clusters & GPU Node Triage

### 4.1 `nvidia-smi` Field Diagnostics
```bash
# General overview of power, temp, and memory
nvidia-smi

# Query uncorrectable ECC memory errors
nvidia-smi --query-gpu=index,name,ecc.errors.uncorrected.volatile.device_memory,ecc.errors.uncorrected.aggregate.device_memory --format=csv

# Check NVLink connection status and error counters
nvidia-smi nvlink -s
nvidia-smi nvlink -e

# Query clock throttling reasons
nvidia-smi -q -d PERFORMANCE | grep -A 5 "Clocks Throttle Reasons"
```

### 4.2 Thermal Throttling & Air Containment
* **Blanking Panels:** In an 8x GPU rack (e.g., H100 SXM5 drawing up to 10.2 kW per chassis), a single missing 1U blanking panel allows 45°C exhaust air to short-circuit into the cold aisle intake, causing thermal throttling within 90 seconds.
* **Liquid Cooling Quick Disconnects:** Inspect for weeping or non-spill valve seal leaks. Never service active cold plates without invoking Stop-Work Authority.

---

## 5. Power Distribution & Safety Hierarchy

### 5.1 Strict Legal Order of Precedence
1. **Federal & State Safety Law:** OSHA 1910.333, OSHA 1910.147 (LOTO), NFPA 70E.
2. **Site EHS & Security SOPs:** Site Method of Procedure (MOP), two-person rule in medium-voltage spaces.
3. **OEM Hardware Specifications:** Manufacturer torque values, ESD protocols.
4. **DCO Role Boundary:** Technicians are strictly authorized for low-voltage IT systems (≤ 50V DC / chassis hot-swap). **Never touch 480V/415V switchgear or UPS battery strings.**

### 5.2 PDU Formulas & IEC Connectors
* Single-Phase AC: $P = V \times I \times \text{PF}$
* Three-Phase AC: $P = \sqrt{3} \times V_{\text{line-to-line}} \times I \times \text{PF}$
* **C13 / C14:** Up to 10A (EU) / 15A (US) at 250V (standard servers).
* **C19 / C20:** Up to 16A (EU) / 20A (US) at 250V (blade chassis, GPU servers).
* **C21 / C22:** High-temperature rated (up to 155°C pin temp) for dense compute.

---

## 6. Top 5 Hyperscale Interview Scenarios & Model Answers

### Scenario 1: Dead Management Port vs. ToR Switch Port Mispatch
* **Question:** *“You connect your crash cart to a newly installed server. The local console works, but the BMC management IP is unreachable from the network. How do you triage this systematically?”*
* **Model Answer:**
  1. **Physical Inspection:** Verify link/activity LEDs on the dedicated OOB management RJ45 port.
  2. **Cable Tracing:** Follow the patch cable from the server management port to the patch panel or Top-of-Rack (ToR) switch port against the cable schedule.
  3. **Local BMC Configuration:** Use local CLI (`ipmitool lan print 1`) via crash cart to verify if the BMC is configured for Static IP or DHCP, and verify VLAN tag assignment.
  4. **Port Status:** If LEDs are unlit, plug test cable into known-good port or laptop NIC. If link comes up, port was administratively disabled (`shutdown` on switch) or placed in the wrong management VLAN.

### Scenario 2: Uncorrectable Multi-Bit ECC Memory Error
* **Question:** *“A production server crashes and reboots. The BMC SEL shows an uncorrectable multi-bit ECC error on CPU 1, Channel B, Slot 1. What is your exact action plan?”*
* **Model Answer:**
  1. **Safety & Change Ticket:** Verify server is removed from production traffic and ticket is in *In Progress* state.
  2. **ESD Protocol:** Don an ESD wrist strap connected to unpainted chassis ground before opening top cover.
  3. **Identification:** Locate the exact physical slot using the motherboard silk-screen map (CPU 1, DIMM B1).
  4. **Component Replacement:** Inspect slot for dust or bent socket pins. Install an identical OEM-certified replacement DIMM (matching capacity, speed, DDR generation, and ECC rank). Reseat firmly until both retention latches snap shut.
  5. **Verification:** Boot server into diagnostic memory test utility for at least one full pass.
  6. **Telemetry & Closure:** Clear BMC SEL log, verify zero new ECC events, and update ticket with serial numbers.

### Scenario 3: DOM Rx Power Alert on 100G Uplink
* **Question:** *“Monitoring reports an optical degradation alert on a 100G LR4 transceiver. Rx power is reading -24.8 dBm (threshold is -12.0 dBm). Tx power on the remote side is normal (+0.2 dBm). What steps do you take?”*
* **Model Answer:**
  1. Recognize that **-24.8 dBm indicates near-total signal loss** (severe attenuation), while remote Tx is healthy. The issue lies in the optical path between endpoints.
  2. Inspect the fiber path for acute macrobends (< 30 mm bend radius) or physical pinching by zip ties.
  3. Clean and inspect both LC connectors on the patch cable using an optical scope and one-click cleaner.
  4. Swap the patch cord with a verified new singlemode cable.
  5. If attenuation persists, perform a loopback test on the transceiver. Replace module if loopback fails.

### Scenario 4: Stop-Work Authority & Unauthorized Command
* **Question:** *“A senior software engineer contacts you directly over Slack and asks you to immediately power-cycle an rPDU breaker for a stuck rack because a critical service is down. What do you do?”*
* **Model Answer:**
  1. **Exercise Stop-Work Authority.** Politely refuse immediate execution without an approved Emergency Change Management (ECM) ticket.
  2. Explain that cycling an entire rPDU breaker cuts power to up to 40 other unaffected servers sharing that PDU branch circuit, risking a catastrophic multi-tenant outage.
  3. Instruct the engineer to open an emergency ticket.
  4. In the meantime, use the BMC Out-of-Band interface or chassis power button to target the specific stuck server individually, leaving all other rack workloads completely isolated.

### Scenario 5: GPU Node Thermal Throttling
* **Question:** *“An 8-way GPU server in rack U18 starts thermal throttling, dropping clock speeds and causing job failures. Ambient data hall temperature is normal (22°C). How do you resolve this?”*
* **Model Answer:**
  1. Inspect physical rack front: check for missing 1U/2U blanking panels or unsealed cable pass-through brushes that cause hot-air recirculation.
  2. Check fan module status via `ipmitool sdr | grep -i fan` to ensure all counter-rotating hot-swap fans are running at full RPM.
  3. Check internal airflow baffles and shroud placement.
  4. Query `nvidia-smi -q -d TEMPERATURE` to verify whether overheating is uniform across all 8 GPUs (airflow restriction) or isolated to a single GPU (degraded thermal interface material / cold plate mounting failure).

---

## 7. The FACTS Incident Triage Framework

* **F — Fault Identification:** Exact symptom, error codes, and alerting sensor (e.g., `BMC SEL 0x12: Uncorrectable ECC DIMM_A1`).
* **A — Affected Configuration:** Hostname, Asset Tag, Rack Location (e.g., `RACK-04-B / U22`), and firmware version.
* **C — Containment & Isolation:** How the blast radius was limited (e.g., `Traffic drained via DNS weight 0, host cordoned`).
* **T — Technical Action Taken:** Step-by-step physical actions (e.g., `ESD strap donned, replaced DIMM SN#984123 with SN#452189, verified POST`).
* **S — Service Verification & Sign-off:** Proof of healthy return to service (e.g., `Diagnostic pass complete, SEL cleared, zero MCEs logged over 30 min`).

---

## 🛠 Complete Hands-On Practice Workbook

For physical paper drills to practice at your workbench with full simulation sheets:

[![Order on Amazon](https://img.shields.io/badge/Order-Amazon%20Paperback%20($19.99)-green?style=for-the-badge&logo=amazon)](https://www.amazon.com/dp/B0HP9JNGXL)

* **Title:** Data Center Operations Technician Field Workbook
* **Publisher:** Hyperscale Infrastructure Labs
* **ASIN:** `B0HP9JNGXL`
* **Direct Amazon Link:** https://www.amazon.com/dp/B0HP9JNGXL
