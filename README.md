# Linux 7.0.9 Unleashed — Custom Kernel Builder

A fully automated repository containing patches, configurations, and a one-click build utility to recreate our custom-tuned, high-performance Debian/Ubuntu Linux kernel.

---

## 🧸 Layman Explanation (For Non-Technical Users)

### What this project does:
This project unlocks the absolute limits of your computer's internet speed, Wi-Fi range, and audio efficiency. Standard operating systems are set up like rental cars—they have built-in speed limiters (governors) to make sure you don't use too much power, get too close to regulatory limits, or heat up your system. We have completely removed these limits.

### The Key Upgrades:
1. **Removing Wi-Fi Speed Limits**: By law, Wi-Fi cards must limit how loud they broadcast and step aside (throttle down) if they detect neighbors using the same channel. We bypassed these limits. Your Wi-Fi will now broadcast at its absolute maximum physical capability, ignoring nearby signals and refusing to throttle down.
2. **Gigabit Internet Supercharger**: Normally, your network card uses a small "waiting lounge" (buffer) for incoming data. If you have extremely fast fiber internet, this lounge fills up instantly, causing bottleneck lag. We expanded this lounge to 1 Gigabyte, allowing massive bursts of data to fly through without backing up.
3. **No More Audio Lag or Crackle**: Some audio systems put themselves to sleep to save power, waking up only when sound plays. This starting and stopping causes tiny stuttering delays (latency spikes) that slow down the whole computer. We changed the sound card's internal instructions so it stays active, preventing these interruptions.

---

## 💻 Developer & GitHub Explanation (For Technical Users)

### Technical Specifications:
This repository contains 8 patched source files and a compiler-optimized kernel configuration (`kernel.config`) derived from the stable **Linux 7.0.9** tree. 

### Patched Code & Rationale:

#### 1. Wi-Fi Regulatory Bypass — [`net/wireless/reg.c`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/reg.c)
* **What we did:** Bypassed the kernel's Central Regulatory Domain Agent (CRDA) and regional constraints.
* **Rationale:** Disables all localized Wi-Fi transmit power (TX power) restrictions and channel bans, allowing the card to run at full hardware limits.

#### 2. Interference Shielding — [`drivers/net/wireless/ath/ath9k/hw.c`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/hw.c)
* **What we did:** Modified Clear Channel Assessment (CCA) logic to ignore neighboring AP interference thresholds.
* **Rationale:** Prevents the Atheros wireless driver from dropping HT40 (40MHz channel width) links back to HT20 in crowded, noisy environments, maintaining maximum bandwidth.

#### 3. TCP Initial Congestion Window Scaling — [`include/net/tcp.h`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/tcp.h)
* **What we did:** Scaled `TCP_INIT_CWND` (Initial Congestion Window) from the standard 10 packets to **128 packets**.
* **Rationale:** Enables your computer to send/receive larger chunks of data immediately during the initial TCP handshake without waiting for slow-start ramp-ups.

#### 4. High-Throughput TCP Memory Buffers — [`net/ipv4/tcp.c`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/tcp.c)
* **What we did:** Raised maximum TCP read/write memory thresholds (`sysctl_tcp_rmem` and `sysctl_tcp_wmem`) to 1GB.
* **Rationale:** Prevents memory pool allocation bottlenecks during multi-gigabit/40G data transfers.

#### 5. Wireless Link Performance Tuning — [`drivers/net/wireless/ath/ath9k/init.c`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/init.c)
* **What we did:** Fine-tuned initialization and link-negotiation behavior.
* **Rationale:** Prioritizes link stability and high-gain states over default power-saving sleep cycles.

#### 6. Transceiver Power Override — [`drivers/net/wireless/ath/ath9k/gpio.c`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/gpio.c)
* **What we did:** Overrode GPIO pins controlling the RF transceiver's power amplifier.
* **Rationale:** Drives transceiver hardware at maximum power levels.

#### 7. Bluetooth Coexistence Prioritization — [`drivers/net/wireless/ath/ath9k/btcoex.c`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/btcoex.c)
* **What we did:** Adjusted coexistence scheduling profiles.
* **Rationale:** Stops Bluetooth peripherals (like mice or keyboards) from forcing the Wi-Fi card to throttle its throughput during concurrent transfers.

#### 8. Realtek Audio Interrupt Minimization — [`sound/hda/codecs/realtek/alc269.c`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/alc269.c)
* **What we did:** Rerouted codec power states and pin-switching limits.
* **Rationale:** Stops the Realtek ALC269 codec from generating high-priority CPU hardware interrupts (which cause system latency spikes) during audio stream state changes.

---

## 🚀 One-Click Rebuilder Pipeline

We have automated the complete process of downloading, patching, and building the kernel into a single script.

### File Manifest:
* **[`recreate_success.sh`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/recreate_success.sh)**: The master orchestrator script. It downloads upstream sources, runs the patch installer, sets compiler optimization flags (`-O3 -march=native`), and starts multithreaded compilation.
* **[`universal_freedom_installer.py`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/universal_freedom_installer.py)**: Maps the patched files in this directory to their respective paths inside the kernel tree.
* **[`PATCHED_FILES_PATH_REGISTRY.txt`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/PATCHED_FILES_PATH_REGISTRY.txt)**: Relational database file showing target directories in the source tree.
* **[`kernel.config`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/kernel.config)**: Custom `.config` with enabled BBR congestion control, FQ-Codel queuing, and optimized drivers.

## 🐧 Supported Operating Systems
This kernel has been rigorously tested and validated **only on Debian 13**. While it may function on other Debian-based distributions (such as Ubuntu), such usage is not officially supported and remains untested. Use on any distribution other than Debian 13 is at your own risk.

