# ntp-arm-static

This repository contains build scripts for compiling **NTP** (Network Time Protocol daemon) for **ARMv7 Linux** systems, with a focus on **static linking where technically feasible** and compatibility with legacy embedded firmware environments.

Two distinct build paths are provided:

* A **fully static musl-based build**, suitable for minimal or self-contained systems
* A **legacy Asuswrt/uClibc build**, compatible with Broadcom-based router firmware

Both build paths include **PPS (Pulse‑Per‑Second) support**, and the release artifacts also bundle **pps-tools** utilities as fully statically linked binaries.

---

## What is NTP?

**NTP** (Network Time Protocol) is a protocol and software suite that synchronizes the clocks of computers over a network. Accurate system time is critical for logging, security, cryptographic validation, and time-sensitive applications—especially in distributed or embedded systems.

Key features:

* Synchronizes system time with public or private NTP servers
* Runs as a daemon (`ntpd`) to continuously discipline the system clock
* Supports multiple time sources and hierarchical configurations
* Supports **hardware PPS time sources** for sub-microsecond accuracy
* Widely deployed on servers, routers, and embedded devices

---

## PPS Support

`ntpd` is compiled with **PPS support enabled** by including `sys/timepps.h` from **pps-tools**.

This allows `ntpd` to discipline the system clock using an external PPS signal (for example, from a GPS receiver), providing significantly improved time accuracy compared to network-only synchronization.

In addition to enabling PPS support in `ntpd`, **all pps-tools programs are included in the release artifacts** and are built as:

* **Fully statically linked binaries**
* No runtime library dependencies
* Suitable for minimal or firmware-based systems

This is true for both musl and Asuswrt/uClibc release artifacts.

---

## Build Methods

This repository currently supports **two independent build paths**, targeting different ARM libc and toolchain environments.

---

### musl Static Build (Fully Static)

* Build script: `ntp-arm-musl.sh`
* Toolchain: standalone `arm-linux-musleabi` cross-compiler
* libc: **musl**
* Output:
  * **Fully statically linked `ntpd`**
  * Fully statically linked **pps-tools** utilities

Binaries built with musl are **completely self-contained**, requiring no shared libraries on the target system. musl’s design explicitly supports static networking, name resolution, threading, and PPS operation, making it well-suited for fully static system daemons such as `ntpd`.

This build is ideal when:

* The target system lacks shared libraries
* You want maximum portability
* You control the entire runtime environment

---

### Asuswrt / uClibc Build (Dynamically Linked `ntpd`)

* Build script: `ntp-arm-asuswrt.sh`
* Toolchain: **arm-brcm-linux-uclibcgnueabi** (Asuswrt)
* Target architecture: **ARMv7 soft-float**
* libc: **uClibc 0.9.32.1**
* Output:
  * `ntpd` dynamically linked against system uClibc
  * Fully statically linked **pps-tools** utilities

This build path exists to support **Asuswrt-based firmware and similar Broadcom ARM environments**, where the vendor toolchain, kernel, and C library are effectively fixed.

#### Static Linking Limitations under uClibc

While **pps-tools** can be fully statically linked under the Asuswrt toolchain, **`ntpd` cannot**.

This is a known limitation of **uClibc 0.9.32.x**, not a build error:

* uClibc’s resolver, NSS, and threading components are not fully static-linkable
* Certain libc objects are only provided as shared libraries
* Static networking support is incomplete by design

As a result, `ntpd` must remain **dynamically linked to uClibc** on Asuswrt systems, even though PPS functionality is fully supported.

---

#### Toolchain Constraints

The Asuswrt toolchain was built with **GCC 4.5.3**, which predates the introduction of **libatomic**. Modern versions of OpenSSL require libatomic for certain atomic operations on ARM.

Limitations:

* GCC 4.5.3 **does not ship libatomic**
* Stock Asuswrt environments are therefore limited to very old OpenSSL releases

---

#### libatomic Backport

To overcome this limitation, **libatomic was built separately from GCC 4.8.1** and integrated into the build environment.

With libatomic available, it was possible to successfully build:

* **OpenSSL 3.6.0**
* **NTP** linked against that OpenSSL version

This enables modern cryptography and NTP functionality on legacy Asuswrt/uClibc systems that would otherwise be locked to obsolete libraries.

---

## Build Instructions

### Clone the repository

```bash
git clone https://github.com/solartracker/ntp-arm-static
cd ntp-arm-static
```

### Build using musl (fully static)

```bash
./ntp-arm-musl.sh
```

### Build using the Asuswrt toolchain

```bash
./ntp-arm-asuswrt.sh
```

---

## Notes

* All builds are intended for **cross-compilation** on a Linux host.
* PPS support is enabled in all builds via **pps-tools**.
* The musl build produces a **fully static** `ntpd` and PPS utilities.
* The Asuswrt build produces a **dynamically linked `ntpd`** with **fully static PPS utilities**.
* Static linking behavior is constrained by libc design; uClibc does not support fully static `ntpd`.

---

## License

See the individual upstream project licenses (NTP, OpenSSL, GCC, pps-tools) for details. This repository contains build scripts only.



