# ntp-arm-static

This repository contains build scripts for compiling **NTP** (Network Time Protocol daemon) as a **fully statically linked executable** for ARMv7 Linux devices.

The resulting binaries run on ARM Linux systems **without requiring any shared libraries**, making them suitable for minimal, embedded, or firmware-based environments.

---

## What is NTP?

**NTP** (Network Time Protocol) is a protocol and software suite that synchronizes the clocks of computers over a network. Accurate system time is critical for logging, security, cryptographic validation, and time-sensitive applications—especially in distributed or embedded systems.

Key features:

* Synchronizes system time with public or private NTP servers
* Runs as a daemon (`ntpd`) to continuously discipline the system clock
* Supports multiple time sources and hierarchical configurations
* Can be compiled statically to bundle all dependencies into a single binary
* Well-suited for resource-constrained ARM platforms

Static compilation ensures `ntpd` runs reliably on systems that lack package managers or shared libraries.

---

## Build Methods

This repository currently supports **two independent build paths**, targeting different ARM libc and toolchain environments.

---

### musl Static Build

* Build script: `ntp-arm-musl.sh`
* Toolchain: standalone `arm-linux-musleabi` cross-compiler
* libc: **musl**
* Output: fully statically linked `ntpd`

Binaries built with musl are typically **smaller, more portable, and easier to statically link**, particularly on older or minimal ARM systems. musl’s predictable static linking behavior makes it ideal when you control the entire runtime environment.

---

### Asuswrt / uClibc Static Build

* Build script: `ntp-arm-asuswrt.sh`
* Toolchain: **arm-brcm-linux-uclibcgnueabi** (Asuswrt)
* Target architecture: **ARMv7**
* libc: **uClibc 0.9.32.1**

This build path exists to support **Asuswrt-based firmware and similar Broadcom ARM environments**, where the vendor toolchain and uClibc version are effectively fixed.

#### Toolchain Constraints

The Asuswrt toolchain was built with **GCC 4.5.3**, which predates the introduction of **libatomic**. Modern versions of OpenSSL require libatomic for certain atomic operations on ARM, which creates a hard limitation:

* GCC 4.5.3 **does not provide libatomic**
* Without libatomic, OpenSSL is normally limited to very old releases

#### libatomic Backport

To work around this limitation, **libatomic was built separately from GCC 4.8.1** and integrated into the build environment. This makes it possible to satisfy OpenSSL’s atomic requirements while continuing to use the legacy Asuswrt toolchain.

With libatomic available, it was possible to successfully build:

* **OpenSSL 3.6.0**
* **NTP** linked against that OpenSSL version

This allows modern NTP to run on legacy Asuswrt/uClibc systems that would otherwise be locked to obsolete cryptographic libraries.

---

## Build Instructions

### Clone the repository

```bash
git clone https://github.com/solartracker/ntp-arm-static
cd ntp-arm-static
```

### Build using musl

```bash
./ntp-arm-musl.sh
```

### Build using the Asuswrt toolchain

```bash
./ntp-arm-asuswrt.sh
```

---

## Notes

* These builds are intended for **cross-compilation** on a Linux host.
* The Asuswrt build intentionally targets a **legacy userspace** while still supporting **modern OpenSSL and NTP**.
* Static linking increases binary size but eliminates runtime dependency issues common on embedded firmware platforms.

---

## License

See the individual upstream project licenses (NTP, OpenSSL, GCC) for details. This repository contains build scripts only.


