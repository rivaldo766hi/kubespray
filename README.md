# Deploy Kubernetes in Virtualbox using Kubespray and Vagrant(using the default Vagrantfile)

Install [vagrant](https://www.vagrantup.com/) and [Virtualbox](https://www.virtualbox.org/) first

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

You can check `Vagrantfile`. The `Vagrantfile` is the config for create virtual machine in Virtualbox. You also can edit the file and custom it for other cases like using other Linux distribution, the number of nodes, and etc. The default Vagrantfile will install 3 virtual machine (2 master node and 1 worker node) with Ubuntu 18.04. For now lets keep it simple and use the default.

Deploy using vagrant command and it will create the nodes on the virtualbox

```
vagrant up
```

Wait until vagrant finish provision the nodes. It will takes around 15 minutes. After that you can check on your virtualbox or run `vagrant status` command. The first on virtual machine is the base for the other virtual machine. That's how vagrant work.

![images-of-vagrant-status](images/vagrant-status.png)

![images-of-virtual-machine-lists](images/virtualbox-vm.png)

