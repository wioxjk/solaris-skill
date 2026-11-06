# solaris-skill

A Solaris / SunOS system administration skill, built for use as a [Vibe](https://chat.mistral.ai) skill (any agent that loads skills from a directory with a `SKILL.md` entrypoint).

## Coverage

- Oracle Solaris 10 and 11, legacy SunOS, and illumos distributions (OpenIndiana, OmniOS, SmartOS)
- SMF services, ZFS, zones, IPS packaging, boot process (OBP / GRUB / illumos loader), patching
- Networking (dladm/ipadm and legacy ifconfig/ndd), users, RBAC, DTrace/kstat/mdb diagnostics
- Hardware-specific guidance for the **Sun Fire V880**: RSC remote management, FC-AL internal disks, DVD rescue boot
- **OmniOS Community Edition**: release/upgrade model, zone brands (lipkg, lx, bhyve, kvm, pkgsrc), Kayak installer, and a curated man.omnios.org reference

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | Main skill instructions (loaded when the skill is selected) |
| `cheatsheet.md` | One-line command reference organized by task |
| `omnios-manpages.md` | man.omnios.org section map and task-to-manpage index |

## Usage

Copy this directory into your skills registry, e.g. `skills/solaris-skill/`, so that `SKILL.md` sits at its root. The YAML frontmatter in `SKILL.md` controls when the skill is loaded.

## License

Provided as-is for system administrators. Verify commands against your system's man pages before running anything destructive.
