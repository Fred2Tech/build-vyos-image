# The custom toml for build vyos image for Proxmox VE

## Description build-vyos-image

This repository contains configuration files (e.g. *build flavors* under `data/build-flavors/`) meant to be used with the official [vyos/vyos-build](https://github.com/vyos/vyos-build) project.

## What is `vyos/vyos-build` for?

`vyos/vyos-build` is VyOS’ official build system. It provides:

- The scripts and build logic needed to produce a VyOS image (ISO/rolling, etc.) from sources.
- A reproducible build environment, commonly via the `vyos/vyos-build` Docker container, so you don’t have to install all dependencies on the host.
- Configuration mechanisms (profiles/*flavors*) to customize what goes into the image (packages, build options, metadata, etc.).

In short: if you want to compile/build a VyOS image, `vyos/vyos-build` is what does the work.


## Build flawor descrition 
 
### Tree:

```text
data/
  build-flavors/
	generic.toml
	qemu.toml
```

### Example File Generic: `data/build-flavors/generic.toml`

```toml
# Generic build image 

build_type = "release"

image_format = "iso"

packages = ["qemu-guest-agent", "cloud-init"]

disk_size = 3

[[includes_chroot]]
  path = "opt/vyatta/etc/config.boot.default"
  data = '''
system {
    host-name vyos
    login {
        user vyos {
            authentication {
                encrypted-password $6$QxPS.uk6mfo$9QBSo8u1FkH16gMyAVhus6fU3LOzvLR9Z9.82m3tiHFAxTtIkhaZSWssSgzt4v4dGAL8rhVQxTg0oAG9/q11h/
                plaintext-password ""
            }
            level admin
        }
    }
    syslog {
        global {
            facility all {
                level notice
            }
            facility protocols {
                level debug
            }
        }
    }
    ntp {
        server "0.pool.ntp.org"
        server "1.pool.ntp.org"
        server "2.pool.ntp.org"
    }
    console {
        device ttyS0 {
            speed 9600
        }
    }
    config-management {
        commit-revisions 100
    }
}

interfaces {
    loopback lo {
    }
	ethernet eth0 {
		address "dhcp"
	}
}
'''

[boot_settings]
  timeout = 3
  console_type = "ttyS"
  console_num = 0
  console_speed = 9600
```

### Example QEMU File: `data/build-flavors/qemu.toml`

```toml
# QEMU build image

build_type = "release"

image_format = "qcow2"

packages = ["qemu-guest-agent", "cloud-init"]

disk_size = 3

[[includes_chroot]]
  path = "opt/vyatta/etc/config.boot.default"
  data = '''
system {
    host-name vyos
    login {
        user vyos {
            authentication {
                encrypted-password $6$QxPS.uk6mfo$9QBSo8u1FkH16gMyAVhus6fU3LOzvLR9Z9.82m3tiHFAxTtIkhaZSWssSgzt4v4dGAL8rhVQxTg0oAG9/q11h/
                plaintext-password ""
            }
            level admin
        }
    }
    syslog {
        global {
            facility all {
                level notice
            }
            facility protocols {
                level debug
            }
        }
    }
    ntp {
        server "0.pool.ntp.org"
        server "1.pool.ntp.org"
        server "2.pool.ntp.org"
    }
    console {
        device ttyS0 {
            speed 9600
        }
    }
    config-management {
        commit-revisions 100
    }
}

interfaces {
    loopback lo {
    }
	ethernet eth0 {
		address "dhcp"
	}
}
'''

[boot_settings]
  timeout = 3
  console_type = "ttyS"
  console_num = 0
  console_speed = 9600
```


## Prerequisite for build custom iso and image vyos for Proxmox VE

* Oprating System Linux Debian or Ubuntu
* Docker

## Installation Docker on Ubuntu / Debian

### 1) Install required packages

```bash
sudo apt update
sudo apt install -y git ca-certificates curl
```

### 2) Install Docker

Option A (simplest, from distro repo):

```bash
sudo apt install -y docker.io
sudo systemctl enable --now docker
```

Option B (recommended for newest Docker Engine): install Docker from the official Docker repository (see Docker docs for your exact Ubuntu/Debian release).

### 3) Allow running Docker without sudo (optional)

```bash
sudo usermod -aG docker $USER
```

Log out/in (or open a new session) so the group change applies.

### 4) Quick check

```bash
docker run --rm hello-world
```


## Example (start a build environment)

```bash
# Clone the vyos-build repository
git clone https://github.com/vyos/vyos-build
cd vyos-build

# Start a build container
docker run --rm -it --privileged -v $(pwd):/vyos -v /dev:/dev -w /vyos vyos/vyos-build:current bash

# or externa comand

# Example command to run the build via Docker
export PARAMS=""
sg docker 'docker run --rm -it -v /dev:/dev -v "$(pwd)":/vyos -w /vyos --privileged --sysctl net.ipv6.conf.lo.disable_ipv6=0 -e GOSU_UID=$(id -u) -e GOSU_GID=$(id -g) vyos/vyos-build:current bash -c "sudo make qemu -- $PARAMS"'
```

## Example (start a build in docker)

```bash
# Clone the vyos-build repository
git clone https://github.com/vyos/vyos-build
cd vyos-build

cat << EOF > data/build-flavors/qemu.toml
# QEMU build image

build_type = "release"

image_format = "qcow2"

packages = ["qemu-guest-agent", "cloud-init"]

disk_size = 3

[[includes_chroot]]
  path = "opt/vyatta/etc/config.boot.default"
  data = '''
system {
    host-name vyos
    login {
        user vyos {
            authentication {
                encrypted-password $6$QxPS.uk6mfo$9QBSo8u1FkH16gMyAVhus6fU3LOzvLR9Z9.82m3tiHFAxTtIkhaZSWssSgzt4v4dGAL8rhVQxTg0oAG9/q11h/
                plaintext-password ""
            }
            level admin
        }
    }
    syslog {
        global {
            facility all {
                level notice
            }
            facility protocols {
                level debug
            }
        }
    }
    ntp {
        server "0.pool.ntp.org"
        server "1.pool.ntp.org"
        server "2.pool.ntp.org"
    }
    console {
        device ttyS0 {
            speed 9600
        }
    }
    config-management {
        commit-revisions 100
    }
}

interfaces {
    loopback lo {
    }
	ethernet eth0 {
		address "dhcp"
	}
}
'''

[boot_settings]
  timeout = 3
  console_type = "ttyS"
  console_num = 0
  console_speed = 9600

EOF

# Example command to run the build via Docker
export PARAMS=""
sg docker 'docker run --rm -it -v /dev:/dev -v "$(pwd)":/vyos -w /vyos --privileged --sysctl net.ipv6.conf.lo.disable_ipv6=0 -e GOSU_UID=$(id -u) -e GOSU_GID=$(id -g) vyos/vyos-build:current bash -c "sudo make qemu -- $PARAMS"'
```

## Create image by iso on Proxmox VE

### 1) Upload the ISO to Proxmox

Assumptions (adjust to your setup):

- Proxmox node name: `pve`
- ISO storage: `local` (path usually `/var/lib/vz/template/iso/`)

Option A (copy ISO to the default ISO folder on the node):

```bash
# From your workstation
scp ./vyos-1.5-rolling-<date-creation>-generic-amd64.iso root@<PVE_HOST>:/var/lib/vz/template/iso/
```

Option B (upload via Proxmox API with `pvesh` on the node):

```bash
# On the Proxmox node
pvesh create /nodes/<NODE>/storage/<ISO_STORAGE>/upload \
	-content iso -filename /path/to/vyos-1.5-rolling-<date-creation>-generic-amd64.iso
```

### 2) Create a new VM (UEFI/OVMF, VirtIO, Cloud-Init, DHCP)

Note (Secure Boot): keep Secure Boot disabled for this VM. In Proxmox this means **do not** pre-enroll Secure Boot keys on the EFI disk.

Pick a VM ID and basic variables:

```bash
VMID=9000
ISO_STORAGE=local
DISK_STORAGE=local-lvm
ISO_NAME=vyos-1.5-rolling-<date-creation>-generic-amd64.iso
BRIDGE=vmbr0
```

Create the VM (UEFI + q35, virtio NIC, serial console):

```bash
qm create $VMID \
	--name vyos-uefi \
	--ostype l26 \
	--machine q35 \
	--bios ovmf \
	--memory 2048 \
	--cores 2 \
	--cpu host \
	--net0 virtio,bridge=$BRIDGE \
	--serial0 socket \
	--vga serial0
```

Add EFI disk (required for UEFI/OVMF):

```bash
qm set $VMID --efidisk0 ${DISK_STORAGE}:0,efitype=4m,pre-enrolled-keys=0
```

Add VirtIO Block disk (virtio-blk):

```bash
qm set $VMID --virtio0 ${DISK_STORAGE}:16,ssd=1,discard=on
```

Attach the ISO as CD-ROM and set boot order:

```bash
qm set $VMID --ide2 ${ISO_STORAGE}:iso/${ISO_NAME},media=cdrom
qm set $VMID --boot order=virtio0\;ide2
```

Enable Cloud-Init drive and set IPv4 DHCP:

```bash
qm set $VMID --scsi0 ${ISO_STORAGE}:cloudinit
qm set $VMID --ipconfig0 ip=dhcp
```

Enable QEMU guest agent:

```bash
qm set $VMID --agent enabled=1
```

Start the VM:

```bash
qm start $VMID
```

## Create VM by importing a QCOW2 image on Proxmox VE

Example QCOW2: `vyos-1.5-rolling-<date-creation>-generic-amd64.qcow2`

### 1) Upload the QCOW2 to the Proxmox node

```bash
# From your workstation
scp ./vyos-1.5-rolling-<date-creation>-generic-amd64.qcow2 root@<PVE_HOST>:/root/
```

### 2) Create the VM (UEFI/OVMF, VirtIO, Cloud-Init, DHCP)

```bash
VMID=9001
DISK_STORAGE=local-lvm
SNIPPET_STORAGE=local
QCOW2=/root/vyos-1.5-rolling-<date-creation>-generic-amd64.qcow2
BRIDGE=vmbr0

# Create the VM shell (no disk yet)
qm create $VMID \
	--name vyos-qcow2-uefi \
	--ostype l26 \
	--machine q35 \
	--bios ovmf \
	--memory 2048 \
	--cores 2 \
	--cpu host \
	--net0 virtio,bridge=$BRIDGE \
	--serial0 socket \
	--vga serial0

# EFI disk (required for UEFI/OVMF)
qm set $VMID --efidisk0 ${DISK_STORAGE}:0,efitype=4m,pre-enrolled-keys=0

# Import the qcow2 into Proxmox storage (creates an "unused" disk)
qm importdisk $VMID $QCOW2 $DISK_STORAGE

# Attach the imported disk as scsi
# NOTE: the exact volume name depends on your storage; list it with: qm config $VMID
qm set $VMID --scsi0 ${DISK_STORAGE}:vm-${VMID}-disk-0,ssd=1,discard=on

# Cloud-Init drive + DHCP IPv4
qm set $VMID --scsi1 ${SNIPPET_STORAGE}:cloudinit
qm set $VMID --ipconfig0 ip=dhcp

# Enable QEMU guest agent
qm set $VMID --agent enabled=1

# Boot from the imported disk
qm set $VMID --boot order=scsi0

# Start
qm start $VMID
```

If `vm-${VMID}-disk-0` is not the right volume name, run:

```bash
qm config $VMID
```

Then set `--virtio0` to the `unused0` volume shown there.

