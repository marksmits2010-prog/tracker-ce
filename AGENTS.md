# Agent guidance: Tracker Mini Community Edition

This repository builds Community Edition firmware for the Original Tracker, Tracker+, and Tracker Mini. Keep work focused on the **Polyend Tracker Mini** unless the task explicitly asks for broader device support. The Mini uses the IMX firmware platform (`PLATFORM=IMX`, `PRODUCT=MINI`) and handheld controls.

## Planning and decisions

- Use [`PLAN.md`](PLAN.md) to track proposed and active feature work. Before implementing a feature, record its user need, Mini-specific behavior, scope, constraints, and a concrete validation plan. Keep the plan current as work progresses; mark completed or deferred items with their outcome rather than leaving them ambiguous.
- Use [`DECISIONS.md`](DECISIONS.md) for durable choices that affect future work, such as input mappings, compatibility approaches, platform boundaries, or memory tradeoffs. Record the date, context, decision, and consequences. Link related plan items or code/docs where useful.
- Keep both files concise and factual. Plans describe work that may change; decisions record choices that have been made. Do not treat a proposal in `PLAN.md` as approved implementation direction unless it is explicitly marked accepted.

## Before changing code

- Read [`CONTRIBUTING.md`](CONTRIBUTING.md) for project philosophy, compatibility rules, build expectations, and hardware testing guidance.
- Use [`docs/developer_guide.md`](docs/developer_guide.md) for platform macros, hardware differences, architecture, and the development console.
- Check [`docs/memory_map.md`](docs/memory_map.md) before changing memory use, linker sections, buffers, or audio resources.
- Consult the focused guides in [`docs/`](docs/Home.md) when touching the file browser, synth SDK, themes, wavetable features, or modular effects.
- Check the existing implementation and build configuration before assuming a behavior is Mini-specific. `PLATFORM_IMX` covers both Tracker+ and Mini; `TRACKER_MINI` identifies Mini-only behavior.

## Tracker Mini implementation priorities

- Preserve the Mini's handheld interaction model. It has a D-pad, `interfaceButtonMiniMiddle`, and Shift combinations; it has no pad grid or desktop jog wheel. When changing UI navigation or shortcuts, account for those controls and document the user-visible behavior.
- Keep hardware-specific code guarded at the narrowest appropriate level. Use `TRACKER_MINI` for Mini-only behavior and `PLATFORM_IMX` only for behavior shared by Tracker+ and Mini.
- Preserve project, instrument, patch, and configuration file compatibility. Do not reorder or renumber serialized enums or move serialized fields without an explicit compatible migration.
- Protect real-time audio paths: do not allocate memory, access the SD card, or block in audio-rate callbacks.
- Treat RAM, flash, and CPU time as constrained resources. Check memory reports and the memory map for changes that add buffers, large state, or audio work.

## Build and review

- For a Mini firmware image, use `make mini`. The output is staged under `build/` as `TrackerMini_CE_*.ptf`.
- Follow the repository contribution guidance for verification: changes to shared firmware should compile for all three targets (`make all`); plugin changes may also require `make ptfx` or `make psyn` as described in `CONTRIBUTING.md`.
- Do not claim hardware verification unless the change was exercised on a physical Tracker Mini. State which devices were built and which were tested when summarizing the work.
- Keep changes scoped to the requested behavior, and update the relevant developer or user documentation when behavior or workflows change.
- Update `PLAN.md` when feature status or scope changes, and add to `DECISIONS.md` when implementation establishes a durable design choice.
