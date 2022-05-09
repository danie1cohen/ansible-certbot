# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "centos/7"

  config.ssh.forward_agent = true

  #config.vm.network :forwarded_port, guest: 22, host: 2218, id: 'ssh'

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end

  config.vm.provider :vmware_desktop do |vmware|
    vmware.vmx["ethernet0.pcislotnumber"] = "32"
  end

  config.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__auto: true

  config.vm.provision :shell, inline: <<-SHELL
    yum install -y epel-release
    yum install -y -q \
      https://repo.ius.io/ius-release-el7.rpm \
      https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm || echo "already installed"

    echo "updating yum repositories..."
    yum update -y -q >/dev/null 1>&2

    # clear the way for mysql installation
    yum erase -y -q mariadb* || echo "mariadb not found"

    # basics
    yum install -y -q \
      vim \
      tree \
      nmap \
      zsh \
      gcc \
      python3 \
      python3-devel \
      redhat-rpm-config \
      libffi-devel \
      openssl-devel

    echo "Installing pip..."
    curl -q https://bootstrap.pypa.io/get-pip.py -o /tmp/get-pip.py
    python3 /tmp/get-pip.py
    export PATH=$PATH:/usr/local/bin

    echo "Installing docker-ce..."
    # Install Docker CE
    ## Set up the repository
    ### Install required packages.
    yum install -y yum-utils device-mapper-persistent-data lvm2

    ### Add Docker repository.
    yum-config-manager --add-repo \
      https://download.docker.com/linux/centos/docker-ce.repo

    ## Install Docker CE.
    yum update -y && yum install -y \
      containerd.io-1.2.13 \
      docker-ce-19.03.8 \
      docker-ce-cli-19.03.8

    ## Create /etc/docker directory.
    mkdir -p /etc/docker

    mkdir -p /etc/systemd/system/docker.service.d

    # Restart Docker
    systemctl daemon-reload
    systemctl restart docker

    usermod -aG docker vagrant

    pip install docker-compose

    echo "Installing ansible..."
    pip install -Uq setuptools pip
    pip install -q enum34 ipaddress six
    pip install -q ansible==2.3.2

    echo "Setting up virtualenv..."
    # set up virtual environment
    pip install pipenv

    echo "Installing profiles..."
    # login script
    echo "export PATH=\$PATH:/home/vagrant/.local/bin"
    echo "" >> /home/vagrant/.profile

    echo "cd /vagrant" >> /home/vagrant/.profile
    echo "source ~/.profile" > /home/vagrant/.zshrc

    # vm config
    echo "colo elflord" > /home/vagrant/.vimrc
    # logout script
    echo "find /home/vagrant/.vault -exec rm {} \\;" > /home/vagrant/.bash_logout
    echo "find /vagrant/playbook -name \"*.retry\" -exec rm {} \\;" >> /home/vagrant/.bash_logout

    # set shell to zsh
    chsh --shell /bin/zsh vagrant
    chown -R vagrant.vagrant /home/vagrant /vagrant

    echo "Done!"
  SHELL

end
