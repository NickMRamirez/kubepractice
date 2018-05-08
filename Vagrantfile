# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  master_ip = "192.168.2.2"
  worker_nodes = 1

  vm_prereqs = %Q{
    echo "----Disabling apache----"
    sudo update-rc.d -f apache2 remove && sudo systemctl stop apache2  

    echo "----Turning off swap as required by kubelet----"
    sudo swapoff -a
  }

  install_docker = %Q{
    if [ ! $(which docker) ]; then
      echo "----Installing docker----"
      sudo apt update
      sudo apt install -y ifupdown apt-transport-https ca-certificates curl software-properties-common
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
      sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
      sudo apt update
      sudo apt install -y docker-ce
    fi
  }

  install_kubeadm = %Q{
    if [ ! $(which kubeadm) ]; then
      echo "----Updating iptables to allow forwarded packets for service nodePorts----"
      sudo iptables -P FORWARD ACCEPT

      echo "----Installing kubeadm----"
      curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
      sudo add-apt-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
      sudo apt update
      sudo apt install -y kubeadm
    fi
  }

  configure_master_node = %Q{
    if [ ! -d /home/vagrant/.kube/ ]; then
      echo "----Initializing with kubeadm----"
      sudo kubeadm init --token db1e3e.5044869ec5bc2393 --apiserver-advertise-address #{master_ip} --pod-network-cidr=10.244.0.0/16 

      sudo mkdir -p $HOME/.kube
      sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config

      sudo mkdir -p /home/vagrant/.kube
      sudo cp -f /etc/kubernetes/admin.conf /home/vagrant/.kube/config
      sudo chown vagrant:vagrant /home/vagrant/.kube/config

      kubectl create -f /vagrant/rbac.yml

      # Use kube-flannel.yml modified to work with Vagrant
      kubectl apply -f /vagrant/kube-flannel.yml

      # Copying client cert and key to vagrant shared folder
      sudo cp /etc/kubernetes/pki/ca.* /vagrant
    fi
  }

  configure_worker_node = %Q{
    echo "----Configuring worker node----"
    sudo kubeadm join --token db1e3e.5044869ec5bc2393 #{master_ip}:6443 --discovery-token-unsafe-skip-ca-verification
  }

  config.vm.define "master1" do |node|
    node.vm.box = "ubuntu/xenial64"
    node.vm.hostname = "master1"
    node.vm.network "private_network", ip: master_ip
    node.vm.provider :virtualbox do |vb|
      vb.customize ['modifyvm', :id, '--memory', '2048']
    end

    node.vm.provision :shell, inline: "sed 's/127\.0\.0\.1.*master.*/192\.168\.2\.2 master1/' -i /etc/hosts"
    node.vm.provision :shell, inline: "sed 's/127\.0\.1\.1.*master.*/192\.168\.2\.2 master1/' -i /etc/hosts"
    node.vm.provision "shell", inline: vm_prereqs
    node.vm.provision "shell", inline: install_docker
    node.vm.provision "shell", inline: install_kubeadm
    node.vm.provision "shell", inline: configure_master_node
  end

  (1..worker_nodes).each do |num|
    config.vm.define "worker#{num}" do |node|
      node.vm.box = "ubuntu/xenial64"
      node.vm.hostname = "worker#{num}"
      node.vm.network "private_network", ip: "192.168.2.#{2 + num}"
      node.vm.provider :virtualbox do |vb|
        vb.customize ['modifyvm', :id, '--memory', '2048']
      end

      node.vm.provision :shell, inline: "sed 's/127\.0\.0\.1.*worker.*/192\.168\.2\.#{2 + num} worker#{num}/' -i /etc/hosts"
      node.vm.provision :shell, inline: "sed 's/127\.0\.1\.1.*worker.*/192\.168\.2\.#{2 + num} worker#{num}/' -i /etc/hosts"
      node.vm.provision "shell", inline: vm_prereqs
      node.vm.provision "shell", inline: install_docker
      node.vm.provision "shell", inline: install_kubeadm
      node.vm.provision "shell", inline: configure_worker_node
    end

  end
end
  