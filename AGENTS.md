# AGENTS.md — TAC5212_HAT / Raspberry Pi 5

Read `CODEX_HANDOFF.md` selectively for project-specific details, and keep this file short to conserve context.

## Project identity: non-negotiable

- The **`https://github.com/arjepsen/TAC5212_HAT`** project: new GPIO audio HAT for **Raspberry Pi 5**, currently being produced, **96 kHz TARGET**; temporary interface is **HiFiBerry DAC+ ADC Pro**.
- **NOT** the unrelated legacy `Pedalboard-integrated-audio` project. Never transfer its decisions, samples rates, schematics, UI architecture or code.
- New HAT reported design history: **TAC5212**, **OPA2365**, **LT3042**, GPIO I²S/I²C; **do not treat any specific connection, part suffix, voltage or pin assignment as confirmed without the submitted netlist/BOM**.
- Pi 5: Raspberry Pi OS Lite ARM64 Debian 13 Trixie on 256 GB Samsung NVMe/Pimoroni NVMe Base, official Active Cooler; headless SSH over Ethernet; kernel `6.18.50+rpt-rpi-2712` with `CONFIG_PREEMPT=y`, **not proven PREEMPT_RT**. `ondemand` governor, 1.5–2.4 GHz, auto fan first threshold 30 °C.

## Software objective

- Lowest **reliable measured** end-to-end guitar-to-output latency without XRUNs while using real LV2/NAM loads.
- Borrow verified ideas from **TreeFallSound/pi-gen-pistomp**, pi-Stomp, **rerdavies/pipedal** and MOD: **MOD/JACK** and **PiPedal/ALSA** in **mutually exclusive modes**.
- Optimize/headless OS before configuring the temporary HAT, then benchmark, then bring up new TAC5212 board once received. **96 kHz remains the goal**; test device limitations do not change it.

## Working method

1. **Read sources first**: exact repo path + commit/tag/date; compare to actual machine; no folklore tuning or legacy board assumptions.
2. Before any change: show current state, alternatives, measurable reason, risks, rollback. Obtain approval before persistent/disruptive edits and any repository write.
3. Read-only initial phase: inspect this repo and verify OS/kernel/RT, governor, services/limits, IRQs and Pi 5/RP1/NVMe implications. **Do not start with generic cyclictest** or install and tweak everything at once.
4. Preserve NVMe boot, SSH, Ethernet, thermal safety and existing recovery path. Do not blindly change kernel, IRQ priorities, clocks, networking, swap or OS image.
5. Benchmark real audio once a supported interface is configured: actual loopback latency, XRUNs, plugin load, heat/throttling, test matrix of rates/periods. Distinguish frame duration from physical round-trip delay.
6. Do not assume exact new HAT circuitry or codec registers without final KiCad/manufacturing revision and manufacturer docs; no hardware changes without approval.
7. Responses to owner: **short and precise**. One issue at a time, typically at most four explicit new commands/changes. State verified, not verified, and expected output. Keep secrets out of logs/repo. **Do not commit, push, or modify GitHub without explicit user instruction.**

## FIRST task

Research-only comparison of actual current pi-Stomp OS tuning and PiPedal real-time requirements versus observed Pi 5 6.18.50/Trixie setup. Provide sourced, prioritized, reversible changes. **No changes before owner's approval.**
