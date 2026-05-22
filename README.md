# Linux 7.0.9 Unleashed — Custom Kernel Builder

A fully automated repository containing patches, configurations, and a one-click build utility to recreate our custom-tuned, high-performance Debian/Ubuntu Linux kernel.

---

## 🧸 Layman Explanation (For Non-Technical Users)

### 🌟 Sharing is Caring: Why We Do This (Open Source Ethos)
This project is built on the core philosophy of open source: **sharing is caring**. Technology shouldn't be locked behind arbitrary barriers, artificial limiters, or complicated instructions. Even though I am still a student and learning every day, I believe in giving back to the community and society that built the very tools we use. By publishing this work and providing pre-compiled packages, we make high-performance systems accessible to everyone—from hardcore developers to regular users who just want their computers to run faster. Layman users and testers are the most consistent and valuable feedback providers. Your real-world testing helps us understand what works and what doesn't. Sharing our tweaks is our way of contributing to a cooperative world.

### 🏛️ The Rationale: Bypassing the "Rental Car" Governors
Standard operating systems are configured like rental cars. They have strict speed limiters (governors) built into their systems to make sure the hardware never runs too hot, never violates old regional regulations, and never draws too much power. This makes them safe for everyone, but it means you are only getting 50% of the performance you actually paid for. 

In this kernel, we have completely cut those handcuffs. Your computer will now run at its absolute maximum hardware limits, treating your networks and processors like a race car on an open track.

---

### 🛠️ The 25 Crucial Upgrades (Explained Simply)

#### 📡 Part 1: Atheros Wi-Fi & Regulatory Overrides (12 Tweaks)
Your Wi-Fi card is physically capable of broadcasting much louder and using wider channels, but it is forced to be "polite." Here is how we unleashed it:
1. **Unlocking Regional Bans**: We stripped the regional code locks. Your card can now scan and broadcast on channels that are normally banned in your country.
2. **Channel 14 Access**: We unlocked Channel 14, the rarest and cleanest channel in the 2.4GHz spectrum, which is normally locked down globally.
3. **Bypassing Radar Locks (DFS)**: Normally, if your card hears a whisper of radar, it shuts down or throttles. We bypassed this, allowing you to use high-speed DFS channels without interruption.
4. **Active Scanning (No-IR Bypass)**: We unlocked active scanning on all channels, allowing your device to actively search for access points instead of waiting passively.
5. **Enabling Wide Lanes (HT40/80/160/320)**: Regional constraints that disable wide frequency lanes (like 40MHz on 2.4GHz or 160/320MHz on 5/6GHz) are stripped, allowing maximum channel widths.
6. **Ignoring Nearby Neighbors**: Normally, if your card hears a neighbor’s Wi-Fi router, it drops from the fast lane (40MHz) to the slow lane (20MHz) to prevent interference. We told the card to ignore neighbors and stay in the fast lane.
7. **Braveheart Power Amplifier Override**: We bypassed power limits, telling the wireless transmitter to output at the maximum possible electrical power the silicon can handle.
8. **Permanent Coexistence Activation**: We forced Bluetooth and Wi-Fi coexistence features to remain permanently active so they coordinate efficiently without crashing.
9. **Boot-Time Power Boost**: We set the driver to initialize the card at maximum broadcast power immediately upon system boot.
10. **Bypassing Decryption Timeout Lag**: We moved Wi-Fi decryption from the card's slow hardware chip to your fast computer CPU. This prevents connection timeouts and delays when waking the laptop from sleep.
11. **WLAN Airtime Dominance**: Wi-Fi and Bluetooth share the same antenna. We configured the controller to give Wi-Fi 100% airtime priority over Bluetooth.
12. **Bluetooth Coexistence Stomping**: If Wi-Fi and Bluetooth try to talk at the exact same millisecond, Wi-Fi "stomps" over the Bluetooth signal, ensuring your internet throughput never drops for accessories.

#### 🔊 Part 2: Realtek Audio Lag Annihilation (3 Tweaks)
Sound cards try to save power by going to sleep, which causes popping, crackling, and latency:
13. **King Kong Audio Roar**: We bypassed the default software gain limits on the Sony VAIO headphone and speaker jacks, unlocking a massive audio boost up to +60dB.
14. **System Interrupt Suppression**: Every time the audio card changes power states, it interrupts the CPU. We stabilized the audio routing to stop these interrupt spikes, eliminating micro-stutters.
15. **VAIO Pin Grounding**: We grounded unused microphone line pins to stop electrical background hiss and static noise from leaking into your headphones.

#### 🚀 Part 3: High-Speed TCP Networking (10 Tweaks)
Linux network settings were designed in the era of dial-up internet. We scaled the buffer pipes to match modern multi-gigabit fiber connections:
16. **Firehose Startup (128 CWND)**: Instead of testing the connection slowly (sending 10 packets), we blast 128 packets of data instantly during the initial connection.
17. **Hyper-Fast Recovery (10ms RTO)**: If a packet is lost, standard systems wait 200 milliseconds to retry. We slashed this to 10 milliseconds.
18. **Unresponsive Host Fail-Fast**: We slashed the time limit for retrying dead connections from 2 minutes to 3 seconds.
19. **Instant Handshake**: Slash initial connection timeout from 1 second to 100 milliseconds to connect to web servers instantly.
20. **Rapid Delayed ACKs**: Slash acknowledgment delays to 20ms maximum to keep data flowing without pauses.
21. **Instant Socket Recycling**: Slash the wait time to destroy dead sockets from 60 seconds to 1 second, keeping memory clean.
22. **Fail-Fast Handshakes**: Limit handshake retries to 3 attempts, instantly freeing resources from dead-weight connections.
23. **Fast Keepalive Probes**: Slashed idle keepalive verification from 2 hours to 15 minutes to clear out dead routes.
24. **Industrial-Sized Data Pipes**: We expanded the memory buffers for reading and writing data from a few kilobytes to 16 Megabytes (default) and 64 Megabytes (maximum).
25. **Preventing Window Collapse**: We locked the initial transmission window clamp to 65,535 packets to prevent the system from shrinking your internet pipes during minor network jitters.

---

## 💻 Developer & GitHub Explanation (For Technical Users)

### Technical Specifications:
This repository contains 8 patched source files and a compiler-optimized kernel configuration (`kernel.config`) derived from the stable **Linux 7.0.9** tree.

### Detailed Patched Code & Rationale:

#### 1. Global Regulatory Annihilation — [`net/wireless/reg.c`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/reg.c)
* **What we did:** Stripped all channel flag restrictions in `map_regdom_flags()`.
* **Details:**
  - Clears `IEEE80211_CHAN_NO_IR` (No Initiate Radiation/Active Scan) globally.
  - Clears `IEEE80211_CHAN_RADAR` (disabling DFS/Radar lockout loops).
  - Clears `IEEE80211_CHAN_DISABLED` (enabling regionally locked frequencies like 2.4GHz Channel 14).
  - Clears all wide-band channel restriction flags (`NO_HT40`, `NO_80MHZ`, `NO_160MHZ`, `NO_HE`, `NO_EHT`, `NO_UHR`, `NO_320MHZ`).

#### 2. Interference Shielding & Max Power — [`drivers/net/wireless/ath/ath9k/hw.c`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/hw.c)
* **What we did:** Bypassed energy detection threshold checks and power limits.
* **Details:**
  - Forces `ah->config.cwm_ignore_extcca = true;` in `ath9k_hw_init_config()` to maintain 40MHz HT channel width even in highly congested 2.4GHz spectrum.
  - Overrides `ath9k_hw_apply_txpower()` variables `chan_pwr` and `new_pwr` to `MAX_COMBINED_POWER`, forcing maximum radio broadcast levels.

#### 3. Bluetooth Coexistence priority — [`drivers/net/wireless/ath/ath9k/btcoex.c`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/btcoex.c)
* **What we did:** Re-scheduled WLAN/Bluetooth priority weights.
* **Details:**
  - Overrode `.wl_active_time = 0xFF` and `.wl_qc_time = 0xFF` in `btcoex.c` to reserve maximum airtime slot ratios for WLAN traffic over Bluetooth.

#### 4. Bluetooth Stomping & GPIO Power — [`drivers/net/wireless/ath/ath9k/gpio.c`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/gpio.c)
* **What we did:** Forced the coexistence stomping policy.
* **Details:**
  - Overrode `stomp_type = ATH_BTCOEX_STOMP_NONE;` inside the priority sequencer to ensure Wi-Fi traffic never yields or throttles to Bluetooth signals during active transfers or background scanning.

#### 5. Driver Crypto Offloading & Initial Power — [`drivers/net/wireless/ath/ath9k/init.c`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/init.c)
* **What we did:** Changed module defaults and forced boot-time power states.
* **Details:**
  - Set `ath9k_modparam_nohwcrypt = 1;` as default to offload WEP/TKIP/AES-CCMP cryptography to host CPU threads, eliminating post-suspend reconnection failures and hardware decryption queue stalls.
  - Set `ath9k_btcoex_enable = 1;` to force-enable coexistence routines.
  - Sets `sc->cur_chan->txpower = MAX_COMBINED_POWER;` at initialization.

#### 6. Realtek Audio Quirk: "King Kong Roar" — [`sound/hda/codecs/realtek/alc269.c`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/alc269.c)
* **What we did:** Added custom PCI vendor hardware quirk `ALC269_FIXUP_KING_KONG`.
* **Details:**
  - Registers `ALC269_FIXUP_KING_KONG` quirk for Sony VAIO sub-vendor ID (`0x104d`).
  - Implements `alc269_fixup_king_kong()` to override output amplifier caps (`query_amp_caps`, clears `AC_AMPCAP_OFFSET`, sets `0x04 << AC_AMPCAP_OFFSET_SHIFT`, and applies `snd_hda_override_amp_caps`) on node pins `0x14` and `0x21` to boost headroom by +60dB.
  - Chains to `ALC269_FIXUP_SONY_VAIO` (which sets Pin 0x19 to `PIN_VREFGRD`), which stabilizes codec power states, grounds micro-lines, and prevents high-priority hardware CPU interrupt spikes.

#### 7. TCP Timing Limits & Handshake Optimizations — [`include/net/tcp.h`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/tcp.h)
* **What we did:** Optimized TCP macros for high-speed, low-latency performance.
* **Details:**
  - `#define TCP_INIT_CWND 128` (Instant multi-gigabit handshake burst).
  - `#define TCP_RTO_MIN ((unsigned)(HZ / 100))` (Reduces retransmission delay to 10ms).
  - `#define TCP_RTO_MAX ((unsigned)(3 * HZ))` (Sets maximum backoff limit to 3 seconds).
  - `#define TCP_TIMEOUT_INIT ((unsigned)(HZ / 10))` (Initial connection timeout of 100ms).
  - `#define TCP_DELACK_MAX ((unsigned)(HZ / 50))` and `TCP_DELACK_MIN` / `TCP_ATO_MIN` to `2U` (Reduces ACK delay to 20ms max, with a 2-jiffy minimum).
  - `#define TCP_TIMEWAIT_LEN (1 * HZ)` (Sockets recycled in 1 second instead of 60).
  - `#define TCP_SYN_RETRIES 3` and `#define TCP_SYNACK_RETRIES 3` (Fail dead routes fast).
  - `#define TCP_RETR2 8` (Linger timeout dropped from 15 retries to 8).
  - `#define TCP_KEEPALIVE_TIME (15 * 60 * HZ)` (Idle socket verification down to 15 minutes).

#### 8. TCP Socket Buffers & Clamping — [`net/ipv4/tcp.c`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/tcp.c)
* **What we did:** Raised read/write socket memory buffers and forced window clamping.
* **Details:**
  - Hardcoded `sysctl_tcp_wmem` and `sysctl_tcp_rmem` arrays to: Min `4096`, Default `16777216` (16MB), Max `67108864` (64MB).
  - Overrode `tp->snd_cwnd_clamp = 65535;` inside `tcp_init_sock()` to prevent dynamic MTU-based reduction of the window clamp boundary.

---

## 📥 Pre-compiled Binaries (Direct Install)

For non-technical users, developers, or testers who want to test the custom-tuned kernel immediately without spending hours compiling, we provide pre-compiled Debian packages.

### How to Download & Install:

1. Go to the [Releases](https://github.com/gorillanobakaa-dot/kernel-7.0.9-unleashed/releases) section of this repository.
2. Download both the image and headers packages:
   - `linux-image-7.0.9-unleashed-*.deb`
   - `linux-headers-7.0.9-unleashed-*.deb`
3. Open a terminal in the folder where you downloaded the files and run:
   ```bash
   sudo dpkg -i linux-image-7.0.9-unleashed-*.deb linux-headers-7.0.9-unleashed-*.deb
   ```
4. Update your bootloader:
   ```bash
   sudo update-grub
   ```
5. Reboot your system:
   ```bash
   sudo reboot
   ```

---

## 🛠️ One-Click Rebuilder Pipeline

We have automated the complete process of downloading, patching, and building the kernel into a single script.

### File Manifest:
* **[`recreate_success.sh`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/recreate_success.sh)**: The master orchestrator script. It downloads upstream sources, runs the patch installer, sets compiler optimization flags (`-O3 -march=native`), and starts multithreaded compilation.
* **[`universal_freedom_installer.py`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/universal_freedom_installer.py)**: Maps the patched files in this directory to their respective paths inside the kernel tree.
* **[`PATCHED_FILES_PATH_REGISTRY.txt`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/PATCHED_FILES_PATH_REGISTRY.txt)**: Relational database file showing target directories in the source tree.
* **[`kernel.config`](file:///home/gorilla/Documents/Debian.Kernel.Work/Kernel.work.21.May/last%20kernel%20built/kernel.config)**: Custom `.config` with enabled BBR congestion control, FQ-Codel queuing, and optimized drivers.

## 🐧 Supported Operating Systems
This kernel has been rigorously tested and validated **only on Debian 13**. While it may function on other Debian-based distributions (such as Ubuntu), such usage is not officially supported and remains untested. Use on any distribution other than Debian 13 is at your own risk.
