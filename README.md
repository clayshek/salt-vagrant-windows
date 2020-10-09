# Salt Windows Demo

Vagrant provisioning of a Salt playground on top of Windows (Hyper-V) and with a Windows minion. Not for production purposes.

Vagrantfile will provision 3 machines:
- saltmaster (Ubuntu 18.04)
- winminion (Windows Server 2019)
- linuxminion (Ubuntu 18.04)

## Requirements:
- [Hyper-V: Installed & functional](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v), with a virtual switch that allows outbound Internet connectivity (for package downloads)
- [Vagrant installed](https://www.vagrantup.com/downloads)

## Usage:

Clone this repo. From within the repo directory, run: `vagrant up`. The only prompt should be asking which Hyper-V virtual switch to use. The Vagrant Hyper-V provider [can't figure this out yet](https://www.vagrantup.com/docs/providers/hyperv/limitations).

First run will be slower while box images are downloaded. On completion the 3 virtual machines referenced above should be running in Hyper-V, with Salt installed and configured. Linux VMs have SSH connectivity, Windows have RDP. IP addresses will be DHCP assigned. Watch "vagrant up" output for IPs, or use commands below to let Vagrant figure it out for you.

SSH to Linux VMs: `vagrant ssh saltmaster` or `vagrant ssh linuxminion`

RDP to Windows VMs: `vagrant rdp winminion`

Credentials for each VM will be `vagrant/vagrant`

The VMs also each have mDNS software (Avahi on Linux, Bonjour on Windows) installed to [help with name resolution](http://www.multicastdns.org/).

## Basic Salt Commands:

Connectivity check (from master): `sudo salt '*' test.ping`

Check version on minions (from master): `sudo salt '*' test.version`

## Cleanup:

Destroy all VMs:  `vagrant destroy --force`  

Destroy specific VMs: `vagrant destroy vmname`

## Reference Docs:

- [Vagrant Docs](https://www.vagrantup.com/docs/index)
- [Salt Docs](docs.saltstack.com/en/latest/topics/index.html)

## Credits

Project: https://github.com/UtahDave/salt-vagrant-demo for Salt conf files and pre-generated keys.