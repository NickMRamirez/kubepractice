# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

    disable_apache = %Q{
      sudo update-rc.d -f apache2 remove && sudo systemctl stop apache2
    }
  
    install_docker = %Q{
      if [ ! $(which docker) ]; then
        echo "----Installing docker----"
        sudo apt update
        sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
        sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
        sudo apt update
        sudo apt install -y docker-ce
      else
        echo "docker already installed."
      fi
    }
  
    # No package for Ubuntu Zesty yet
    install_kubeadm = %Q{
      if [ ! $(which kubeadm) ]; then
        echo "----Installing kubeadm----"
        sudo swapoff -a # kublet service complains if memory swap is enabled
        curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
        sudo add-apt-repository "deb http://apt.kubernetes.io/ kubernetes-xenial-unstable main"
        sudo apt update
        sudo apt install -y kubeadm
        sudo iptables -P FORWARD ACCEPT # allow access to nodePorts
      else
        echo "kubeadm already installed."
      fi
    }

    configure_master_node = %Q{
      if [ ! -d /home/vagrant/.kube/ ]; then
        echo "----Initializing with kubeadm----"
        sudo kubeadm init --token db1e3e.5044869ec5bc2393 --apiserver-advertise-address 172.28.128.3 --pod-network-cidr=10.244.0.0/16 

        sudo mkdir -p $HOME/.kube
        sudo cp -f -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config

        sudo mkdir -p /home/vagrant/.kube
        sudo cp -f -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
        sudo chown vagrant:vagrant /home/vagrant/.kube/config

        kubectl taint nodes --all node-role.kubernetes.io/master-
        kubectl create -f /vagrant/rbac.yml
      else
        echo "kubeadm already initialized"
      fi
    }

    install_flannel = %Q{
      kubectl apply -f https://git.io/weave-kube-1.6
    }

    configure_worker_node = %Q{
      echo "----Configuring worker node----"
      sudo kubeadm join --token db1e3e.5044869ec5bc2393 172.28.128.3:6443

      sudo mkdir -p /home/vagrant/.kube
      sudo cp -f -i /etc/kubernetes/kubelet.conf /home/vagrant/.kube/config
      sudo chown vagrant:vagrant /home/vagrant/.kube/config
    }

    # Run this on this VM:
    # sudo kubeadm token list
    config.vm.define "master1" do |node|
      node.vm.box = "silverhighway/zesty64"
      node.vm.hostname = "master1"
      node.vm.network "private_network", type: "dhcp"
      node.vm.provider :virtualbox do |vb|
        vb.customize ['modifyvm', :id, '--memory', '2048']
      end

      node.vm.provision "shell", inline: disable_apache
      node.vm.provision "shell", inline: install_docker
      node.vm.provision "shell", inline: install_kubeadm
      node.vm.provision "shell", inline: configure_master_node
      node.vm.provision "shell", inline: install_flannel
    end

    # Run this on these VMs:
    # sudo kubeadm join --token [token] 172.28.128.3:6443
    worker_nodes = 1
  
    (1..worker_nodes).each do |num|
      config.vm.define "worker#{num}" do |node|
        node.vm.box = "silverhighway/zesty64"
        node.vm.hostname = "worker#{num}"
        node.vm.network "private_network", type: "dhcp"
        node.vm.provider :virtualbox do |vb|
          vb.customize ['modifyvm', :id, '--memory', '2048']
        end

        node.vm.provision "shell", inline: disable_apache
        node.vm.provision "shell", inline: install_docker
        node.vm.provision "shell", inline: install_kubeadm
        node.vm.provision "shell", inline: configure_worker_node
      end
    end
  end
  