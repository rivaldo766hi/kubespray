
Fork the [Kubespray repo](https://github.com/kubernetes-sigs/kubespray)

After that clone the forked repo

```
git clone <YOUR-FORKED-REPO>
cd kubespray
```

Install ansible and other requirements

```
sudo pip install -r requirements.txt
```

You can check `Vagrantfile`. The `Vagrantfile` is the config for create virtual machine in Virtualbox. Now edit the Vagranfile so the file will looks like this.(just delete the default code and crate the new one).

```
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
```

Deploy using vagrant command and it will create the nodes on the virtualbox

```
vagrant up
```

Wait until vagrant finish provision the nodes. It will takes around 15 minutes. After that you can check on your virtualbox or run `vagrant status` command. The first on virtual machine is the base for the other virtual machine. That's how vagrant work.

![image-of-vagrant-status](images/vagrant-status.png)

![image-of-virtual-machine-lists](images/virtualbox-vm.png)

And add thess lines to `etc/hosts`

```
k8s-master-1 192.168.33.101
k8s-master-2 192.168.33.102
k8s-worker-1 192.168.33.201
k8s-worker-2 192.168.33.202
```
after that copy your ssh-key to all nodes, if you dont have key, generate it first
```
ssh-copy-id <node-ip>
```

Then edit `inventory/mycluster/inventory.ini` just likes below

```
[all]
 node1 ansible_host=192.168.33.101   ip=192.168.33.101 etcd_member_name=etcd1
 node2 ansible_host=192.168.33.102   ip=192.168.33.102 etcd_member_name=etcd2
 node3 ansible_host=192.168.33.201   ip=192.168.33.201 
 node4 ansible_host=192.168.33.202   ip=192.168.33.202
[kube_control_plane]
 node1
 node2

[etcd]
 node1

[kube_node]
 node3
 node4
[k8s_cluster:children]
kube_control_plane
kube_node
```

then run the playbook to install the cluster

```
ansible-playbook -i inventory/mycluster/inventory.ini --become --become-user=root cluster.yml
```

Wait until the installation finish. It will takes time arount 15 minutes. 

After installation is finish, let's try to access and getting nodes inforamtion using kubectl. First ensure you already install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

![image-of-kubectl-version](images/kubctl-version.png)

Then follow these command so we can access the nodes using kubectl

```
mkdir ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g ) ~/.kube/config
```

Now we have done to set up kubernetes cluster and let's verify with run this command

```
kubectl get nodes
```

![image-of-kubectl-get-nodes](images/kubectl-get-nodes.png)
