---
name: 3d-modeling
description: Use when creating or refining a 3D object and correctness depends on shape, proportion, and spatial reading, not just code or dimensions. Applies to CAD, procedural modeling, mesh generation, or scripted 3D workflows that should be validated with exported renders and repeated comparison.
---

# 3D Modeling

Build 3D objects with a render-driven loop instead of editing geometry blind. Treat validation renders as a required part of the modeling process, not an optional afterthought.

## Core workflow

Use this loop:

1. Understand the target object and what defines correctness.
2. Start with a short reference web search when the object may have published dimensions, naming conventions, standards, or common real-world proportions. Use that search to separate known facts from assumptions before modeling.
3. Choose the simplest editable modeling approach that can express the form.
4. Build or revise the model.
5. Export validation renders from fixed views.
6. Open and inspect the rendered images directly.
7. Compare the renders against the intended object.
8. Adjust the model and repeat.

Do not rely on dimensions alone when the visual reading is still wrong.

For mechanical or standardized parts, do not guess dimensions that are easy to verify from standards tables, manufacturer drawings, or comparable real products. Record which dimensions are sourced and which are inferred.

## Modeling rules

- Prefer parametric or otherwise editable models when possible.
- Keep known dimensions, assumptions, and inferred geometry separate. 
- Model the correct object class first. Do not mistake a matching silhouette for the right physical form.
- Lock the main volume and silhouette before adding fine details.
- If the chosen modeling strategy is fighting the object, change strategy rather than layering hacks.

## Tool choice

- Default to `CadQuery` for scripted 3D creation unless runtime verification fails after probing and installation attempts.
- Prefer `CadQuery` for scripted parametric solids, mechanical parts, fixtures, holders, brackets, enclosures, and other dimension-driven objects.
- Prefer mesh or DCC tools when the object is organic, sculptural, or surface-led rather than dimension-led.
- Prefer the modeling tool's direct render/export path before introducing a second viewer.
- If using `CadQuery`, prefer `cadquery.vis.show(..., screenshot=..., interact=False)` for validation renders when available, and run the `cadquery.vis.show` screenshot render path outside the sandbox. Default to smooth shaded renders with `edges=False`; only enable edge overlays when they help diagnose geometry.
- Do not use SVG validation outputs as the primary review artifact when direct PNG rendering is available. Prefer direct PNG renders from the 3D tool.
- With `cadquery.vis.show`, do not assume camera settings are safely isolated across multiple renders in one process. If views look inconsistent or repeated, render each view in a fresh process with explicit camera parameters.

## Environment briefing

Before choosing or installing tools, check whether the workspace already contains a working modeling environment.

- Look for project-local environment signals first, such as `pyproject.toml`, `uv.lock`, `.venv`, existing model scripts, or existing export/render scripts.
- Prefer reusing the workspace's known-good runner over creating a fresh environment.
- For Python-based workflows, prefer `uv run ...` or the project-local interpreter before trying global `python` or `pip`.
- Verify the exact command path that works in this workspace before concluding a tool is unavailable.
- When `uv` is sandboxed and cannot write to `~/.cache/uv`, set `UV_CACHE_DIR` to a workspace-local path such as `.uv-cache`.

For `CadQuery`, treat workspace reuse as mandatory and probe in this order:

1. Try the project runner first, for example `uv run python -c "import cadquery as cq; print(cq.__version__)"` or the equivalent project-local interpreter.
2. If the workspace has no suitable environment yet, create or reuse a local `uv` virtualenv before trying anything global. Prefer Python `3.11` or `3.12` for `CadQuery` unless the workspace already has a known-good version:
   `env UV_CACHE_DIR=.uv-cache uv venv --python 3.11 .venv`
3. Probe the local env directly, for example `./.venv/bin/python -c "import cadquery as cq; print(cq.__version__)"`.
4. If import fails because `CadQuery` is missing, install it into that local env instead of switching tools:
   `env UV_CACHE_DIR=.uv-cache uv pip install --python .venv/bin/python cadquery`
5. If the probe hangs or fails only inside the sandbox, retry the same probe outside the sandbox before treating it as a dependency failure.
6. Verify the installed runtime with a minimal solid-generation command and one real export, such as a small `STEP` file, before using it for the actual model.
7. Only if installation or runtime verification actually fails should you conclude that `CadQuery` is unavailable and fall back to another stack.

Do not assume that a failed global install path means the tool is unavailable on the system. A working local environment may already exist.
Do not skip directly to a fallback tool just because `CadQuery` is absent on first probe.
Do not treat a sandbox-only hang as proof that `CadQuery` is broken. Reuse the verified escalated execution path for exports and renders if needed.

## Validation renders

After each meaningful geometry change, generate renders from the current model.

- Remove stale render artifacts before creating a new validation set so review never mixes old and new images.
- Prefer rendering directly from the source 3D geometry.
- Export validation renders as `PNG` by default. For shaded review images, suppress tessellation and triangle overlays unless they are needed for debugging.
- Do not substitute `SVG` line exports for validation renders unless direct PNG rendering is genuinely unavailable after probing the native render path.
- Use images large enough for comparison, typically `1200` to `1600` pixels on the long side.
- Center the model before rendering when possible so framing stays stable.
- Use the same camera set across iterations.
- After every render pass, open the generated images and inspect them before declaring the geometry acceptable or moving on.
- If the renders are clipped, badly framed, or ambiguous, fix the render setup before changing geometry again.
- If a renderer appears to reuse or drift camera state between views, split the render job into one process per view before changing geometry.

## Standard view set

Export at least these views:

1. `reference`
3/4 hero view aligned as closely as possible to how the object is intended to be understood.

2. `top`
Shows plan shape, openings, and left-right relationships.

3. `front`
Shows visible silhouette and proportional read.

4. `side`
Shows depth, profile, and section-like behavior.

Add one or more detail views when a specific region is uncertain or distinctive.

## Comparison method

- Judge overall likeness using the `reference` view.
- Use `top`, `front`, and `side` to diagnose why the object is wrong.
- Prefer changing one geometric idea at a time, then rerender the same view set.
- The render loop is incomplete until the agent has actually viewed the generated images and checked that they are both visually clear and geometrically credible.
- Do not keep changing camera angles between iterations. That destroys comparability.

## Environment and tooling

- Prefer reproducible, scripted exports.
- For Python-based workflows, prefer `uv` with a project-local environment and lockfile.
- When `CadQuery` is missing, install it into the probed workspace environment first, then rerun the verification probe before considering any alternative tool.
- For `CadQuery`, prefer a local `.venv` runner such as `./.venv/bin/python model.py` once it has been verified.
- Export exchange formats appropriate to the task, such as `STEP`, `STL`, `OBJ`, or `glTF`.
- Treat `cadquery.vis.show(..., screenshot=..., interact=False)` as an outside-sandbox execution path for validation renders, not an in-sandbox default. Request escalation early and reuse the approved execution path for later `cadquery.vis.show` screenshot renders.
- When `CadQuery` is the modeling tool, validation image generation should normally mean direct `PNG` screenshots from `cadquery.vis.show`, not SVG exports plus later conversion.
- For `CadQuery` screenshot rendering, prefer explicit `position`/`focus`/`viewup` camera definitions over loose angle-only settings when reproducibility matters, and default to `edges=False` so screenshots read as object surfaces rather than meshes.
- If multiple validation views are needed from `CadQuery`, prefer a subprocess-per-view render pattern to avoid viewer state leaking across outputs.
- Distinguish environment failure from sandbox/runtime failure. If a tool works only outside the sandbox, reuse that execution path instead of reinstalling or switching stacks immediately.

## Expected outputs

Aim to produce:

- editable source model
- exported interchange files
- for multipart designs, both per-part exports and at least one combined export showing the closed or intended assembled state
- rendered validation images
- concise notes listing assumptions, unresolved uncertainties, and the next geometric corrections

## Failure modes

- Matching one view while failing in others
- Overfitting details before the main form is correct
- Treating bad renders as geometry problems
- Validating only by measurements when the object still reads wrong
- Modeling a different object type that happens to share part of the silhouette

## Escalation

If the object remains ambiguous after several iterations:

- request more reference information
- add more diagnostic views
- switch modeling strategy if the current one is too rigid
- add a secondary viewer only if direct renders are insufficient
