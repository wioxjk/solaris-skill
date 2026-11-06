---
name: |
  solaris-admin
description: |
  Load this skill when the user asks about Solaris or SunOS system administration - SMF services, ZFS, zones, IPS packaging, boot and firmware (OBP/GRUB/illumos), patching, networking, DTrace/kstat diagnostics, user and filesystem management, or migration between Solaris versions. Covers Oracle Solaris 10/11 and illumos distributions (OpenIndiana, OmniOS, SmartOS), with hardware-specific guidance for the Sun Fire V880 (RSC, FC-AL disks) and Solaris 10 on SPARC.
---

# Solaris / SunOS Administration

Practical guidance for administering Oracle Solaris 10/11, legacy SunOS, and illumos-based systems.

## Version Detection

First determine what is actually running, since commands differ significantly:

- `uname -a` and `cat /etc/release` identify Solaris 10 vs 11 vs illumos.
- Solaris 10 and earlier: SysV init (`/etc/init.d`, run levels), `pkgadd`, `patchadd`, `format`, UFS as root filesystem.
- Solaris 11+: SMF, IPS (`pkg`), ZFS-only root, `dladm`/`ipadm`/`netadm`, Automated Installer.
- illumos (OpenIndiana, OmniOS, SmartOS): mostly Solaris 11-style tooling; SmartOS is zone-only with `vmadm`/`imgadm`.

## Service Management (SMF)

- `svcs -a`, `svcs -xv` (explain failures), `svcs -l <svc>` — list and inspect.
- `svcadm enable|disable|restart|refresh|clear <svc>` — `clear` resets maintenance state.
- `svccfg -s <svc>` for property edits; `svccfg import/export` manifests.
- `svcs -L <svc>` shows the log file — first stop for a broken service.
- Legacy: Solaris 10 still has `/etc/init.d` scripts and `inetd`/`inetconv`.
- Default manifests live in `/lib/svc/manifest`; site manifests in `/lib/svc/manifest/site`.

## ZFS

- Pools: `zpool status -v`, `zpool iostat -vl 5`, `zpool list`, `zpool import -a`, `zpool scrub` (check `zpool status` for the last scrub date).
- Datasets: `zfs list -o space`, `zfs get all`, `zfs snapshot -r pool/data@name`, `zfs send/recv` for replication, `zfs diff`, `zfs hold`.
- Properties that matter: `compression=on`, `atime=off` for performance, `quota`/`refquota`, `reservation`, `mountpoint`, `canmount`, `share.nfs.*`.
- Boot environments: `beadm` (Solaris 11 / illumos); Solaris 10 uses Live Upgrade (`lucreate`/`lustatus`/`luactivate`).
- Replacing a failed disk: `zpool replace pool <old> <new>`, then confirm resilvering with `zpool status`.
- Disk labels: `zpool label`; use `format -e` to switch between EFI and SMI (VTOC) labels when needed.

## Zones / Virtualization

- Solaris 10/11 zones: `zonecfg -z <zone>` (create, add net/fs/dataset), `zoneadm -z <zone> install|boot|halt|verify|clone|detach|attach`, `zlogin -C <zone>` for the console.
- Shared-IP vs exclusive-IP zones matter for network design; validate with `zonecfg -z <zone> info`.
- Solaris 11 kernel zones: `zonecfg` with `create -t SYSsolaris-kz`.
- SmartOS: `vmadm create` (JSON payload), `vmadm list`, `vmadm get <uuid>`, `imgadm import`.
- LDoms (Oracle VM Server for SPARC): `ldm ls`, `ldm bind|start`, domain roles control/service/guest.
- Pre-migration check: `zonep2vchk` before converting a physical host to a zone.

## Packaging & Patching

- Solaris 11 IPS:
  - `pkg list`, `pkg search`, `pkg info`, `pkg install`, `pkg update -v` (creates a new BE), `pkg history`.
  - Publishers: `pkg publisher`, `pkg set-publisher -p <url>`.
  - Repair: `pkg verify`, `pkg fix`; optional components via `pkg change-facet` / `change-variant`.
  - `pkg update` may activate a new boot environment — always check with `beadm list` afterward.
- Solaris 10: `pkgadd -d`, `pkgrm`, `pkginfo`, `patchadd`, `patchrm`, `showrev -p`; Recommended Patch Clusters; `smpatch analyze`.
- illumos: OpenIndiana uses IPS; OmniOS uses IPS with the core + `extra.omnios` publishers (see the OmniOS section below for details).

## Boot Process

- SPARC OBP `ok` prompt: `boot`, `boot -s` (single user), `boot -r` (reconfiguration boot after hardware changes), `boot cdrom`, `probe-scsi-all`, `devalias`, `printenv`/`setenv` (auto-boot?, boot-device); `eeprom` from the OS side.
- x86: GRUB (Solaris 11.4 uses GRUB2); `bootadm update-archive`, `bootadm list-menu`, `bootadm set-menu`.
- Run levels: `init S|0|1|2|3|5|6`; `who -r` shows the current one. SMF milestone recovery: boot with `-m milestone=none`.
- UFS rescue on Solaris 10: boot `-s`, then `fsck -y /dev/rdsk/...`.
- Reconfiguration boot alternatives: `reboot -- -r` or `touch /reconfigure` (Solaris 10), `devfsadm -v` from a running system.

## Users, Security & Permissions

- Users: `useradd -m -d /export/home/<user>`, `usermod`, `roleadd`, `passwd`; defaults in `/etc/default/*` (login, passwd, su, cron).
- RBAC: `/etc/security/` (auth_attr, exec_attr, prof_attr); assign with `usermod -P <profile>`, inspect with `profiles`. Prefer RBAC over handing out root.
- Privileges: `ppriv`, `ps -fP`; grant specific privileges instead of root where possible.
- sudo is not standard on Solaris 10; use RBAC. Solaris 11.4 ships and supports `sudo`.
- File integrity baselines (Solaris 10): `bart create -R / > /var/bart/manifest` and `bart compare` — the Solaris 10 BluePrint on automating file-integrity checks covers the policy-file workflow; see the security section of [sun-docs-library.md](sun-docs-library.md).
- ZFS ACLs: `ls -v`, `chmod A+user:joe:rwx:allow:file`, `getfacl`/`setfacl`.
- Naming: `ldapclient` for LDAP/AD integration; `svcadm restart name-service-cache` after changes.

## Networking

- Solaris 11+: `dladm` (links), `ipadm` (addresses), `netadm`/`netcfg` (NWAM profiles); `ipadm show-addr` and `dladm show-link` are primary.
- Solaris 10: `/etc/hostname.<iface>`, `ifconfig`, `ndd` for driver tuning, `/etc/nodename`, `/etc/defaultdomain`.
- DNS: `/etc/resolv.conf` + `nsswitch.conf`; restart `name-service-cache` after edits.
- Multipathing: `mpathadm`; aggregation: `dladm create-aggr`.
- Firewalls: IPFilter (`ipf`, `ipfstat`) on Solaris; `pf` on illumos/SmartOS.

## Filesystems & Storage

- UFS (Solaris 10 and earlier): `newfs`, `fsck`, `tunefs`, `format` (partition/label, fdisk on x86).
- ZFS volumes: `zfs create -V`; NFS sharing via `zfs set share.nfs=on` (11) or `/etc/dfs/dfstab` + `shareall` (10).
- SMB: Solaris 11 `sharemgr` and the SMF shares service.
- Hot-plug and device rebuild: `cfgadm`, `devfsadm -v`.
- Crash dumps: `dumpadm -d <device>`; analyze with `mdb`.

## Diagnostics & Performance

- Process tools: `prstat` (and `prstat -t`, `-a`, `-Z` per zone), `ptree`, `pargs`, `pfiles`, `pmap`, `pstack`, `pwdx`, `truss`.
- Kernel stats: `kstat`, `iostat -xzn 5`, `vmstat`, `echo "::memstat" | mdb -k`.
- DTrace: `dtrace -l` to list probes, e.g. `dtrace -n 'syscall:::entry { @[execname] = count(); }'`; sample scripts in `/usr/demo/dtrace`.
- `mdb`: `mdb /var/crash/<host>/vmcore.N unix.N`, then `::status`, `::panicinfo`, `::stack`.
- Fault management: `fmadm faulty`, `fmadm repair <uuid>`, `fmdump -e`.
- Cluster (if present): `scstat` for quorum and resource groups.

## Backup & DR

- ZFS snapshots and replication (`zfs send -R ... | ssh host zfs recv ...`) as the primary backup mechanism.
- Solaris 10: `ufsdump`/`ufsrestore`, `fssnap` for consistent UFS snapshots, `flarcreate` for flash archives.
- Installers: JumpStart (10) vs Automated Installer (11) — `installadm create-service`, `installadm create-client`.
- System identity reset: `sys-unconfig` / `sysconfig create-profile` (11).

## Sun Fire V880 (with Solaris 10 SPARC)

Hardware reference for the Sun Fire V880 (and its close sibling V890 / V880z "Serverblade"-class chassis): 2–8 UltraSPARC III CPUs (750 MHz–1.2 GHz), up to 64 GB RAM on CPU/memory boards, 12 hot-swap FC-AL internal disks (36/73/146 GB 10k), 9 hot-plug PCI slots, dual onboard Gigabit Ethernet, redundant hot-swap power supplies and fan trays, front keyswitch (Standby / Forced Off / Diag / Locked / Normal).

### V880 hardware specifics

- Internal disks are **FC-AL, not SCSI**: they enumerate through the fibre-channel loop (`fp`/`qlc` drivers) and appear as ordinary `cXtYdZ` devices in `format`. Don't expect `probe-scsi-all` to list them — use `probe-fcal-all` at the OBP `ok` prompt.
- `prtdiag -v` is the primary hardware health tool: shows CPU/MEM board layout, PCI slots, memory interleave, plus environmental status (temperatures, fan banks, power supplies).
- Hot-plug: PCI cards, disks, power supplies, fan trays are hot-swappable; PCI board config via `cfgadm -al`. ASR (Automatic System Recovery) auto-isolates failed components at POST — check `prtdiag` for deconfigured CPUs/memory.
- Onboard dual GbE uses the Cassini `ce` driver (jumbo frames and tuning via `ndd /dev/ce`); second serial port is a plain tty, the other may be the RSC/console. The CPU architecture datasheet (`sun_ultrasparc_iii.pdf`) is mirrored in the filibeto hardware docs — see [sun-docs-library.md](sun-docs-library.md).
- After adding PCI hardware: `boot -r` (or `reboot -- -r`, or `devfsadm` online) to rebuild `/devices`.

### RSC (Remote System Control) — the V880's lights-out

The V880 uses **RSC 2.x**, not ALOM/ILOM (ALOM is V490/V890-era; ILOM is later Sun servers):

- Configure from the OS: `rscadm` (SUNWrsc package) — `rscadm shownetwork`, `rscadm set ip_mode`, `rscadm usershow`, `rscadm useradd`, `rscadm resetrsc` (power-cycle the RSC card itself).
- Access remotely: telnet/ssh/GUI client to the RSC IP; escape from console back to `rsc>` with `#.`; send a break (to reach OBP) with the `break` command at the `rsc>` prompt.
- Redirect the OS console to RSC permanently: `eeprom input-device=rsc output-device=rsc` (revert with `ttya`). If a used machine boots with console on RSC and you only have the serial port, the OS-side fix requires either RSC access or pulling the RSC card.
- RSC provides power on/off, resets, console log, environmental/fault monitoring (`rscadm status`). RSC config lives on the card, so it survives OS reinstalls; RSC passwords can only be reset from the OS (`rscadm`) or by clearing the card.

### Solaris 10 on SPARC (V880) specifics

- UltraSPARC III runs the 64-bit kernel only; verify with `isainfo -kv` (expect `sparcv9 kernel 64-bit`).
- Boot: OBP `ok` prompt via `Stop-A` (or RSC `break`); disable accidental Stop-A with `kbd -a disable`. Useful OBP: `.version`, `banner`, `printenv`/`setenv`, `probe-fcal-all`, `devalias`, `nvalias`, `test /pci@...` for a single board, `printenv boot-device` and `setenv boot-device disk`.
- `eeprom` from the OS to inspect/change NVRAM settings (`eeprom boot-device`, `eeprom auto-boot?`).
- Root on UFS (possibly encapsulated by VxVM/SDS/SVM metadevices `dXX` or `md:`); check `cat /etc/vfstab` and `metastat -p` before touching disks. ZFS is available on S10 for data pools, but not the boot pool (no `zfs root` on S10).
- Disk naming: `c0t0d0s0` (controller/target(disk WWN on FC)/LUN/slice); slices 0–7, slice 2 = whole disk; label type SMI (VTOC) for boot disks. `format` -> `partition` -> `print`; `fmthard` for scripted partitioning.
- Patching: `patchadd /path/to/patchdir`, `patchrm`, inventory with `showrev -p` or `patchadd -p`; always `patchadd` with a safe run level (single-user or stop app services first) unless instructions say otherwise; keep `/var/sadm/patch` intact so `patchrm` works (never `patchadd -d` unless the patch can't be backed out).
- Live Upgrade (`lucreate`, `luupgrade -t` to patch an inactive BE, `luactivate`, `lustatus`) is the low-downtime way to patch; also `flarcreate` for images and JumpStart for install.
- Fault management exists on S10: `fmadm faulty`, `fmdump -v`. Also `picld`/`prtpicl` for platform data, `psrinfo -v` for CPU status, `memory`/ECC errors surface via `prtdiag` and `/var/adm/messages` — watch for UEs/CEs from `sckmd`/UltraSPARC III cache sections; `Locked`/disabled CPUs appear in `psrinfo` as offline.
- Explorer output (`/opt/SUNWexplo`) is the standard bundle to hand Oracle/Sun support.

### Booting (live/rescue boot) from the DVD on a V880

Use this to recover a lost root password, repair a broken `vfstab`, or fix a system that won't boot. The Solaris 10 install DVD boots a miniroot — there is no desktop "live CD", but `boot cdrom -s` gives a fully usable root shell with the system disks available.

1. Insert the Solaris 10 SPARC DVD (internal DVD-ROM is IDE/ATAPI; use the matching Solaris release for the system — a newer DVD works for rescue, but restore with the same or older version than installed).
2. Get to the OBP `ok` prompt: `init 0` from the OS, `Stop-A`/RSC `break` if hung, or power on with `auto-boot? false`.
3. Boot:
   - `boot cdrom -s` — single-user rescue shell (the standard recovery path).
   - `boot cdrom` — full installer (graphical/CDE or console install); `boot cdrom - nowin` for the text installer.
4. From the rescue shell, mount and repair the real root:
   ```sh
   fsck -y /dev/rdsk/c1t0d0s0     # check before mounting
   mount /dev/dsk/c1t0d0s0 /a
   # mount other slices under /a as needed (e.g. mount /dev/dsk/c1t0d0s5 /a/usr)
   TERM=vt100; export TERM
   vi /a/etc/shadow               # classic lost-root-password fix
   # chroot /a ... if binaries from the real root are needed
   ```
5. Exit the shell, `reboot` — remove the DVD or the system may boot back into the installer.

Notes and troubleshooting:

- No `cdrom` alias: find the device with `devalias` or `show-devs`, then boot by full path, e.g. `boot /pci@1f,700000/ide@d/disk@0,0:f`. (V880 internal FC-AL disks are `probe-fcal-all`; the IDE DVD shows in `probe-ide`.)
- "Can't open boot device" or "Bad magic number in disk label" from the DVD almost always means bad media: a DVD burned with the ISO copied onto it as a *file* instead of burned *as an image*. Re-burn with "burn image" mode.
- The `boot -F failsafe` (or `boot failsafe`) miniroot from disk is an alternative when the kernel itself is fine but the boot archive is broken — no DVD needed.
- Everything (POST, OBP, installer, rescue shell) is visible over the RSC console too; only the escape `#.` returns you to `rsc>`.
- Never interrupt probe commands mid-run; they finish with a message before returning to `ok`.

### Resetting the RSC card on a V880

From the OS (root), `rscadm` lives at the platform path:

```sh
/usr/platform/SUNW,Sun-Fire-880/rsc/rscadm <command>
```

- Soft reset (fixes hung RSC console, failed console redirection after `bootmode -u`, unresponsive firmware): `rscadm resetrsc` — power-cycles the RSC card itself; the host keeps running. Wait ~1–2 min, then reconnect.
- Reset only the RSC password: `rscadm userpassword <user>` (or `rscadm useradd` a new admin user). Requires the SUNWrsc(rsc) package.
- Reset the **whole RSC configuration** (IP, users, alerts): run `rsc-config` and step through the interactive setup — this rewrites all settings and user accounts.
- RSC firmware not responding at all: the supported fallbacks are `rscadm resetrsc`; failing that, physically reseat the RSC card (keyswitch to Forced Off, remove/reinsert the card, back to Normal). Config lives on the card, not the disk — reseating does not wipe it.
- Firmware updates: `rscadm download <file>` with the RSC 2.2.x firmware image (check `rscadm version` first).

No OS access (lost root *and* RSC passwords):

- RSC passwords cannot be reset from OBP or the RSC serial port. Practical route: connect a terminal to the V880's plain serial port (it is separate from the RSC port), boot the Solaris DVD (`boot cdrom -s`), mount the root slice, and reset from there.
- If the console is permanently redirected to RSC (`eeprom input-device=rsc`), removing the RSC card forces console back to the standard serial port — work on the OS from there, then reconfigure RSC from inside Solaris.
- A physically dead/replaced RSC card also fails back to serial console automatically.

## OmniOS (Community Edition)

OmniOS CE is the illumos distribution aimed at servers, storage and virtualization hosts. Minimal userland by design: core `pkg` repo plus the `extra.omnios` repository for everything else (kvmadm, mdata tools, ooce packages). No pkgsrc unless you create a pkgsrc-branded zone.

### Releases and upgrade

- **r151xxx numbering**: even releases are stable (early May and early November), odd releases are "bloody" (development). Stable releases get 1 year of support; every fourth stable release is LTS with 3 years (r151022, r151030, r151038, r151046, r151054...). Check the supported matrix on omnios.org/upgrade before skipping releases — you may have to upgrade through intermediates.
- Standard release-to-release upgrade:
  ```sh
  beadm create backup-$(date +%F)                       # safety BE
  pkg set-publisher -r -O https://pkg.omnios.org/r151058/core omnios
  pkg set-publisher -r -O https://pkg.omnios.org/r151058/extra extra.omnios
  # detach ipkg-branded zones first (see below), then:
  pkg update -f -r --be-name=r151058
  init 6
  zoneadm -z <zone> attach -u                          # re-attach ipkg zones
  ```
- `-r` recurses into all linked zones (lipkg/sparse); ipkg zones are independent images and must be detached (`zoneadm -z <z> detach`) before the update and re-attached with `-u` after.
- Pre-upgrade checks that bite people:
  - GRUB is unsupported from the r151054 LTS onward — migrate to the illumos `loader` before upgrading (`bootadm` on OmniOS is loader-based).
  - Remove legacy SunSSH options from `sshd_config`/`ssh_config` before upgrading or the SSH service will fail to start afterwards.
  - Old OmniTI-era OmniOS installs need the OmniOS CE CA certificate in `/etc/ssl/pkg` first.
- Weekly updates within a release are usually rebootless: plain `pkg update`; only kernel changes force a new BE.

### Zones and VMs on OmniOS

- Default brand for new zones is `lipkg` with exclusive IP; templates exist: `zonecfg -z z1` then `create -t <type>`.
- Brands: `ipkg` (independent OmniOS image), `lipkg`/`sparse` (linked to global zone), `pkgsrc` (sparse with pkgsrc preinstalled, manage with `pkgin`, everything under `/opt/local`), `lx` (Linux syscall emulation — run whole distros/containers), `illumos` (run a different illumos userland on the shared kernel), `bhyve` and `kvm` (hardware VMs managed as zones).
- bhyve/KVM zones are minimal — they share the global zone's files (no `pkg` inside) and are configured via zone attributes:
  ```sh
  dladm create-vnic -l net0 bhyve0
  zfs create -V 30G rpool/bhyve0
  zonecfg -z debian
    create -b
    set brand=bhyve
    set zonepath=/data/zone/bhyve
    set ip-type=exclusive
    add net; set physical=bhyve0; set allowed-address=10.0.0.112/24; end
    add device; set match=/dev/zvol/rdsk/rpool/bhyve0; end
    add attr; set name=bootdisk; set type=string; set value=rpool/bhyve0; end
    add attr; set name=cdrom; set type=string; set value=/rpool/iso/debian.iso; end
    add attr; set name=vnc; set type=string; set value=on; end
    exit
  zoneadm -z debian install && zoneadm -z debian boot
  zlogin -C debian            # serial console
  ```
  Post-install: `remove attr name=cdrom` or the VM boots back into the installer. VNC via the bundled socat helper.
- KVM: prefer `kvmadm` from the extra repo (`pkg install kvmadm`) over hand-rolled SMF/qemu scripts; the qemu binary is `/usr/bin/qemu-system-x86_64`. KVM and bhyve can run concurrently (r151028+); bhyve is the recommended path going forward.
- Zone-level IPFilter: rules under `<zoneroot>/etc/ipf/` cannot be overridden from inside the zone (works for all brands except kvm; bhyve supports it).

### Install and other notes

- **Kayak** is the network installer (PXE, fast, suitable for fleets); ISO install is text-based. Kayak lives in the `kayak` package with `network/pump` service for PXE.
- Boot environments are the rollback story: anything risky (upgrade, kernel change) creates a BE; `beadm list/activate/destroy`.
- Building OmniOS from source: `omni` utility from the extra repo (`developer/omni`), forks of illumos-omnios, omnios-build, pkg5, kayak on GitHub.
- Identity: `cat /etc/release` shows `OmniOS v11 r1510xx`; `uname -v` shows the omnios build string.
- **Man pages**: the full illumos man set is browsable at man.omnios.org (`https://man.omnios.org/<section>/<name>`, e.g. `man8/zfs`). Prefer it over Oracle Solaris docs — flags and defaults differ (loader vs GRUB, `beadm`, LX brands). Legacy `1m` admin pages live in section `8`. Task-to-page mapping in [omnios-manpages.md](omnios-manpages.md).

## Common Pitfalls

- Root pool disks must be replaced with `zpool replace`, not removed.
- SMF maintenance states usually mean a dependency problem: `svcs -xv` plus `svcs -d`/`svcs -D` to untangle.
- `pkg update` can activate a new BE — verify with `beadm list` and `beadm activate` before rebooting.
- `format` destroys data: confirm device identity via `cfgadm`/`zpool status` first.
- Never `kill -9` `svc.startd` or `svc.configd` — recovery requires booting to `-m milestone=none`.
- Set `auto-boot? false` in OBP before doing firmware-level work to avoid surprise boots.
- `zoneadm attach` often fails across Solaris versions; run `zonep2vchk` first.

## Reference Libraries

- **Sun BluePrints archive (filibeto.org/sun/lib/)** — ~120 official Sun engineering best-practice PDFs plus hardware datasheets, security whitepapers, and Solaris 7–10 certification courseware, snapshotted ~2007. The go-to source for Solaris 8/9/10-era deep dives: minimization/hardening, JumpStart, DiskSuite/VxVM boot disks, Sun Cluster 3.0, Containers, Live Upgrade patching, Ethernet autonegotiation, Sun Fire 15K/midrange practices. Task-to-title index and site map in [sun-docs-library.md](sun-docs-library.md).
- **OmniOS man pages (man.omnios.org)** — current command reference for illumos/OmniOS; see [omnios-manpages.md](omnios-manpages.md).
- For anything newer than ~2007 (later Solaris 10 updates, Solaris 11, illumos), prefer man pages and distribution docs over the BluePrints archive.

## Support Files

- [cheatsheet.md](cheatsheet.md) — one-line command reference organized by task.
- [omnios-manpages.md](omnios-manpages.md) — man.omnios.org reference: section map and task-to-manpage index for OmniOS.
- [sun-docs-library.md](sun-docs-library.md) — filibeto.org Sun library: site map and BluePrints organized by admin task.
