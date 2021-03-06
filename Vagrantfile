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

    echo "================================"
    echo "installing docker..."
    echo "================================"
    yum install -y yum-utils \
      device-mapper-persistent-data \
      lvm2 wget

    yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo

    yum install -y docker-ce-selinux-17.03.3.ce-1.el7 containerd.io

    mkdir -p /etc/docker
    cat <<EOF >/etc/docker/daemon.json
    {
        "insecure-registries": ["172.30.0.0/16"],
        "dns":["8.8.8.8","8.8.4.4"]
    }
EOF

    #groupadd docker
    usermod -aG docker vagrant #$USER

    systemctl enable docker.service
    systemctl start docker.service

    echo "================================"
    echo "download and install OKD ..."
    echo "================================"
    wget -q https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-server-v3.11.0-0cbc58b-linux-64bit.tar.gz
    tar xvf openshift-origin-server-v3.11.0-0cbc58b-linux-64bit.tar.gz
    mv openshift-origin-server-v3.11.0-0cbc58b-linux-64bit /opt/okd
    echo "export PATH='$PATH:/opt/okd'" >> /home/vagrant/.bashrc
    echo "==========================================================================="
    # echo 'use : "oc cluster up --public-hostname=192.168.33.10" to start the cluster"'
    echo "Starting OKD cluster ..."
    echo "==========================================================================="
    IP_ADDR=`ip addr show eth1 | grep "inet " | awk '{print $2}' | cut -f1 -d/`
    sudo --user=vagrant /opt/okd/oc cluster up --public-hostname=$IP_ADDR
  SHELL
end
