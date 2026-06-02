# Engineering Standards — brs-desktop (AudioShelf automation fork)

> **Scope.** This document governs changes made to **our fork** of `brs-desktop`, the Electron
> "Roku device" we own and are overhauling into the **trustworthy oracle** for device-free testing
> of the `AudioShelf-Roku` channel. It is additive to the upstream
> [`contributing.md`](./contributing.md). See also the engine's
> [`brs-engine/docs/ENGINEERING_STANDARDS.md`](../../brs-engine/docs/ENGINEERING_STANDARDS.md) and
> the channel's `AudioShelf-Roku/docs/`.

---

## 1. Cardinal principle — the desktop must behave like a real Roku

brs-desktop is the **oracle**: when we test in it instead of on hardware, its behavior is the
verdict. So it inherits the engine's fidelity rule — **never more permissive than a real device** —
and adds one of its own: **never silently diverge from hardware in rendering, input, or media.**

> **Validated 2026-06-02:** AudioShelf was sideloaded to a physical Roku and confirmed to behave
> **exactly** as it does in brs-desktop. That equivalence is the asset we protect — every change
> must preserve it. Unlike the headless Node CLI, the desktop runs the engine in a worker thread,
> so **SceneGraph Tasks execute here** (the CLI cannot run them yet).

---

## 2. Preserve the automation hooks (do not break these)

The device-free harness drives the desktop through these interfaces. Treat them as a contract —
changes that alter their behavior need a corresponding harness update and a note in the PR:

| Hook | Port / mechanism | Used for |
|------|------------------|----------|
| ECP remote | `:8060` — `POST /keypress/<key>`, `GET /query/...` | Simulated D-pad/OK input + `active-app`/`device-info` queries |
| Registry telemetry | `GET /query/registry/dev` → section `telemetry` | App/scene state assertions (`writeTelemetry` from the channel) |
| Screenshot capture | engine frame → PNG | Visual verification |
| Telnet console | `:8085` | Engine logs (note: **not** BrightScript `print`) |

---

## 3. Operational discipline (hard-won; keep it)

- **Launch bare, then open the package.** Start with `npx electron .` (no `-o <zip>`) and load via
  **File → Open App Package**. Launching with `-o <zip>` races the engine startup and yields a
  black screen — this cost us multiple debugging sessions; it is a launch bug, not a channel bug.
- **Trust the screen, not stale telemetry.** The dev registry **persists across runs**, so
  `GET /query/registry/dev` can return values from a *previous* launch. Verify a fresh boot
  visually (or clear/correlate the registry first); do not certify a boot from telemetry alone.
- **Repoint to the local engine** when validating engine changes, so the desktop exercises your
  edits and not the published npm build (see [`build-from-source.md`](./build-from-source.md)).

---

## 4. Code conventions

Same as the engine fork: **minimal additive diffs**, match existing patterns rather than inventing
new ones, comment the *why*, do **not** mass-reformat pre-existing files, keep tests green, and
distinguish pre-existing issues from your own. SceneGraph support here is **alpha** — when the
channel needs a feature the desktop lacks, decide per task whether to work around it in the channel
or close the gap in `brs-engine`/`brs-desktop` (we own both).
