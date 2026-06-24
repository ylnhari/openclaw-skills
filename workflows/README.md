# Workflows

This folder holds **reusable GitHub Actions workflows** that can be invoked
by individual skills via `workflow_call`.

The actual workflow files for each skill live under
`skills/<skill-slug>/.github/workflows/` (the standard GitHub location
where Actions picks them up automatically).

## Why split it this way?

Putting `skill-publish.yml` in each skill folder keeps each skill
self-contained for anyone copying just that one skill out of the umbrella
repo. The umbrella-level `workflows/` folder holds shared, reusable
workflow definitions that skills can reference via `uses:`.
