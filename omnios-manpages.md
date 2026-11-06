# OmniOS Man Page Reference (man.omnios.org)

OmniOS ships the full illumos man set, readable online at https://man.omnios.org/. Use it as the authoritative reference when this skill's cheat sheet is not enough.

## Site layout

- Index of all sections: https://man.omnios.org/
- Section index: `https://man.omnios.org/<section>` — e.g. https://man.omnios.org/man8
- Individual page: `https://man.omnios.org/<section>/<name>` — e.g. https://man.omnios.org/man8/zfs, https://man.omnios.org/man8/zoneadm
- Underbars escape in index names (`cfgadm\_fp` → `cfgadm_fp`).

On the system itself: `man <name>`, `man -s <section> <name>`, keyword search with `man -k <keyword>` (apropos). Note that legacy Solaris `1m` admin pages are section `8` on OmniOS/illumos (`man 8 zpool`, not `man 1m`).

## Section map

| Section | Contents | When you consult it |
|---|---|---|
| man1 | User commands (`mdb(1)`, `pkg(1)`, userland tools) | Everyday commands |
| man1b | BSD compatibility commands | Old BSD-style names |
| man2 | System calls | Developer work |
| man3 / man3* | Libraries: man3c (libc), man3scf (SMF C API), man3kstat, man3dlpi (datalink), man3nvpair, man3socket, man3secdb (RBAC) | Scripting/programming against subsystems |
| man4 / man4* | man4d (device drivers), man4fs (filesystems: `ufs`, `lofs`, `tmpfs`, `pcfs`, `hsfs`, ...), man4i (ioctls), man4p (protocols) | Driver/device semantics |
| man5 | File formats and configurations (`pkg(5)` IPS image/repo format) | Config file syntax |
| man7 | Standards, environments, macros (`acl(7)`, `brands(7)`, `environ(7)`, `attributes(7)`, crypto algorithms) | Conventions and shared concepts |
| man8 | Maintenance/admin commands — the admin's main section | All system administration |
| man9 / man9e/f/p/s | Kernel concepts, driver entry points, kernel functions, properties, data structures | Driver/kernel development |

## Curated admin map (section 8 unless noted)

- **Boot environments**: `beadm(8)` (create/list/mount/activate; BEs are plain ZFS datasets), `bootadm(8)` (illumos loader menu, boot archives), `installgrub(8)` (legacy only — GRUB unsupported from r151054).
- **ZFS/storage**: `zfs(8)`, `zpool(8)`, `arcstat(8)` (ARC/L2ARC sizing), `fstyp(8)`, `lofiadm(8)` (loopback file devices), `growfs(8)`, `sbdadm(8)`, `stmfadm(8)`, `isnsadm(8)`, `share_nfs(8)`, `smbadm(8)`.
- **Zones/VMs**: `zonecfg(8)`, `zoneadm(8)`, `zlogin(8)`, `txzonemgr(8)` (zone manager TUI), `bhyve(8)`, `bhyvectl(8)`, `bhhwcompat(8)` (check host bhyve compatibility).
- **SMF**: `svcadm(8)`, `svccfg(8)` (plus `svc.configd`), `inetadm(8)`, and the C API in `man3scf`; SMF method conventions in `smf_method(5)`.
- **Networking**: `dladm(8)` (links, VNICs, aggr, bridges), `ipadm(8)` (addresses), `flowadm(8)` (per-flow QoS), `netstat(8)`, `snoop(8)`, `route(8)`, `ipf(8)`/`ipnat(8)` (IPFilter), `ndd(8)`, `tcpkey(8)`/`ipsecconf(8)` (IPsec).
- **Hardware/diagnostics**: `prtdiag(8)`, `prtconf(8)`, `cfgadm(8)` (+ `cfgadm_pci`, `cfgadm_sata`, ...), `fmadm(8)`/`fmdump(8)`/`fmstat(8)` (fault management), `dmesg(8)`, `savecore(8)`, `dumpadm(8)`, `kstat(8)`, `dtrace(8)`, `intrstat(8)`, `lockstat(8)`, `mpstat(8)`, `powertop(8)`, `sasinfo(8)`, `nvmeadm(8)`, `fcinfo(8)`, `raidctl(8)`.
- **Users/security**: `roleadd(8)`, `usermod` family, `sudo(8)`, `auditconfig(8)`/`auditd(8)`/`praudit(8)`, `cryptoadm(8)`, `kadmin(8)` (Kerberos), `ldapclient(8)`, `idmap(8)`/`idmapd(8)`.
- **Packaging**: `pkg(1)` client, `pkg(5)` image/repository format, plus legacy SVR4 `pkgadd(8)`/`pkgchk(8)`/`pkgrm(8)` for old packages.
- **Kernel/DRM**: `modinfo(8)`, `add_drv(8)`/`rem_drv(8)`, `devfsadm(8)`, `dispadmin(8)` (scheduler), `rctladm(8)`, `pooladm(8)`/`poolbind(8)` (resource pools), `pbind(8)`, `psrset(8)`.
- **Crash analysis**: `mdb(1)` (the modular debugger; walk through `::status`, `::panicinfo`), `savecore(8)`.

## Cross-references to prefer

When accuracy matters, prefer the man page over third-party blogs: OmniOS/illumos flags and defaults often differ from Oracle Solaris (e.g. `beadm`, loader instead of GRUB, LX brand semantics). If a command behaves unexpectedly, check `man.omnios.org` first and the omnios.org handbooks second.
