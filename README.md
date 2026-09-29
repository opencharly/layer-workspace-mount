# layer-workspace-mount

Mount a virtiofs `workspace` share at `/workspace` inside an OpenCharly VM guest
via a systemd `.mount` unit.

The `workspace-mount` candy writes and enables
`/etc/systemd/system/workspace.mount` so a virtiofs share tagged `workspace`
re-mounts at `/workspace` on every boot — load-bearing for an autostarting VM
that must come back with its share already mounted. It is guest-side only and
meaningful on a running VM that has the share attached; at image build where no
device exists it is a tolerated no-op.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `workspace-mount` |
| Unit | `/etc/systemd/system/workspace.mount` (`Type=virtiofs`) |
| Mount | `What=workspace` → `Where=/workspace`, `WantedBy=multi-user.target` |
| Scope | systemd-based guests (Arch/CachyOS, Fedora, bootc) |
| Service / port | none |

## How to use it

Compose the layer by pinning this repo in a VM box's `candy:` list:

```yaml
my-vm:
  candy:
    base: fedora
    candy:
      - '@github.com/opencharly/layer-workspace-mount:v2026.239.1626'
```

The mount **tag `workspace`** is the contract: the `kind: vm` entity must declare
a matching virtiofs filesystem:

```yaml
libvirt:
  devices:
    filesystems:
      - {driver: virtiofs, accessmode: passthrough, source: /home/me, target: workspace}
```

The candy's `plan:` asserts the `.mount` unit contains `Type=virtiofs`, and a
skip-aware check that `/workspace` is a writable virtiofs mount when a share is
attached (and N/A when none is).

## Layout

- `charly.yml` — the `workspace-mount:` candy entity (the `mkdir:`/`write:`/
  `systemctl` plan steps, the `check:` assertions) and the embedded
  `workspace-mount-skill:` skill entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-distros:workspace-mount`
- `/charly-internals:libvirt-renderer` — `mapFilesystem` + `ensureVirtiofsSharedMemory`
- `/charly-vm:vms-catalog` — `filesystems:` authoring on the kind:vm entity
- `/charly-vm:cachyos-bootstrap-vm` — the CachyOS VM family that consumes it
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
