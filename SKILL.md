---
name: comsol-matlab-simulation
description: Build, run, debug, postprocess, and document COMSOL with MATLAB / LiveLink for MATLAB simulations. Use when Codex needs to script COMSOL models from MATLAB, modify .mph models or COMSOL-exported .m files, configure studies and solvers, extract results, save .mph/.mat/.csv/.png outputs, inspect MATLAB and COMSOL logs, or maintain a reusable simulation error ledger.
---

# COMSOL with MATLAB Simulation

## Script-First Execution

Use AI coding to produce reusable MATLAB / LiveLink scripts. Group coherent model operations into scripts and run them through the available MATLAB execution channel, instead of issuing a separate agent tool call for every parameter, geometry feature, or solver setting. MCP may still be the execution channel; this skill does not require a COMSOL-specific MCP server.

- Keep generated code in the user's simulation project, separate from this installed skill.
- For sweeps, implement the case loop in MATLAB and save per-case results and failure summaries. Avoid an agent round trip for each case unless inspection or a decision is needed.
- Return concise status, key outputs, and artifact paths to the agent. Keep full logs on disk and read them when diagnosing failures.
- Reuse saved scripts for reruns. If execution is unavailable, deliver the scripts and run instructions and clearly state they have not been run.
- Do not claim measured quota, token, cost, or time savings without a comparable baseline. Fewer tool calls do not necessarily mean lower account usage.

## Operating Assumption

This skill targets an existing COMSOL with MATLAB / LiveLink for MATLAB environment. If the user confirms the connection works, proceed without repeating connection troubleshooting. If readiness is unknown, inspect the available environment or perform a small connectivity check before solving; do not assume a public user's setup is already connected. Treat explicit missing MATLAB, COMSOL, LiveLink functions, license, server/port, or path failures as environment errors and record diagnosed issues in the simulation project's `docs/error_ledger.md`.

## Standard Workflow

For every simulation task:

1. Clarify the simulation goal.
2. Check for an existing `.mph` model or COMSOL-exported `.m` script.
3. Prefer loading the `.mph`; otherwise refactor the exported `.m`; otherwise create a minimal runnable model.
4. Set parameters with units.
5. Create or modify geometry.
6. Set materials.
7. Add physics.
8. Set boundary conditions and initial conditions.
9. Create mesh.
10. Create study.
11. Configure solver.
12. Run the solve.
13. Postprocess results.
14. Export figures, tables, `.mat` data, and generated `.mph` models.
15. Save run and solver logs.
16. Summarize results.
17. Write any new solved error and prevention rule to `docs/error_ledger.md`.

## Development Order

Use this order unless the user gives a stronger project-specific reason:

1. If the user provides a `.mph`, load it and preserve the original.
2. If the user provides a COMSOL GUI-exported `.m`, use it as the API reference and parameterize it.
3. If there is no base model, build the smallest runnable model first.
4. Avoid starting with a large coupled model.
5. After each major module, run a small validation.
6. Make the model run before adding sweeps, batch cases, or advanced postprocessing.

## Required Context Check

Before writing simulation code, decide whether these are clear:

- Physics field: heat transfer, structural mechanics, electromagnetics, fluids, acoustics, multiphysics, etc.
- Geometry dimension: 1D, 2D, 2D axisymmetric, or 3D.
- Geometry sizes.
- Material parameters.
- Boundary conditions.
- Initial conditions.
- Study type: stationary, time-dependent, frequency domain, eigenfrequency, etc.
- Target outputs: temperature, stress, displacement, electric field, magnetic field, velocity, pressure, frequency response, etc.
- Output formats: `.mph`, `.mat`, `.csv`, `.png`, `.txt`.
- Parameter sweep requirements.
- Existing `.mph` or exported `.m` files.

Ask the user only when missing information blocks the physics: geometry sizes, material parameters, boundary conditions, physics type, study type, target outputs, required base model, license/module confirmation, or manual COMSOL GUI action. Choose reasonable defaults for code organization, filenames, logs, and ordinary paths.

## Implementation Rules

- Put orchestration in `src/main.m`.
- Put model creation/modification in `src/build_model.m`.
- Put per-case solving and error capture in `src/run_case.m`.
- Put result extraction and export in `src/postprocess.m`.
- Put shared parameters in `configs/parameters.m` and case overrides in `configs/case_*.m`.
- Use `fullfile` for paths.
- Create output directories when missing.
- Save generated models to `models/generated/`.
- Save figures, tables, and `.mat` data under `results/`.
- Save logs under `logs/`.
- Never overwrite original `.mph` files.
- Do not invent uncertain COMSOL API calls. Prefer existing exported `.m` files, project notes, verified local scripts, or official documentation.
- Prefer named selections or explicit selections over guessed boundary/domain IDs.
- Use units for important physical parameters, for example `model.param.set('L', '10[mm]')`.

## Validation Checklist

After editing MATLAB code, run the smallest feasible validation and inspect:

1. MATLAB executed without errors.
2. COMSOL model loaded or created.
3. Geometry built.
4. Mesh generated.
5. Study ran.
6. Solver converged.
7. Postprocessing succeeded.
8. Output files exist.
9. Key results are not `NaN`, `Inf`, or physically obvious nonsense.
10. Logs were saved to `logs/run_latest.log` and, when available, `logs/solver_latest.log`.

If validation fails, read the complete error, classify it, locate the file and code area, fix it, rerun, and record the solved problem in `docs/error_ledger.md`.

## Error Classes

Use these categories in `docs/error_ledger.md`:

- Environment error: MATLAB, COMSOL, LiveLink, license, server, port, or missing `mph*` functions.
- File path error: missing `.mph`/`.m`, missing output directory, permission issue, wrong working directory.
- COMSOL API error: wrong method, wrong argument format, version mismatch, invalid feature creation.
- Tag error: missing `comp1`, `geom1`, `mesh1`, `std1`, `sol1`, dataset tag, or result feature tag.
- Selection error: invalid boundary/domain/point IDs, geometry changed after selection, stale entity numbers.
- Unit error: missing units, incompatible dimensions, mixed SI/engineering units, bad COMSOL unit expression.
- Geometry error: unbuilt geometry, failed Boolean operation, invalid parameter geometry, tag conflict.
- Mesh error: mesh generation failure, excessive density, poor element quality, bad local size.
- Solver error: nonconvergence, bad initial values, nonlinear divergence, large time steps, conflicting boundary conditions, incomplete physics.
- Postprocessing error: wrong variable, dataset, solution tag, `mphinterp` expression, export, or plotting failure.

## Documentation Habits

- All `src/`, `configs/`, `models/`, `results/`, `logs/`, and `docs/` paths refer to the user's simulation project, not the installed skill directory.
- When project documentation is absent, create concise task-specific documents as needed. An absent checklist or ledger is not a blocker. The checklist should reflect the validation steps above; ledger entries should include the error, cause, verified fix, and prevention rule.
- Read `docs/workflow_checklist.md` at the start and end of a simulation task.
- Read `docs/error_ledger.md` before fixing a repeated error.
- Update `docs/comsol_livelink_notes.md` only with project-verified API patterns and short notes, not copied manual content.
- Report final goal, changed files, run command, success/failure, outputs, key results, generated artifacts, errors, ledger entries, and next step.
