## Remix API Changelog

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.4.2]

### Added
- MaterialInfoOpaqueEXT.displaceOut

### Changed
- renamed MaterialInfoOpaqueEXT.heightTextureStrength to MaterialInfoOpaqueEXT.displaceIn

### Fixed

### Removed


## [0.4.3]

### Added
- remixapi_MaterialInfoOpaqueSubsurfaceEXT.subsurfaceDiffusionProfile
- remixapi_MaterialInfoOpaqueSubsurfaceEXT.subsurfaceRadius
- remixapi_MaterialInfoOpaqueSubsurfaceEXT.subsurfaceRadiusScale
- remixapi_MaterialInfoOpaqueSubsurfaceEXT.subsurfaceMaxSampleRadius
- GameStateStore keys `__weather.drift_speed` and `__weather.drift_intensity` — plugin-controlled cloud-drift speed and intensity multipliers. Both default to 1.0 when unset. Smoothed inside the renderer with tau = 1.0s. See [`docs/integrators/weather-presets.md`](integrators/weather-presets.md) section 8 for the recommended per-preset values and integration pattern.

### Changed

### Fixed

### Removed
- `rtx.atmosphere.sunDisc` (GameStateStore/config key) — removed. The option had no consumer (the sun disc is rendered via the sun-as-distant-light / NEE path); setting it had no effect.


## [0.1000.0]

Remix Plus adopts its own ABI version line (reserved MINOR `1000`), distinct
from stock NVIDIA dxvk-remix `0.6.x`. Because the runtime treats every minor as
breaking while MAJOR is 0, this version makes the runtime reject binaries built
against stock Remix `0.6.x` or against older Remix Plus `0.6.x` — the
`remixapi_Interface` layout and `remixapi_InstanceCategoryBit` ABI differ
between them. Rebuild plugins/hosts against this header.

### Added
- `REMIXAPI_INSTANCE_CATEGORY_BIT_SMOOTH_NORMALS` (bit 24) — the upstream name
  for the category previously exposed (under the fork) only as `LEGACY_EMISSIVE`.
  Use `SMOOTH_NORMALS`; the old alias has been removed (see below).

### Changed
- `remixapi_InstanceCategoryBit` bit values now match upstream NVIDIA exactly.
  An earlier fork build had shifted `IGNORE_ALPHA_CHANNEL` to bit 8 (cascading
  bits 8–20 up by one) to mirror the internal `InstanceCategories` order; this
  is reverted. The C↔internal mapping in `toRtCategories()` is by-name, so the
  public bit values are free to — and now do — match upstream. **Breaking** for
  any consumer that had serialized or hard-coded the shifted bit values.
- `remixapi_Interface.SetCameraMediumMaterial` moved from the middle of the
  struct (between `SetupCamera` and `DrawInstance`) to immediately after
  `Present`, mirroring upstream's canonical layout (upstream `2bac8874`). Fork
  extension functions remain appended after it. **Breaking** struct-offset
  change; rebuild consumers. The `sizeof(remixapi_Interface)` sentinel is
  unchanged (move, not add/remove).
- `REMIXAPI_VERSION` is now `0.1000.0` (was `0.6.4`).

### Fixed
- `remixapi_AutoInstancePersistentLights` no longer emits a per-frame
  `LockDevice` + empty `EmitCs` on the native-D3D9 present path when no external
  (C-API) light has ever been registered and no C-API scene work is queued.
  This empty per-frame dispatch disturbed the light pipeline for native-only
  consumers, manifesting as "persistent lights break all lights / heavy
  flicker." Genuine C-API light consumers are unaffected (the persistent
  re-instancing path is preserved).

### Removed
- `REMIXAPI_INSTANCE_CATEGORY_BIT_LEGACY_EMISSIVE` is **removed**. Its name
  implied emissive behavior, but it routed to bit 24 / `InstanceCategories::SmoothNormals`
  — so any caller using it got a silent wrong-category result. Removing it turns
  that into a compile error; use `REMIXAPI_INSTANCE_CATEGORY_BIT_SMOOTH_NORMALS`.
  Source-only break — the bit layout, enum size, and struct offsets are unchanged
  (bit 24 still exists as `SMOOTH_NORMALS`), so no binary/ABI change and the
  `0.1000.0` version line is unaffected.


## [0.1000.1] — 2026-08-08

Additive only: two new `remixapi_InstanceCategoryBit` enumerators. No struct
layout, enum size, or function-pointer offset changes, and the compat check
ignores PATCH while MAJOR is 0 — so binaries built against `0.1000.0` keep
working and no consumer rebuild is required. Rebuild only to use the new bits.

### Added
- `REMIXAPI_INSTANCE_CATEGORY_BIT_VIEW_MODEL` (bit 26) — tags first-person
  arms/weapon instances so they render through the view-model pass. Previously
  the API had no way to express this at all: `DrawInstance` hardcoded
  `CameraType::Main` on every external draw, so `REMIXAPI_CAMERA_TYPE_VIEW_MODEL`
  could be submitted but no instance could ever reference it. This is the only
  category bit that does not map to an internal instance category — it selects
  `CameraType::ViewModel` for the draw instead.

  Tagging alone is not sufficient. The pass additionally requires a
  `REMIXAPI_CAMERA_TYPE_VIEW_MODEL` camera submitted every frame (it is not
  synthesized — the perspective correction is built from both that camera and the
  world camera) and `rtx.viewModel.enable = True`, which defaults to `False`.
  See [View-model instances](RemixApi.md#view-model-instances).
- `REMIXAPI_INSTANCE_CATEGORY_BIT_HAIR_CARDS` (bit 25) — **reserved, not
  implemented.** Allocated by upstream NVIDIA (REMIX-2901). Declared here so the
  bit stays claimed and this enum matches upstream byte-for-byte; without it the
  fork's next-free-bit would have collided with upstream on the next sync.
  Setting it is accepted and ignored.

### Changed
- `REMIXAPI_VERSION` is now `0.1000.1` (was `0.1000.0`).

### Fixed

### Removed
