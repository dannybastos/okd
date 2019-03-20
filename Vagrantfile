# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "centos/7"
  config.vm.box_check_update = false

  config.vm.network "forwarded_port", guest: 8080, host: 8080
  config.vm.network "forwarded_port", guest: 8443, host: 8443
  config.vm.network "forwarded_port", guest: 53, host: 53

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.33.10"

  # config.vm.synced_folder "../data", "/vagrant_data"

  config.vm.provider "virtualbox" do |vb|
    # vb.gui = true
    vb.memory = "4096"
    vb.cpus = 2
    vb.customize ["modifyvm", :id, "--audio", "none"]
  end

  config.vm.provision "shell", inline: <<-SHELL
    mkdir -p /etc/docker
    touch /etc/docker/daemon.json
    yum install -y curl wget
    apt-get install -y curl wget
    curl -fsSL https://get.docker.com | bash
    usermod -aG docker vagrant
    echo '{
  "insecure-registries": ["172.30.0.0/16"],
  "dns":["8.8.8.8","8.8.4.4"]
}' >> /etc/docker/daemon.json
    systemctl daemon-reload
    systemctl restart docker
    wget -q https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-server-v3.11.0-0cbc58b-linux-64bit.tar.gz
    tar xvf openshift-origin-server-v3.11.0-0cbc58b-linux-64bit.tar.gz
    mv openshift-origin-server-v3.11.0-0cbc58b-linux-64bit /opt/okd
    echo "export PATH='$PATH:/opt/okd'" >> /home/vagrant/.bashrc
    echo "==========================================================================="
    echo 'use : "oc cluster up --public-hostname=192.168.33.10" to start the cluster"'
    echo "==========================================================================="
  SHELL
end
