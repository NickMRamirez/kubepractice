# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  
    install_docker = %Q{
      if [ ! $(which docker) ]; then
        sudo apt update
        sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
        sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
        sudo apt update
        sudo apt install -y docker-ce
      fi
    }
  
    # No package for Ubuntu Zesty yet
    install_kubeadm = %Q{
      if [ ! $(which kubeadm) ]; then
        sudo swapoff -a
        sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
        sudo apt update
        curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
        sudo add-apt-repository "deb http://apt.kubernetes.io/ kubernetes-xenial-unstable main"
        apt update
        sudo swapoff -a # necessary or starting kubelet will fail with message about needing to disable swap
        apt install -y kubeadm
        sudo apt update && sudo apt upgrade -y
        sudo iptables -P FORWARD ACCEPT
      fi
    }

    # Run this on the master node manually
    configure_kubernetes = %Q{
      sudo kubeadm init --apiserver-advertise-address 172.28.128.3 --pod-network-cidr=10.244.0.0/16
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
      kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.8.0/Documentation/kube-flannel.yml
      kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.8.0/Documentation/kube-flannel-rbac.yml
      sudo kubeadm token list
    }

    # Run on the other nodes:
    # sudo kubeadm join --token [token] 172.28.128.3:6443
  
    number_of_hosts = 2
  
    (1..number_of_hosts).each do |num|
      config.vm.define "host#{num}" do |node|
        node.vm.box = "silverhighway/zesty64"
        node.vm.hostname = "host#{num}"
        node.vm.network "private_network", type: "dhcp"
        node.vm.provider :virtualbox do |vb|
          vb.customize ['modifyvm', :id, '--memory', '2048']
        end
        node.vm.provision "shell", inline: install_docker
        node.vm.provision "shell", inline: install_kubeadm
      end
    end
  end
  