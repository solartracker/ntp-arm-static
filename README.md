# ntp-arm-static

This repository contains a build script for compiling **NTP** (Network Time Protocol daemon) as a **fully statically linked executable** for ARMv7 Linux devices.

The resulting binary runs on ARM Linux devices **without requiring any shared libraries**, making it ideal for minimal or embedded systems.

---

## What is NTP?

**NTP** (Network Time Protocol) is a protocol and software suite that synchronizes the clocks of computers over a network. Accurate system time is critical for logging, security, and time-sensitive applications, particularly in distributed or embedded environments.

Key features:

- Synchronizes system time with NTP servers on the Internet or local network  
- Can run as a daemon to maintain continuous time accuracy  
- Supports multiple time sources and hierarchical server configurations  
- Can be compiled statically to include all dependencies in a single binary  
- Lightweight and suitable for resource-constrained ARM devices  

Static compilation ensures that `ntpd` runs reliably on systems without the need for shared libraries.

---

## Build Method

### musl Static Build

- Build script: `ntp-arm-musl.sh`  
- Uses a standalone `arm-linux-musleabi` cross-compiler based on musl libc  
- Produces a fully statically linked `ntpd` executable  

Binaries built with musl are typically **smaller, more efficient, and portable**, particularly on **older or resource-constrained ARM hardware**, due to musl’s compact runtime and predictable static linking behavior.

---

## Build Instructions

1. **Clone this repository**

   ```bash
   git clone https://github.com/solartracker/ntp-arm-static
   cd ntp-arm-static

2. **Run the build script**

   ```bash
   ./ntp-arm-musl.sh
   ```

---


