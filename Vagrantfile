
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
  config.vm.box_check_update = false
  
  (1..2).each do |i|
    config.vm.define "k8s-master-#{i}" do |node|
      node.vm.hostname = "k8s-master-#{i}"
      node.vm.network "private_network", ip: "192.168.33.10#{i}"
      node.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
      end
    end
  end

  (1..2).each do |i|
    config.vm.define "k8s-worker-#{i}" do |node|
      node.vm.hostname = "k8s-worker-#{i}"
      node.vm.network "private_network", ip: "192.168.33.20#{i}"
      node.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
      end
    end
  end
end