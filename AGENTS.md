# AGENTS.md — layer-workspace-mount

Standalone candy repo for the `workspace-mount` layer — the guest-side virtiofs
`/workspace` mount unit for VMs. The candy lives in `charly.yml` at the repo
root: the `mkdir:`/`write:`/`systemctl` plan steps, the `check:` assertions, and
the embedded `skill:` entity projected into the marketplace corpus as
`/charly-distros:workspace-mount`.

Canonical files:

- `charly.yml` — the `workspace-mount:` candy entity and the
  `workspace-mount-skill:` skill entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-distros:workspace-mount` — the owning skill. The mount-tag contract,
  the `.mount` unit, and the skip-aware check. Load before editing or
  troubleshooting the layer.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:` and `agent-check:`, per-distro `distro:`
  arms, package/repo sections, service declarations). Load before editing any
  entity field or plan step.

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships only `.github/workflows/tag-on-merge.yml`.
- The candy's `plan:` `check:` steps are the functional evidence. The unit
  content check (`Type=virtiofs`) is deterministic; the read-write probe is
  skip-aware (N/A when no share is attached) and must stay that way.
- The `.mount` unit deliberately has no `After=local-fs.target` — adding it
  creates an ordering cycle that systemd breaks non-deterministically. Keep it
  out.

## Modify this repo

- Edit the `workspace-mount:` candy entity AND the `workspace-mount-skill:` skill
  entity in `charly.yml` together. The skill is the projected usage source, so a
  behaviour change not mirrored in the skill leaves the corpus stale.
- The mount tag `workspace` is a cross-repo contract with the `kind: vm` entity's
  `target:`; changing it breaks every consumer.
- New behaviour claims belong in the `plan:` as an observable `check:` step, and
  in the skill body.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
