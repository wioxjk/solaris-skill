# Solaris / SunOS Command Cheat Sheet

## Identify the system
```sh
uname -a
cat /etc/release          # Solaris vs illumos
showrev -p                # Solaris 10 patches
pkg list entire           # Solaris 11 IPS version
```

## SMF services
```sh
svcs -a                   # all services
svcs -xv                  # explain problems
svcs -l <svc>             # details
svcs -L <svc>             # log file path
svcs -d <svc>             # dependencies
svcadm enable|disable|restart|refresh|clear <svc>
svccfg -s <svc> listprop
svccfg import <manifest.xml>
```

## ZFS pools & datasets
```sh
zpool status -v           # incl. scrub + errors
zpool iostat -vl 5
zpool list
zpool scrub <pool>
zpool replace <pool> <old> <new>
zpool import -a
zfs list -o space
zfs get all <dataset>
zfs snapshot -r rpool/data@snap1
zfs send -R rpool/data@snap1 | ssh host zfs recv pool/data
zfs set compression=on,atime=off rpool/data
```

## Boot environments
```sh
beadm list                # Solaris 11 / illumos
beadm activate <BE>
lustatus                  # Solaris 10 Live Upgrade
luactivate <BE>
```

## Zones
```sh
zonecfg -z <zone>
zoneadm -z <zone> install && zoneadm -z <zone> boot
zoneadm -z <zone> list -cv
zlogin -C <zone>
zonep2vchk -h <host>      # physical-to-zone migration check
# SmartOS
vmadm list
imgadm show <uuid>
```

## Packaging
```sh
# Solaris 11 IPS
pkg search <term>
pkg install <pkg>
pkg update -v
pkg verify -v && pkg fix
pkg publisher
pkg history -l            # what changed, when
# Solaris 10
pkgadd -d <dir> <pkg>
patchadd <patch-id>
showrev -p
```

## Boot & firmware
```sh
# SPARC OBP (at 'ok' prompt)
boot -s                   # single user
boot -r                   # reconfiguration boot
probe-scsi-all
printenv auto-boot?
setenv auto-boot? false
devalias
# From the OS
eeprom auto-boot?=true
bootadm update-archive    # x86
reboot -- -r
who -r                    # current run level
```

## Users, RBAC, privileges
```sh
useradd -m -d /export/home/alice alice
usermod -P "Primary Administrator" alice
profiles alice
ppriv -l                 # list all privileges
ppriv <pid>
# File integrity baseline (Solaris 10)
bart create -R / > /var/bart/manifest.$(date +%F)
bart compare -i read,write < old.manifest  # diff against baseline
```

## Networking
```sh
# Solaris 11+
dladm show-link
ipadm show-addr
netadm list
ipadm create-addr -T static -a 192.0.2.10/24 bge0/v4
route -p add default 192.0.2.1
# Solaris 10
ifconfig bge0 plumb
ifconfig bge0 192.0.2.10 netmask 255.255.255.0 up
ndd -set /dev/tcp tcp_conn_req_max_q 2048
# Common
svcadm restart name-service-cache
```

## Diagnostics
```sh
prstat -t
ps -ef | grep <proc>
truss -p <pid>
pfiles <pid>
ptree <pid>
iostat -xzn 5
vmstat 5
echo "::memstat" | mdb -k
dtrace -n 'syscall:::entry { @[execname] = count(); }'
fmadm faulty
```

## Crash analysis
```sh
dumpadm                   # current dump config
mdb /var/crash/<host>/vmcore.1 unix.1
  ::status
  ::panicinfo
  ::stack
```

## Sun Fire V880 hardware
```sh
prtdiag -v                # CPU/MEM boards, PCI slots, temps, fans, PSUs
cfgadm -al                # hot-plug config (PCI, CPU/MEM boards)
psrinfo -v                # CPU status (offline = ASR-deconfigured)
fmadm faulty              # fault management (Solaris 10+)
fmdump -v                 # fault log detail
# Internal disks are FC-AL (fp/qlc drivers) - show as cXtYdZ in 'format'
```

## RSC (V880 remote management)
```sh
# From the OS
rscadm shownetwork        # RSC IP config
rscadm status             # environmental + fault status
rscadm useradd admin
rscadm resetrsc           # power-cycle the RSC card
eeprom input-device=rsc output-device=rsc   # console to RSC
# From rsc> prompt
# .                       # escape console back to rsc
break                     # send break -> OBP ok prompt
poweron / poweroff
```

## DVD rescue boot (V880 / Solaris 10 SPARC)
```sh
# At the OBP ok prompt (init 0, Stop-A, or rsc> break)
boot cdrom -s            # single-user rescue shell from install DVD
boot cdrom               # full installer (nowin = text install)
boot -F failsafe         # miniroot from disk, no DVD needed
probe-ide                # find the internal (IDE) DVD if no cdrom alias
boot /pci@1f,700000/ide@d/disk@0,0:f    # boot by full path
# Rescue shell repair:
fsck -y /dev/rdsk/c1t0d0s0
mount /dev/dsk/c1t0d0s0 /a
vi /a/etc/shadow         # lost root password fix
```

## Reset the RSC card (V880)
```sh
RSCADM=/usr/platform/SUNW,Sun-Fire-880/rsc/rscadm
$RSCADM resetrsc         # soft-reset the card (host keeps running)
$RSCADM userpassword admin     # reset one user's password
$RSCADM useradd newadmin       # add a fresh admin
$RSCADM version           # firmware version (before rscadm download)
rsc-config                # full interactive reconfig (IP, users, alerts)
# RSC hung & resetrsc failed: keyswitch -> Forced Off, reseat RSC card, -> Normal
# No OS access: console on ttya (or pull RSC card), boot cdrom -s, fix from OS
```

## Solaris 10 SPARC (V880)
```sh
isainfo -kv               # sparcv9 kernel 64-bit
showrev -p                # installed patches
patchadd /path/patchdir   # install patch (keep /var/sadm/patch for patchrm)
patchrm <patch-id>
lucreate -n newBE         # Live Upgrade
luupgrade -t -n newBE /path/patchdir
luactivate newBE && init 6
kbd -a disable            # stop accidental Stop-A
eeprom | more             # NVRAM settings
metastat -p               # SDS/SVM metadevices (check vfstab first)
```

## OBP specifics (SPARC)
```sh
probe-fcal-all            # internal FC-AL disks (probe-scsi-all won't show them)
banner                    # machine type, memory, MACs, path
.version
test /pci@1f,700000       # test a single PCI board path
nvalias disk /pci@...    # persistent device alias
setenv boot-device disk net
```

## OmniOS CE
```sh
cat /etc/release                 # r1510xx; even=stable, odd=bloody
pkg publisher                    # omnios + extra.omnios
# Release-to-release upgrade
beadm create backup-$(date +%F)
pkg set-publisher -r -O https://pkg.omnios.org/r151058/core omnios
pkg set-publisher -r -O https://pkg.omnios.org/r151058/extra extra.omnios
zoneadm -z <ipkg-zone> detach     # detach ipkg zones first
pkg update -f -r --be-name=r151058
init 6
zoneadm -z <ipkg-zone> attach -u
# Weekly updates (usually no reboot)
pkg update -v
# Extra repo tools
pkg install kvmadm               # KVM helper
# Zones
zonecfg -z z1                    # then: create -t <lipkg|ipkg|sparse|pkgsrc|lx|bhyve|kvm|illumos>
# bhyve VM: vnic + zvol + zone attrs (bootdisk, cdrom, vnc)
dladm create-vnic -l net0 bhyve0
zfs create -V 30G rpool/bhyve0
zlogin -C <vm>                   # VM serial console
pkgin install <pkg>              # inside pkgsrc zones (/opt/local)
```

## Backup / recovery
```sh
# ZFS replication
zfs snapshot -r rpool/data@daily
zfs send -R rpool/data@daily | ssh backup zfs recv -F tank/data
# Solaris 10
ufsdump 0uf /dev/rmt/0 /export/home
ufsrestore rf /dev/rmt/0
fssnap -F ufs -o bs=/snapshot /export/home
flarcreate -n name /tmp/name.flar
```
