This repository is a personal fork maintained seperately, but pushed as a new repository to allow for issue tracking.

The original repository can be found here. Many thanks to [chriswayg](https://github.com/chriswayg/packer-proxmox-templates)


Changes:
- Bump alpine to 3.12
- Removed virtual bridge (i'm doing dhcp from my router as this is a small experiment from my homelab.)
- Changed mirrors to mirrors.nus.edu.sg
- Fixed qemu-guest-agent not working.
- Bumped to python3.


# packer-proxmox-templates
:package: Packer templates for building Proxmox KVM images from ISO

#### [Alpine](https://wiki.alpinelinux.org/wiki/Alpine_Linux:Releases)  Linux [Packer Template](https://github.com/chriswayg/packer-proxmox-templates/tree/master/alpine-3.10-x86_64-proxmox) using Packer Proxmox Builder to build a Proxmox VM image template


## Proxmox KVM image templates

- downloads the ISO and places it in Proxmox
- creates a Proxmox VM using Packer
- builds the image using preseed.cfg (Debian/Ubuntu) and Ansible
- stores it as a Proxmox Template
- see README.md for Usage details on each template

### Check Prerequisites

The build script which will run the packer template is *configured to run on the Proxmox server*. Thus the following pre-requisites should be installed on the Proxmox server:

- Ensure that [Proxmox 6](https://www.proxmox.com/en/downloads) is installed
- Set up a DHCP server on `vmbr1` (for example `isc-dhcp-server`) see section [DHCP](https://github.com/chriswayg/ansible-proxmox/blob/master/tasks/main.yml)

```
printf  "Proxmox $(pveversion)\n"
```

- Install [Packer](https://www.packer.io/downloads.html) on the Proxmox server

```
apt -y install unzip
packer_ver=1.5.5
wget https://releases.hashicorp.com/packer/${packer_ver}/packer_${packer_ver}_linux_amd64.zip
sudo unzip packer_${packer_ver}_linux_amd64.zip -d /usr/local/bin
packer --version
```

- Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) on the Proxmox server

```
apt -y install python3-pip
pip3 install ansible==2.7.10
pip3 install py-bcrypt
```

- Install [j2cli](https://github.com/kolypto/j2cli) on the Proxmox server

```
pip3 install j2cli[yaml]
```

### Download the latest release of packer-proxmox-templates on the Proxmox server

`wget https://github.com/chriswayg/packer-proxmox-templates/archive/v1.5.zip && unzip v1.5.zip && cd packer-proxmox-templates-*`

### Usage

On the Proxmox Server with Packer installed:

```
cd OSname-xy-amd64-proxmox

# for example
cd debian-10-amd64-proxmox
cd ubuntu-18.04-amd64-proxmox
cd openbsd-6-amd64-proxmox

```

- edit `build.conf`, especially the Proxmox URL & ISO download links (for the latest distro version)
- edit `playbook/server-template-vars.yml`, especially the SSH Key & regional repos

```sh
../build.sh proxmox
```

- The template can be checked in the Proxmox GUI while it is being created
- Login using the default username as set in `build.conf`

### Build Options

```sh
../build.sh (proxmox|debug [new_VM_ID])

proxmox    - Build and create a Proxmox VM template
debug      - Debug Mode: Build and create a Proxmox VM template

VM_ID     - VM ID for new VM template (or use default from build.conf)

Enter Passwwords when prompted or provide them via ENV variables:
(use a space in front of ' export' to keep passwords out of bash_history)
 export proxmox_password=MyLoginPassword
 export ssh_password=MyPasswordInVM
```

#### Build environment

The Packer templates have been tested with the following versions of Packer and Ansible. If you use different versions, some details may need to be updated.

```sh
printf  "$(lsb_release -d) $(cat /etc/debian_version)\n" && \
  printf  "Proxmox $(pveversion)\n" &&
  packer version && \
  ansible --version |  sed -n '1p' && \
  ansible --version |  sed -n '6p' && \
  j2 --version

        Description:	Debian GNU/Linux 10 (buster) 10.3
        Proxmox pve-manager/6.1-8/806edfe1 (running kernel: 5.3.18-3-pve)
        Packer v1.5.5
        ansible 2.9.7
          python version = 3.7.3 (default, Dec 20 2019, 18:57:59) [GCC 8.3.0]
        j2cli 0.3.10, Jinja2 2.11.2
```

**NOTE:** For security reasons it would be preferable to build the Proxmox template images on a local Proxmox staging server (for example in a VM) and then to transfer the Proxmox templates using migration onto the live server(s).
