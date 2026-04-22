---
theme: seriph
background: https://cover.sli.dev
title: Reproducible Linux Installations on Your Laptop Without Data Loss
info: ODP 2026 · Holger Fischer · hoonetorg
class: text-center
transition: slide-left
duration: 30min
---

# Reproducible Linux Installations
## on Your Laptop Without Data Loss

*...or having multiple Linux installs side-by-side*

<div class="abs-br m-6 text-sm opacity-50">
  Holger Fischer · hoonetorg · ODP 2026
</div>

---
transition: fade-out
---

# The Problem

Modern Linux installers are great at **first installs**.

They fall apart the moment you want to **reinstall** without destroying:

- an existing Windows installation
- other Linux systems
- your data partition

The typical outcome: *everything gets wiped. again.*

> The installer doesn't know what's already there — and doesn't ask.

---

# The Strategy: Know Your Disk State

Two modes — decided by what's already on disk:

| Situation | Layout |
|---|---|
| Empty disk / single OS planned | **single-os** |
| Existing OS(es) detected | **multi-os** |

A Python pre-script inspects the real partition table before touching anything.

**It aborts if the state doesn't match expectations.**

No guessing. No silent data loss.

---
layout: two-cols
---

# Disk Layout: Multi-OS Example

```
1  EFI          <- preserved, reused
2  MS Reserved  <- untouched
3  Windows      <- untouched
4  WinRecovery  <- untouched
------------------ startpartition
5  /boot        <- created or emptied
6  LUKS root    <- created or emptied
------------------
7  data (LUKS btrfs)  <- never touched
   subvol: holger  -> /home/holger
   subvol: images  -> /var/lib/libvirt/images
```

::right::

### Rules

- Partitions 1–4: **never touched**
- Partitions 5+6: **created on first install, emptied on reinstall**
- Partitions after 6: **never touched**
- EFI is reused → other OSes keep booting
- Distribution swap on reinstall: ✓

<br>

> Ubuntu → Fedora on same partitions.
> Windows and data: untouched.

---

# The Builders (1)

Everything runs in **Podman containers**. No special host setup beyond:
`podman` · `tar` · `rsync` - `gpg`

<br>

### `faibuilder` / `ksbuilder`
Produce a **bootable disk image** (btrfs, thumbdrive-ready) with:
- the installer (FAI or Kickstart-based)
- a pre-calculated `EXTRA` partition for additional content:
  - Ansible playbooks (encrypted)
  - Offline package repos (e.g. Fedora 43 WS — no installer ISO exists yet)
  - Whatever you need

**Installations run fully offline.** No internet required.

<br>

---

# The Builders (2)

Everything runs in **Podman containers**. No special host setup beyond:
`podman` · `tar` · `rsync` - `gpg`

### `nsbldr` (Ansible builder)
Packages Ansible content into a **GPG-symmetrically encrypted tar**.
Extracted after first boot → run via a dedicated system user `hoo`.

---

# The Full Workflow

```
 faibuilder                          nsbldr
 or ksbuilder                        (Ansible builder)
      |                                    |
      v                                    v
 Thumbdrive image                  Encrypted ansible tar
      |                                    |
      v                                    |
 Install / Reinstall                       |
 (pre-script verifies disk state)          |
      |                                    |
      v                                    v
 First boot  <--------------------------  import & run ansible
                                           |
                                           v
                                    verify/create data subvols
                                    mountpoints + permissions
                                    create local users
```

---
layout: center
class: text-center
---

# Live Demo

*First install → Reinstall → Distribution swap*

*Windows and data: untouched throughout*

<br>
<br>

<div class="text-sm opacity-60">
  Ask questions while it runs — installs take a few minutes
</div>

---
layout: center
class: text-center
---

# Takeaways & Links

Even if your setup looks nothing like mine — steal what's useful:

- LUKS + Btrfs subvolumes as a sane data/OS separation
- Pre-script state verification before any destructive action
- Offline-first installer images via Podman
- Agama support: **work in progress** 🔜

<br>

**hoonetorg** · github.com/hoonetorg

*Slides + code links after the talk*

<div class="abs-br m-6 text-sm opacity-40">
  Holger Fischer · ODP 2026
</div>
