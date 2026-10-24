# CODEX HANDOFF — Raspberry Pi 5 / TAC5212_HAT

**Prepared:** 2026-10-24  
**Purpose:** Build a reproducible, exceptionally stable, low-latency guitar/bass audio software environment on a Raspberry Pi 5, starting before the newly manufactured custom HAT arrives.  
**Project repository:** https://github.com/arjepsen/TAC5212_HAT  
**Authority:** This conversation’s explicit project goals and observed Pi 5 terminal output. The actual, most recent HAT design files take precedence over any historical discussions when accessible. Where current board files are not available, details are marked **unverified**.  
**Status:** Working NVMe-based development machine; no audio integration, real-time performance, or end-to-end latency measurements have yet been validated on it.

> **PROJECT-IDENTITY RULE:** This is **only** the `arjepsen/TAC5212_HAT` Raspberry Pi 5 project. The older `Pedalboard-integrated-audio` project is separate. **Do not import that project's hardware, sample-rate settings, UI design, software assumptions or files.** This handoff supersedes the previously incorrect versions.

## 1. The actual project and desired result

Develop the **software platform for a NEW, compact, purpose-built Raspberry Pi 5 GPIO audio HAT**, currently being manufactured, optimized for guitar/bass effects. **The development and validation host is the Raspberry Pi 5.** Do not introduce unrelated carrier-board assumptions.

**Non-negotiable objectives:**

- **96,000 samples/s (96 kHz)** is the **target operating rate**. Never silently substitute another sample rate because of defaults, existing projects or convenience. Record incompatibilities; investigate properly. Tests at other rates can be explicitly labeled comparisons, **not** a decision to change our target.
- Lowest **reliable** measured guitar-input-to-headphone/line-output latency; prioritize dropout-free live playing over impressive but unstable buffer numbers.
- Great sound quality, appropriate gain structure and dynamics, reliable operation when processing demanding LV2 effects and Neural Amp Modeler (NAM)-type plugins.
- Study **pi-Stomp, PiPedal and MOD** sources and practices rather than applying generic, unverified Linux tuning recipes.
- **Two mutually exclusive software modes:** (a) MOD-UI + mod-host + JACK, drawing from pi-Stomp; (b) PiPedal using direct ALSA. Only one engine owns the sound card at a time. Implementation of mode selection comes later and must be reversible.
- Run **headless** on Pi; web UI used remotely. The system must survive reboot and eventually power-loss scenarios without manual login.
- Build the software against temporary **HiFiBerry DAC+ ADC Pro** until the custom HAT arrives. The temporary card and production card have different capabilities and likely different ALSA/Device Tree configuration. Tests of one do **not** prove the other.
- Use **the existing `arjepsen/TAC5212_HAT` GitHub repository** for this project. Work within its actual structure; do not create a separate repository, initialize/overwrite files, commit, or push without the owner’s explicit instruction.
- Implement changes in small, verifiable stages with short user-facing results. Use written configuration manifests, clear rollback procedures, and commits.

## 2. Actual new HAT: known design information vs what remains unverified

The new HAT, from **October 2026 design/manufacturing discussions**, is a Raspberry-Pi-header-mounted audio card. The following is **design-history context**, *not* an independently verified final-manufacturing BOM or netlist:

| Item | Current project information | Verification required |
|---|---|---|
| SBC | Raspberry Pi 5; direct 40-pin GPIO header connection; official Pi Active Cooler; NVMe Base physically below Pi | Check final board geometry, cooler clearance, stacking and electrical loading on arrival |
| Codec | **TI TAC5212** (exact ordered part/package must be verified) | Inspect **exact manufactured** schematic/BOM, package, I²C address and driver variant before codec bring-up |
| Instrument front end | **OPA2365**-based high-impedance guitar front end (previous design discussions) | Confirm final populated stages and gain using actual schematics, not old draft |
| Line input | In more recent layout, **passive line-input path**; an earlier second-op-amp stage (U3) was removed | Check final schematic and channel mapping |
| Audio outputs | Headphone and stereo line-output provisions; recent design discussions mention **680 µF headphone** and **100 µF line** coupling capacitors | Verify exact capacitor positions, polarization, common grounds and codec output configuration |
| Analog power | **LT3042** analog supply proposed from Pi 5 header power (target voltage to verify) | Verify rail startup, regulator current, power sequencing, voltage and resistor values on actual manufactured board |
| Digital codec I/O | Codec IOVDD from host 3.3 V (design intent) | Verify final netlist, voltage at pins and compatibility with driver startup |
| Control | Pi I²C pins 3 (GPIO2/SDA) and 5 (GPIO3/SCL), design intent | Check wiring, pull-ups and final I²C address |
| Audio bus | I²S over GPIO header, design intent: pin 12 BCLK; pin 35 LRCLK/FSYNC; pin 38 codec→Pi; pin 40 Pi→codec | Verify actual HAT netlist and BCM2712/RP1 pinmux, overlays and audio driver binding |
| Audio clocking | Planned stereo 32-bit I²S slots, Pi-generated BCLK/FSYNC, **no separate MCLK**, target **96 kHz** | 2 channels × 32 bits × 96,000 = **6.144 MHz BCLK**. Check codec modes, actual clock accuracy and ALSA capabilities |
| Connectors | Audio connectors are **chassis wired**, not soldered audio jacks on HAT | Final harness pinouts and ESD/mechanical protections require verified board files |
| Manufacture | Order preparation called for **5 boards, 2 assembled**, JLCPCB Economic PCBA top-side, genuine TI TAC5212 required. User subsequently reported the HAT is being produced. | Obtain latest actual order files, BOM and assembly revision before claiming production configuration |

**Historical design iterations are not final facts.** Earlier proposals included two OPA2365 devices and other HAT layouts; these must not be silently reintroduced. An earlier discussion referenced **U1 LT3042**, **U2 OPA2365**, **IC1 TAC5212** and DNP items, but the actual submitted files have **not been directly inspected for this handoff**. None of these reference designators or DNP facts should be treated as final. The latest board files/manufacturing release are mandatory source-of-truth for physical interfaces. **Software development can proceed without them; HAT-specific hardware assumptions cannot.**

**Special constraint:** The Pi 5 + NVMe Base + Active Cooler mechanical/electrical stack is distinct from the HAT's DSP software work. Do not remove the NVMe, disable PCIe, change clocks on audio pins or power external HAT hardware from unverified nets merely for convenience.

## 3. Current Raspberry Pi 5: evidence from real terminal output

**This is the observed starting point, not hypothetical installation advice.**

| Property | Observed result |
|---|---|
| OS | Raspberry Pi OS Lite, 64-bit, Debian 13 (Trixie); previous OS check showed Debian 13.7 |
| Kernel | `6.18.50+rpt-rpi-2712` on Pi 5 ARM64 |
| Root | `/dev/nvme0n1p2`, ext4, **238 GiB** |
| Boot | `/dev/nvme0n1p1`, vfat, **512 MiB**, mounted `/boot/firmware` |
| Storage | Samsung **256 GB** NVMe SSD on Pimoroni NVMe Base; boots with microSD removed |
| Access | SSH from another computer to `anders@pi5`; Ethernet functional |
| Preemption | `/boot/config-$(uname -r)` reports `CONFIG_PREEMPT=y`; `/sys/kernel/realtime` does **not** exist |
| Power condition | `vcgencmd get_throttled` = `throttled=0x0` at test time |
| Temperature | `vcgencmd measure_temp` = **41.7 °C** at test time |
| CPU governor | `ondemand` |
| Available governors | `conservative ondemand userspace powersave performance schedutil` |
| CPU limits | `scaling_min_freq` **1500000 kHz**; `scaling_max_freq` **2400000 kHz** |
| Cooling | Official Raspberry Pi 5 **Active Cooler**; fan **2835 RPM**, `pwm1=75` of 255, `pwm1_enable=1` when inspected |
| Fan customization | Under `[all]` in `/boot/firmware/config.txt`: `dtparam=fan_temp0=30000` |
| Locale | User chose `en_DK.UTF-8`; intended timezone Europe/Copenhagen; confirm live setting |
| Audio | **Not yet installed/configured/tested** with HiFiBerry, PiPedal or MOD |
| Software updates | Recent update was discussed; package/firmware state should be inspected rather than assumed |

**Interpretation:** NVMe boot, SSH, cooler and baseline temperature/power look functional. That does *not* prove a PREEMPT_RT kernel, audio performance, worst-case thermals or sustained absence of underruns.

**Crucial PREEMPT_RT caveat:** `CONFIG_PREEMPT=y` means a preemptible kernel but is **not by itself** proof of `CONFIG_PREEMPT_RT=y`. PiPedal's current documentation makes statements about RT kernel availability on Raspberry Pi OS that **must be checked against this actually running 6.18.50 image**; in our initial observations `/sys/kernel/realtime` was absent. Don't overwrite reality to match documentation.

**Cooling:** Fan is already automatically regulated through the Pi 5 thermal system, with the first level starting at 30 °C. At measured 41.7 °C the fan ran around 29% PWM. **No fan polling service is needed.** Keep thermal safety fallback. Don't replace PWM firmware/thermal management with background scripts.

## 4. Authoritative reference sources — read the right parts, not everything

The following are references **for software methods**, not authority to alter our hardware or 96 kHz target.

1. **pi-Stomp OS build:** https://github.com/TreeFallSound/pi-gen-pistomp — study current OS scripts and generated configuration. Their official image is Raspberry Pi OS Lite with RT kernel, JACK2, mod-host, MOD-UI and pi-Stomp. **Do not flash their complete image onto our working NVMe installation.** Inspect implementations, compatibility with Trixie/our kernel and exact files.
2. **pi-Stomp main software:** https://github.com/TreeFallSound/pi-stomp — study audio-startup interactions, services, controller boundaries, MOD integration and runtime monitoring. Separate hardware-dependent code from reusable architecture. Do not assume pi-Stomp-specific GPIO layout or peripherals.
3. **pi-Stomp manual:** https://treefallsound.github.io/pistomp-manual/ — performance tuning, developer/image-build documentation and audio management. Some statements describe their own card and hardware only.
4. **PiPedal source:** https://github.com/rerdavies/pipedal — especially installation/configuration, real-time audio thread setup, ALSA and latency tester. PiPedal 2.x is currently documented for Raspberry Pi 5 and Trixie.
5. **PiPedal latency discussion:** https://github.com/rerdavies/pipedal/blob/main/docs/AudioLatency.md — explains ALSA path, scheduling vs buffering, xruns and loopback round-trip measurement. Some historic USB and PREEMPT commentary predates the present Pi 5; verify version relevance.
6. **PiPedal requirements:** https://rerdavies.github.io/pipedal/SystemRequirements.html — check published requirements against *our running kernel*; do not infer its RT state from wording alone.
7. **Official Raspberry Pi documentation:** https://www.raspberrypi.com/documentation/computers/raspberry-pi.html — Pi 5 cooler, boot firmware, GPIO, OS features and current hardware guidance.
8. **Optional source snapshot archive:** `software-references.zip` (if it is separately available to Codex; it is not automatically part of the GitHub repository). It contains source snapshots of `pipedal-main`, `pedalboard-os-main`, `pedalboard-bridge-main`, `pedalboard-midi-main`, `pedalboard-config-main`, `pedalboard-cli-main`, `midi-controller-main`, `led-ring-main`. Treat these as **snapshots**; establish commit/date before trusting for present pi-Stomp or PiPedal behavior. Most legacy pedalboard-controller details are irrelevant to this headless Pi 5 work.
9. **Manufacturer data:** current TAC5212 and LT3042 data sheets and the *new HAT's final* schematic/BOM for codec bring-up. Codec device-tree/driver decisions require primary-source verification; do not infer from the older CM5 system.

**Source discipline:** At the start of every investigation state the URL/path and commit/tag/date actually inspected. Distinguish (1) upstream behavior, (2) Pi 5 observed behavior, (3) proposal. Record conflicts and make a test to resolve them. Do not cherry-pick a configuration solely because it appears in a familiar pi-Stomp or PiPedal document. Upstream statements that predate the Pi 5 or current kernel must be revalidated.

## 5. Correct architecture

```text
                           Other computer / phone
                             Web UI / management
                                     |
                                  Ethernet / Wi-Fi
                                     |
                  Raspberry Pi 5 — headless Debian 13
                  boot from Samsung NVMe / Pimoroni Base
                                     |
                  [One active audio-engine profile at a time]
                           /                     \
                  PiPedal mode                MOD mode
                  ALSA direct                JACK2 + mod-host
                  PiPedal service            MOD-UI / LV2 plugins
                           \                     /
                            ALSA kernel driver
                                     |
                              I²S + I²C
                                     |
                  temporary: HiFiBerry DAC+ ADC Pro
                         or later: NEW TAC5212 HAT
                                     |
                guitar / optional line in -> processing
                          -> headphone / line out
```

The modes differ intentionally; don't force one daemon to sit inside the other mode. Use explicit systemd orchestration with a single-device-ownership invariant, complete service start/stop logging, and recovery if service selection fails. The production HAT's ALSA card identity is **not** yet known.

**Sampling:** 96 kHz is the design target and default for investigations. At 96 kHz, one 32-sample period lasts ~0.333 ms and one 64-sample period ~0.667 ms. Neither figure is a prediction of loopback latency. Actual A/D, D/A, analog filtering, ALSA/JACK buffering, scheduling and plugins add latency. Confirm I²S controller, codec, ALSA driver and individual engines support desired rates/slots before enabling.

## 6. Phase plan — ordered, gated, and reversible

### Phase 0 — Identity, evidence and immutable baseline (FIRST TASK)

**READ ONLY** until user approves a change plan.

- Confirm new project identity and 96 kHz target in the working software repo. Explicitly exclude the legacy CM5 repository.
- Verify live OS: `/etc/os-release`, `uname -a`, `/proc/cmdline`, `lsblk`, `findmnt`, `vcgencmd get_throttled`, `vcgencmd measure_temp`.
- Capture boot and fan configuration, package/kernel versions, systemd enabled/running services, audio subsystems already present, network state and current power/CPU-frequency policies.
- Determine the installed kernel preemption options (not just `CONFIG_PREEMPT=y`), capabilities and IRQ threading; inspect whether dynamic preemption is available.
- Establish update status and whether reboot is required. Do not automatically upgrade during an evidence-gathering pass.
- Check networking/recovery, audio group and RT limits, udev/ALSA enumeration, current swap/zram **without changing them**.
- Store **sanitized** machine inventory, base config and source manifest in version control; never record passwords, keys, tokens, cloud-init password hashes, Wi-Fi secrets.
- Enumerate which decisions are verified vs unknown. Output a concise recommendation table: `setting | current | pi-Stomp approach | PiPedal approach | Pi 5 applicability | proposed action | justification | rollback`.

**Gate:** Baseline exists, no remote-access risk, no changes applied.

### Phase 1 — Evidence-based OS tuning

Study the actual `pi-gen-pistomp` image scripts/configs on the relevant branch and PiPedal service/source, especially:

- Kernel PREEMPT/PREEMPT_RT availability and whether switching improves Pi 5 scheduling enough to justify compatibility/maintenance risk.
- CPU governors (`ondemand` vs `performance`), governor transition latency, thermal/throttling behavior, power consumption and expected NAM load.
- Audio-thread RT permissions, `rtprio`, `memlock`, `LimitRTPRIO`, `LimitMEMLOCK`, resource limits and any priority inversion risks.
- Interrupt priority and affinity. Pi 5 RP1 and I²S/PCIe/NVMe behavior must be **detected from our machine**; do not copy another Pi's IRQ numbers, and do not promote a shared controller IRQ indiscriminately.
- ALSA/JACK service interlocks and autostart; background network/power/SD activity; filesystem mount, journal and writeback behavior.
- Swap/zram, memory pressure, OOM protection, page faults and plugin load times: measure before disabling anything. Keep NVMe reliability.
- Cooling: preserve existing temperature-based PWM control and safety thresholds. Adjust only based on long-duration thermal testing.

**Only after study:** propose each candidate separately with expected benefit, drawback, how to change it, how to measure it, and exact rollback. No folklore-based `isolcpus`, `nohz_full`, irqbalance removal, disabling all services, permanent max CPU clocks or RT kernel swapping without supporting tests.

**Gate:** Reproducible OS config with a known-good kernel boot entry and stable SSH. No reported throttling or new instability.

### Phase 2 — HiFiBerry temporary audio reference

- Identify physical installed model as **HiFiBerry DAC+ ADC Pro**, verify current Raspberry Pi 5/Trixie support and the right Device Tree overlay against current HiFiBerry documentation/source (`hifiberry-dacplusadcpro` has been considered, but confirm for the actual board and OS revision).
- Confirm ALSA playback and capture card, channel map, capture gain, output path, supported rates/formats/period sizes, hardware mixers, clocking, simultaneous full duplex.
- Specifically test **96 kHz** *before* presenting it as working. If the temporary card or driver doesn't support 96 kHz reliably, log that as a **temporary test-device limitation**, not a change to the custom HAT target.
- Avoid changing mixer capture gain while measuring latency; record every setting.
- Verify signal loopback with safe levels and correct output/input (never send powered outputs into incompatible inputs; use attenuation where required).
- Record baseline ALSA behavior before installing either audio engine.

**Gate:** clean, correctly routed capture and playback with repeatable nominal audio quality and known supported sample-rate/period matrix.

### Phase 3 — Install and characterize PiPedal alone

- Use current documented PiPedal release/package suitable for Pi 5 ARM64/Trixie, inspect its install scripts before changing services, and keep a rollback path.
- Study source/config for direct ALSA device access, xrun counters, RT priority and memory policy; avoid JACK unless specifically needed for diagnostic work.
- Configure selected supported 96 kHz mode, actual hardware channels, initial gain and plugin state. Do not assume ALSA card indices are stable across reboots.
- Establish low-complexity straight-through/bypass reference; then LV2 and NAM-type loads.
- Use PiPedal's `pipedal_latency_test` following **the installed version's help and verified systemd service name** (published docs have used both `pipedal` and `pipedald` in different places); record physical wiring and test method. Cross-check with an independent method if practical.
- Test 16/32/48/64/128 frame sizes only as hardware/engine allow, with reproducible XRUN/load and real loopback latency. Do not label a buffer calculation "round-trip latency".

**Gate:** reproducible service boot, stable 96 kHz when supported by temporary card, measured loopback, no unexplained XRUNs under documented workload.

### Phase 4 — MOD/pi-Stomp JACK mode, separately

- Review pi-Stomp's pinned current distribution and exact JACK, mod-host and MOD-UI startup chain, not generic historical PiPedal information.
- Extract only portable OS/JACK/service behaviors; do not copy pi-Stomp-specific display, footswitch, RGB, SPI, ADC or MIDI controller stack.
- Confirm legal/technical availability of dependencies and ARM64/Trixie versions before installation. Explain build/packaging effort rather than assuming a turnkey package works on our vanilla Trixie system.
- Validate sample-rate and period settings at **96 kHz**, channel topology and plugin compatibility; compare the same effects/loads and capture conditions to PiPedal mode.
- Detect XRUNs, startup/recovery failures and potential plugin incompatibility. Avoid any side effects on PiPedal setup.

**Gate:** documented working mode and separate clean process tree, with reproducible measurements.

### Phase 5 — Safe audio-mode switching and product behavior

- Design minimal, maintainable services and dependencies: one active engine; clean shutdown of old ALSA clients; predictable JACK teardown/startup; no racing devices.
- Profile state persisted robustly to NVMe. Defined fallback on failed boot or audio initialization. Avoid audio pops where possible: mute sequencing and routing policy must ultimately consider TAC5212 codec and audio connectors.
- Make remote troubleshooting possible (logs, service status, limited restart). Never break SSH for a UI convenience feature.
- Headless default on power up, configurable mode, explicit state, clear upgrade/rollback path.
- Do not predesign controller firmware or display/UI for this HAT; no separate MCU architecture is established as part of *this* software task; do not import legacy UI decisions.

**Gate:** repeated switches and reboots do not leave competing audio daemons or unusable audio card.

### Phase 6 — Production **TAC5212 HAT** bring-up, after delivery

**Require the manufacturing-revision design and real board. Do not substitute the old CM5 schematic.**

- Save and review actual KiCad schematic/netlist, BOM, JLC files/assembly markings and manufacturer's TAC5212 latest datasheet and binding/driver code. This is a prerequisite for hardware-specific instructions.
- Verify 5 V and 3.3 V header power distribution, LT3042 rail, codec AVDD/IOVDD sequencing, reset and startup delay **before** enabling full audio.
- Verify I²C detection and correct exact codec model/address; verify pins and 96 kHz/32-bit I²S clock directions, bit clock rate and Device Tree/ASoC link. Do not assume a TAC5x1x kernel driver supports every TAC5212 feature or topology.
- Check kernel/driver mainline or out-of-tree support on the **installed** kernel; prefer maintainable small upstreamable overlays/patches and build/package version pinning.
- Establish 96 kHz stereo capture/playback and correct channel mapping. Configure TAC5212 ADC/PGA/gain, filter selection and headphone/line output routing using validated register/API descriptions.
- Independently measure noise, clipping/headroom, low-frequency response, pop/click behavior and headphone loading; report analog issues instead of "fixing" them with undocumented codec registers.
- Compare its latency, error count and audio quality under identical conditions with HiFiBerry where possible. The HiFiBerry remains recovery/test reference.

**Gate:** robust measured direct I²S interface with no mysterious channel swapping, clock mismatch, distorted output or xrun regressions.

### Phase 7 — Stress and release qualification

- Cold/warm reboot × repeated cycles; no keyboard/monitor; Ethernet connected/disconnected, Wi-Fi when configured; volume/gain controlled.
- Sustained idle, typical preset and CPU-intensive NAM+IR+reverb workload; short transients and preset switching; network/web UI usage and NVMe I/O during audio.
- Log temperature, actual CPU frequency, throttle status, memory pressure, audio IRQ/RX/TX counters and xrun rate; measure worst-case rather than just averages.
- Validate no unintended audio pop and safe recovery after service crash; avoid long filesystem write operations on RT paths.
- Build reproducible deployment: pinned versions, Git commits, clear `install`/`verify`/`rollback`, and a machine-readable test-results log.

**Gate:** user-approved acceptance thresholds for max loopback latency, xrun rate, CPU load, thermals and quality. Don't invent thresholds as though approved.

## 7. Test methodology — separate metrics

**At minimum record:** OS/kernel + preemption mode; firmware; governor and actual clocks; cooling state; interface, driver/overlay, ALSA device; rate, format, frames/period, period count; physical cable arrangement; engine version; plugin chain; actual loopback latency (median, worst, samples or ms), XRUN count with observation duration; temperature and throttle flags; any audio artifacts; test date.

- **Scheduling latency** (e.g. `cyclictest`) is a *secondary diagnostic*, not proof of audio latency or a reason to switch kernels on its own.
- **Buffer duration** = frames/rate. At 96 kHz: 16 frames=0.167 ms, 32=0.333 ms, 64=0.667 ms, 128=1.333 ms. **These are not total or round-trip latencies.**
- **Actual round-trip** requires audio output routed physically back to input and a measurement utility able to measure the resulting delay (including converters/analog path). Control amplitude and detect hidden resampling.
- **Stability** means under documented sustained and interrupting workloads; zero XRUNs in a short idle run is insufficient.
- **Audio quality** measurement (noise, THD, frequency response, clipping) is separate from timing measurements. Codecs and coupling networks can contribute measurable latency and frequency-dependent effects.
- **Governor comparison**: first set up a reproducible audio workload. Compare `ondemand`/`performance` transiently with unchanged plugin, sample rate, buffer, room/thermal conditions and monitoring. Report any latency benefit vs temperature/power penalty; don't permanently change by default.
- **RT kernel comparison**: after establishing need, check release support and backup/boot alternative. Compare exact same audio scenarios and note possible newer-kernel driver regressions and NVMe/Ethernet impact.

## 8. Repo, configuration, logging and response format

Suggested documentation/software layout **only if compatible with the current contents of `TAC5212_HAT`** (adapt to existing files; no automatic reorganization):

```text
README.md                    # Current goals, live state, how to operate
AGENTS.md                   # Short recurring Codex rules
CODEX_HANDOFF.md            # This document (reference only)
docs/
  decisions.md              # User-approved decisions, date, reason
  source-inventory.md       # Upstream URL+commit/version and relevant files
  baseline.md               # Sanitized read-only inventory, known OS state
  tests.md                  # Reproducible benchmark procedures
  results/                  # Measured results and machine state for each run
  recovery.md               # Proven SSH/boot and rollback procedures
scripts/
  inspect/                  # Read-only diagnostics (safe to run repeatedly)
  setup/                    # Reviewed small idempotent setup steps
  benchmark/                # Reproducible comparisons
systemd/                    # Versioned overrides and mode services *after approval*
```

If a script would be difficult to audit, prefer a few simple commands or an established package. Do not add dependencies or install tracking agents without a clear benefit.

For each Codex interaction use a **short answer first**:

1. What source(s) were verified and what was found.
2. Exact recommendation, including why it is *better than alternatives* for **this Pi 5 / HAT**, and what remains unknown.
3. Up to 1–4 precise commands or changes **only if already justified**; distinguish read-only/temporary/persistent.
4. Expected output and how to roll back.
5. Stop and ask for output before the next potentially disruptive change.

Never claim a command was run or an issue fixed without actual evidence. Never make changes to network, boot, power, kernel, IRQ policy or anything production-critical without the owner's approval. Don't generate extensive prose when a succinct diff or table suffices.

### GitHub and change authority

- The repository is **`https://github.com/arjepsen/TAC5212_HAT`**. The user created it; Codex must inspect its *actual checked-out files* first. Do not assume it remains empty or automatically replace its README, KiCad files, or directory layout.
- **No automatic remote actions.** Never create branches, commit, push, open PRs, or alter GitHub settings unless the user explicitly requests that specific action. For each proposed local/system change, state whether it affects the Pi, the working tree, or GitHub.
- Do not modify or delete schematic/BOM/manufacturing source merely to enable software work. Source observations and recommended changes are different from implementation permission.
- Keep any machine inventory free of credentials, SSH keys, tokens, Wi-Fi passwords, cloud-init user data, and personal configuration secrets.

## 9. The correct **FIRST Codex assignment** — copy this brief, not all future tasks

> We are working in **`arjepsen/TAC5212_HAT`** on a new Raspberry Pi 5 guitar-audio HAT project targeting **96 kHz**, not the older, unrelated pedalboard project. The Pi 5 runs Debian 13/Trixie Lite, boots NVMe, uses Active Cooler, and has preemptible 6.18.50 kernel, `ondemand` governor; hardware HAT is being manufactured. Temporary soundcard is HiFiBerry DAC+ ADC Pro. We want PiPedal (direct ALSA) and MOD/pi-Stomp (JACK) as separate modes, and lowest stable measured latency.
>
> **Read only for now.** Inspect the relevant *current* `TreeFallSound/pi-gen-pistomp` OS tuning scripts and PiPedal package/service source plus their dates/commits. Compare them with the actual OS if SSH is available. Focus on kernel PREEMPT_RT state, CPU governor, audio RT permissions/limits, threaded interrupts/IRQ practices, memory/swap/writeback policy, background services and safe headless boot. Flag Pi 5/RP1/NVMe-specific risks, and any documentation contradictions. **Do not change anything or install a benchmark yet.** Deliver a small table of verified findings and ranked reversible OS proposals with clear rationale and rollback. Ask for the machine read-only results you genuinely need, then proceed incrementally after approval.

## 10. Prohibited assumptions / common failure traps

- **Target 96 kHz; do not substitute a different operating rate without an explicit user decision.** Do not import hardware or software settings from unrelated projects.
- Never call `CONFIG_PREEMPT=y` or low idle temperature "real-time audio verified"; measure.
- Never assume current Raspberry Pi OS uses the same RT kernel advertised by an upstream website; check machine.
- Never claim a codec supports a rate merely because the silicon data sheet supports it; verify the actual Linux ASoC machine driver, clocks and configuration.
- Never mix MOD/JACK and PiPedal/ALSA ownership or compare latency using mismatched physical paths/levels or unequal buffer settings.
- Never replace a working, updated NVMe system with pi-Stomp OS or a custom kernel without verified recovery and explicit owner approval.
- Never invent source filenames, Git revisions, pin assignments, ALSA card numbers, IRQ numbers or benchmark results.
- Never optimize all services away; preserve networking and long-term stability. Keep cooling failsafes.
- Never read a historical source bundle as if it were today's actual design.

---

### Verification and limitations of this handoff

**Verified from user-provided terminal output:** current root/boot devices and sizes; running kernel and `CONFIG_PREEMPT=y`; absent `/sys/kernel/realtime`; governor/clocks; fan config and measured RPM/PWM; idle temperature and throttle flag.

**Grounded in project discussions (not independently re-read from final manufactured KiCad files):** distinct Pi 5 HAT and 96 kHz target; TAC5212, OPA2365, LT3042 and digital/analog topology as *reported design intentions*; manufacturing status as reported. Final implementation is not independently verified. Treat these as design history until the exact fabricated revision is supplied.

**Checked upstream for this handoff:** pi-Stomp's image/build repo describes RT kernel and MOD/JACK distribution; PiPedal documents Pi 5/Trixie support and direct ALSA access; Raspberry Pi documents firmware-controlled Pi 5 fan thresholds. **Codex must re-check the live upstream revisions and pin them before making changes.** Current upstream versions/config are to be pinned by Codex when implementing.

**Not yet verified:** current GitHub repository file contents, exact final HAT netlist/order files, I²S clock compatibility, TAC5212 Linux ASoC implementation, working 96 kHz mode on either device, actual loopback latency, stable real-time scheduling under load, and which optimizations ultimately prove worthwhile.
