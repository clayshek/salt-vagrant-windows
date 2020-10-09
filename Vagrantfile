# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  
  # Disale default folder sync
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Windows specific settings
  # Max time to wait for guest shutdown
  config.windows.halt_timeout = 25
  config.winrm.username = "vagrant"
  config.winrm.password = "vagrant"
  config.winrm.port = 25985

  # Salt Master
  config.vm.define "saltmaster", primary: true do |saltmaster|
    saltmaster.vm.box = "bento/ubuntu-18.04"
    saltmaster.vm.host_name = "saltmaster"
    saltmaster.vm.provision "file", source: "saltstack/salt/", destination: "/tmp/salt"
    saltmaster.vm.provision "shell", inline: "mv /tmp/salt /srv/salt"
    saltmaster.vm.provision "file", source: "saltstack/pillar/", destination: "/tmp/pillar"
    saltmaster.vm.provision "shell", inline: "mv /tmp/pillar /srv/pillar"
    saltmaster.vm.provider "hyperv" do |hv|
        hv.memory = "1024"
        hv.vmname = "saltmaster"
    end
    saltmaster.vm.provision "shell", inline: "apt install -y avahi-daemon"

    saltmaster.vm.provision :salt do |salt|
      salt.master_config = "saltstack/etc/master"
      salt.master_key = "saltstack/keys/master_minion.pem"
      salt.master_pub = "saltstack/keys/master_minion.pub"
      salt.minion_key = "saltstack/keys/master_minion.pem"
      salt.minion_pub = "saltstack/keys/master_minion.pub"
      salt.seed_master = {
                          "winminion" => "saltstack/keys/minion1.pub",
                          "linuxminion" => "saltstack/keys/minion2.pub"
                         }
      salt.install_type = "stable"
      salt.install_master = true
      salt.no_minion = true
      salt.verbose = true
      salt.colorize = true
      salt.bootstrap_options = "-P -c /tmp -x python3"
    end

  end

  # Windows Minion
  config.vm.define "winminion" do |winminion|
    winminion.vm.box = "StefanScherer/windows_2019"
    winminion.vm.guest = :windows
    winminion.vm.host_name = "winminion"
    winminion.vm.network :forwarded_port, guest: 3389, host: 23389
    winminion.vm.network :forwarded_port, guest: 5985, host: 25985
    winminion.vm.provider "hyperv" do |hv|
        hv.memory = "1024"
        hv.vmname = "winminion"
    end   

    # Install Chocolatey & Bounjour mDNS
    winminion.vm.provision "shell", privileged: "true", inline: "Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))"
    winminion.vm.provision "shell", privileged: "true", inline: "choco install bonjour -y"

    winminion.vm.provision :salt do |salt|
      salt.minion_config = "saltstack/etc/minion1"
      salt.minion_key = "saltstack/keys/minion1.pem"
      salt.minion_pub = "saltstack/keys/minion1.pub"
      salt.verbose = true
      salt.colorize = true
      # Need to set version to 2019.2.5 until below PR is commited to master:
      # https://github.com/saltstack/salt-bootstrap/pull/1430
      salt.bootstrap_options = "-version 2019.2.5"
    end
    # Give the service a kick to fix check-in issue
    winminion.vm.provision "shell", privileged: "true", inline: "Restart-Service -Name salt-minion"   

  end  

  # Linux Minion
  config.vm.define "linuxminion" do |linuxminion|
    linuxminion.vm.box = "bento/ubuntu-18.04" 
    linuxminion.vm.host_name = "linuxminion"
    linuxminion.vm.provision "file", source: "saltstack/salt/", destination: "/tmp/salt"
    linuxminion.vm.provision "shell", inline: "mv /tmp/salt /srv/salt"
    linuxminion.vm.provision "file", source: "saltstack/pillar/", destination: "/tmp/pillar"
    linuxminion.vm.provision "shell", inline: "mv /tmp/pillar /srv/pillar"    
    linuxminion.vm.provider "hyperv" do |hv|
        hv.memory = "1024"
        hv.vmname = "linuxminion"
    end  
    linuxminion.vm.provision "shell", inline: "apt install -y avahi-daemon"

    linuxminion.vm.provision :salt do |salt|
      salt.minion_config = "saltstack/etc/minion2"
      salt.minion_key = "saltstack/keys/minion2.pem"
      salt.minion_pub = "saltstack/keys/minion2.pub"
      salt.install_type = "stable"
      salt.verbose = true
      salt.colorize = true
      salt.bootstrap_options = "-P -c /tmp -x python3"
    end    

  end   

end  