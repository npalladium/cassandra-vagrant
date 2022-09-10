# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. 
# Please don't change it unless you know what you're doing.
VAGRANTFILE_API_VERSION = "2"

CCM_PROVISION_SCRIPT = <<EOF
#!/bin/bash

export DEBIAN_FRONTEND=noninteractive
sudo apt-get update -qq
sudo apt-get install -y -qq -o=Dpkg::Use-Pty=0 python3 python3-pip python3-dev python3-venv python3-setuptools curl wget gnupg
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 0xB1998361219BD9C9
sudo curl -O https://cdn.azul.com/zulu/bin/zulu-repo_1.0.0-3_all.deb
sudo apt-get install -y -qq ./zulu-repo_1.0.0-3_all.deb
sudo apt-get update -qq
sudo apt-get install -y -qq zulu8-jdk
export DEBIAN_FRONTEND=dialog

python3 -m pip install --quiet --upgrade pip
python3 -m pip install --quiet pyyaml six psutil ccm
export PATH="$(python3 -m site --user-base)/bin:$PATH"

# python3 -m pip install --quiet pipx
# pipx install cqlsh
EOF

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.define "cass" do |cass|
    cass.vm.box = "hashicorp/bionic64"
    cass.vm.network "forwarded_port", guest: 80, host: 8080
    cass.vm.network "forwarded_port", guest: 9042, guest_ip: "127.0.0.1", host: 9042

    cass.trigger.before :destroy do |trigger|
      trigger.run_remote = {inline: "ccm remove", privileged: false}
    end
    cass.trigger.before :suspend, :halt do |trigger|
      trigger.run_remote = {inline: "ccm flush; ccm stop", privileged: false}
    end
  end


  config.vm.provider :virtualbox do |vb|
    vb.name = "cass"
    vb.gui = false
    vb.memory = "3072"
  end

  config.vm.provision :shell do |s|
     s.name = "setup"
     s.privileged = false
     s.inline = CCM_PROVISION_SCRIPT
  end

  config.vm.provision :shell do |s|
     s.name = "start"
     s.privileged = false
     s.inline = "ccm create cass -v 4.0.0 -n 3 -s"
  end
end
