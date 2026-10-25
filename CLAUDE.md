# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Chinchillin is a Unity game made by AJ and Maria, who is new to coding. The project was just created from Unity's URP template, so nothing in `Assets/` is game code yet. `Assets/TutorialInfo/` and `Assets/Readme.asset` are template leftovers that can be deleted.

**Source of truth: `docs/PLAN.md`** (not written yet). It will hold the game concept, phases and milestones. Before starting a milestone, read it. If work has no entry there, add one first; the `anthropic-skills:game-build-planner` skill builds this plan.

- Unity **6000.6.5f1** (Unity 6), Universal Render Pipeline 17.6 with `PC_RPAsset` and `Mobile_RPAsset` quality tiers in `Assets/Settings/`.
- **Input System 1.20** (`activeInputHandler: 1`, which means the new Input System only; the old `UnityEngine.Input` API won't work). Actions are in `Assets/InputSystem_Actions.inputactions`.
- Installed packages worth knowing about: Test Framework 1.8, AI Navigation 2.0, Timeline, uGUI, Visual Scripting, Unity AI Assistant/Inference, and **Pipeline** (`com.unity.pipeline`), which lets the `unity` CLI drive the open Editor.
- The only scene in the build is `Assets/Scenes/SampleScene.unity`.

## The agent workflow

The agents are in `.claude/agents/`. Each milestone goes through them in this order, and their hand-off files live in `docs/review/`:

1. **planner** runs once per phase or milestone and writes `phase-N-plan.md`. It is never part of the revision loop.
2. **coder** implements the plan. On later rounds its only task list is the last `round-N-verdict.md`.
3. **reviewer** writes `round-N-review.md`.
4. **adversary** checks the review against the code and writes `round-N-adversary.md`.
5. **gatekeeper** writes `round-N-verdict.md`, which starts with `VERDICT: SATISFIED` or `VERDICT: REVISE`. On REVISE, go back to step 2 with N+1. After round 3, a REVISE includes `ESCALATE: yes`; stop and hand the open items to AJ.
6. **tester** runs once the verdict is SATISFIED, writes tests from the milestone (not from the code), and writes `tester-report.md`.

**teacher** sits outside the loop. Use it whenever Maria (or AJ) asks how or why something works: C# syntax, OOP, Unity concepts, gameplay patterns, or a piece of the project's own code. It explains, quizzes and gives hints, and writes Maria's code only if she asks it to. Its notes go in `docs/learning/`.

The orchestrator (the main session) passes each agent the milestone, the round number and the files it needs. Agents don't commit unless AJ asked for a commit.

## Unity plugin and skills

The official Unity plugin (`unity:*` skills) is installed. Check for a matching skill before doing Unity work by hand. The ones most likely to matter here:

- `unity:unity-cli`: drive the open Editor, run tests, build, read logs.
- `unity:unity-package-management`: add or remove packages.
- `unity:ui-ugui`, `unity:ui-uitk`: UI work.
- `unity:initialize-ai-navigation`: NavMesh and agents.
- `unity:2d-pixel-perfect`, `unity:sprite-editor`, `unity:manage-sprite-atlas`, `unity:tilemap-*`: 2D work.
- `unity:urp-postprocessing`, `unity:shader-graph-create-custom-node`, `unity:validate-urp-render-graph-renderer-feature`: rendering.
- `unity:optimize-audio`, `unity:audio-setup-mixers`, `unity:optimize-text-mesh-pro`, `unity:localization`.
- `unity:generate-editor-search-query`: find assets or scene objects.

**Scenes, prefabs and other `.asset` files are YAML that Unity owns.** If the Editor is open, change them through `unity command` (see `unity:unity-cli`) rather than editing the YAML by hand. If you must hand-edit, keep `fileID`/`guid` references intact and never hand-write a `.meta` file.

## Commands

Run these from the project root (`chinchillin'/`). The folder name has an apostrophe, so quote paths in shells.

```
unity status                                   # is an Editor connected? (look for "ready")
unity command                                  # list commands the open Editor exposes
unity command eval '<C#>'                      # run C# in the open Editor
unity logs --follow --level info               # Editor log
unity test . --mode EditMode                   # EditMode tests (exit 0 pass, 8 failing tests, 6 no verdict e.g. compile error)
unity test . --mode PlayMode
unity test . --mode EditMode --filter "Chinchillin.Tests.PlayerMovementTests"   # one class or test
unity build . --target StandaloneWindows64 --output-path ./Builds/Chinchillin.exe
```

`unity test` and `unity build` launch their own batch-mode Editor, which fails while the same project is open in the Editor. Close the Editor first, or run the tests from the open Editor's Test Runner window.

If `unity status` can't connect while the Editor is open, the project probably has compile errors and the Editor is in Safe Mode. Run `unity pipeline list` to confirm, then fix the C#; don't fall back to hand-editing scenes.

## Conventions

- Game code goes under `Assets/_Project/` (Scripts, Prefabs, Scenes, Art, Audio), with an assembly definition per area. Tests go in `Assets/_Project/Tests/EditMode` and `.../PlayMode`, each with its own `.asmdef` that references the code assembly. The Test Framework only finds tests in assemblies that have asmdefs.
- Namespaces start with `Chinchillin.` and follow the folder.
- Use `[SerializeField] private` fields for Inspector wiring instead of public fields. When renaming a serialized field, add `[FormerlySerializedAs("oldName")]` so scenes and prefabs keep their values.
- Keep logic that can be tested as plain C# classes, and keep `MonoBehaviour`s thin.
- Read input through Input System actions, never `UnityEngine.Input`.
- `.gitattributes` sends binary assets (png, fbx, wav, and so on) to Git LFS and forces LF for Unity YAML/JSON. Commit every `.meta` file with its asset.
- `.gitattributes` names a `unityyamlmerge` merge driver, but git isn't configured for it yet. Scene and prefab merge conflicts won't resolve automatically, so avoid two people editing the same scene at once.
