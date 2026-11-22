# Tracker Mini Feature Plan

Use this file to track proposed and active feature work for the Polyend Tracker Mini Community Edition. Keep entries focused on the user need and Mini behavior. Mark proposals clearly; listing an idea here does not mean it has been accepted.

## Proposed

<!-- Add ideas with a short title, user need, expected Mini interaction, constraints, and validation notes. -->

### Proposed: Per-track polymetric length and pattern sync

- **User need:** Let each track cycle at its own step length while the pattern continues at its overall length, creating polymetric repetition without changing the pattern's shared timeline.
- **Expected Mini behavior:** Provide a per-track option to enable polymetric length, set that track's step length for the current pattern, and set how many repeats of that track cycle occur before it synchronizes to the pattern's step index. Expose the controls through the Mini's handheld UI (D-pad, `interfaceButtonMiniMiddle`, and Shift combinations); do not rely on pads or a jog wheel. Keep the option and values scoped to each track in each pattern.
- **Scope:** Planning proposal for Tracker Mini. Determine whether the eventual implementation belongs in shared sequencer code or needs device-specific UI; preserve existing pattern playback when the option is off.
- **Constraints:** Preserve project compatibility; do not reorder serialized enums or shift serialized fields without a compatible migration. Keep sequencing work bounded and real-time safe. Review memory use and existing pattern/track timing behavior before implementation. Document the user-facing control flow and compatibility behavior.
- **Validation plan:** Inspect the sequencer's pattern and track step-index handling; define expected sync behavior for lengths that divide and do not divide the overall pattern length; verify that disabled tracks retain current behavior and that the option/values survive save and reload. Build all three targets if the sequencer implementation is shared, and test the Mini interaction and playback on hardware.
- **Clarification needed before implementation:** Confirm what “how many pattern repetitions ... to wait to sync up to the pattern's step index” means: should the track keep looping its own length and then, after the chosen number of track cycles, jump to the step index corresponding to the global pattern position; or should the chosen number count full pattern repetitions before phase-resetting the track? Also confirm whether the track length counts steps (and its allowed range) and whether “sync” means resetting its playhead to the global pattern step, or merely aligning its next cycle boundary with the pattern boundary.

## Accepted / In progress

<!-- For each feature: goal, scope, Mini-specific UX, technical constraints, and verification plan. -->

## Completed or deferred

<!-- Record outcome and link relevant PRs, commits, docs, or decision entries. -->
