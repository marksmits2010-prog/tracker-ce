# Tracker Mini Development Decisions

Record durable design decisions that will help future contributors understand why the firmware works as it does. Keep entries factual and concise. Use [`PLAN.md`](PLAN.md) for tentative or active feature proposals.

## Decision log

These entries summarize established project choices documented in the repository. Dates are omitted where the original decision date is not recorded.

### Community Edition changes should preserve the Tracker workflow

- **Context:** The firmware serves musicians with established Tracker workflows across several devices.
- **Decision:** Prefer broadly useful improvements that fit the existing workflow and controls. Discuss major workflow or architectural changes with the community first; keep highly specialized experiments in forks or loadable plugins unless community feedback supports including them in CE.
- **Consequences:** Feature proposals should explain their user benefit and Mini interaction model. UI changes must account for handheld controls.
- **References:** [`CONTRIBUTING.md`](CONTRIBUTING.md#1-project-philosophy-preserving-the-spirit-of-the-tracker), [`AGENTS.md`](AGENTS.md#tracker-mini-implementation-priorities).

### Shared firmware stays cross-device buildable; device behavior uses explicit guards

- **Context:** The repository shares application code across Original Tracker, Tracker+, and Tracker Mini, while the Mini has a distinct handheld interface.
- **Decision:** Keep shared changes buildable for all three targets. Use `PLATFORM_IMX` for behavior shared by Tracker+ and Mini, and `TRACKER_MINI` for Mini-only behavior.
- **Consequences:** Mini-specific changes should be narrowly guarded and shared changes should avoid breaking the other targets. User-visible input behavior should be documented.
- **References:** [`CONTRIBUTING.md`](CONTRIBUTING.md#2-core-pull-request-rules), [`docs/developer_guide.md`](docs/developer_guide.md#5-conditional-compilation-targeting-specific-platforms-or-devices), [`AGENTS.md`](AGENTS.md#tracker-mini-implementation-priorities).

### Existing serialized project and instrument formats remain compatible

- **Context:** Projects and instruments are exchanged as files, and firmware updates must continue to load existing user content.
- **Decision:** Do not reorder or renumber serialized enums or shift fields in project, patch, or configuration structures without a compatible migration. Preserve and validate legacy `.pti` layouts during loading.
- **Consequences:** Additive data changes need explicit versioning and loader handling; binary layout assumptions should be protected by size checks and tests already present in the codebase.
- **References:** [`CONTRIBUTING.md`](CONTRIBUTING.md#3-technical-requirements--safety-checklist), [`docs/developer_guide.md`](docs/developer_guide.md#1-copy-on-write-workspaceinstruments-incremental-project-saver--hybrid-sd-streaming), [`core/source/mtStructs.h`](core/source/mtStructs.h).

### Large sample files use header-only workspace copies and eligible SD streaming

- **Context:** Copying full PCM payloads during project open/save consumes time and SDRAM, while playback still needs reliable immediate note starts.
- **Decision:** Use header-only Copy-on-Write workspace instrument files and stream eligible large samples from SD with an SDRAM pre-cache and background ring-buffer refill. Promote samples to resident RAM when editing or using a non-streamable play mode requires it.
- **Consequences:** Changes to instrument loading, saving, play modes, or sample editing must preserve promotion/demotion behavior and project self-containment on Save As. Tracker Mini and Tracker+ can stream slice modes; Original Tracker has more restrictive streaming support.
- **References:** [`docs/developer_guide.md`](docs/developer_guide.md#1-copy-on-write-workspaceinstruments-incremental-project-saver--hybrid-sd-streaming), [`docs/memory_map.md`](docs/memory_map.md#2-external-sdram-pool--reclaimable-sample-ram-external_ram).

### Audio-rate code must remain real-time safe

- **Context:** Audio callbacks run on a fixed schedule; blocking or unpredictable work can cause audible dropouts.
- **Decision:** Do not allocate or free heap memory, perform SD/FatFs I/O, or block in audio-rate functions. Move file and other potentially slow work to the main thread or offline/modal paths.
- **Consequences:** Audio features need preallocated state and a clear handoff between control work and rendering. Avoid operations such as heap-growing containers and float formatting in audio callbacks.
- **References:** [`CONTRIBUTING.md`](CONTRIBUTING.md#3-technical-requirements--safety-checklist), [`docs/memory_map.md`](docs/memory_map.md#6-memory-placement-rules--developer-gotchas).

### Memory use must respect fixed hardware budgets and share temporary buffers

- **Context:** Tracker Mini runs on IMX hardware with tight internal SRAM and finite SDRAM shared by samples, synths, effects, and UI systems.
- **Decision:** Keep large or non-real-time buffers out of default `.bss`; explicitly initialize `__NOINIT(EXTERNAL_RAM)` allocations; reuse the shared scratch buffer for modal/offline work when its partitions permit it. Preserve protected bootloader flash origins.
- **Consequences:** New persistent buffers require a documented placement and budget review. Simultaneous scratch-buffer consumers must respect assigned regions, and linker memory output should be reviewed for firmware changes.
- **References:** [`docs/memory_map.md`](docs/memory_map.md#1-hardware-memory-regions--baseline-headroom), [`docs/memory_map.md`](docs/memory_map.md#3-shared-768-kb-scratch-buffer-map-externalramscratchbuff), [`README.md`](README.md#4-flashing-on-hardware--emergency-recovery).

### Synth engines and Sample Editor effects use loadable plugin formats

- **Context:** Optional DSP features must fit within firmware memory and remain extensible for community authors.
- **Decision:** Distribute synth engines as `.psyn` plugins loaded from the SD card and Sample Editor effects as `.ptfx` plugins, rather than requiring each engine/effect to be built into firmware.
- **Consequences:** Plugin ABI, memory limits, and documented SDK constraints are part of compatibility. Firmware changes to plugin loading should preserve existing assets and author workflows.
- **References:** [`docs/custom_synth_sdk.md`](docs/custom_synth_sdk.md), [`docs/modular_effects_system.md`](docs/modular_effects_system.md), [`docs/memory_map.md`](docs/memory_map.md#4-synthesizer-sdram-pool-psyn_ram_pool_size--imx).
