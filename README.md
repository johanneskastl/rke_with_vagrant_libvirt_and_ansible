# rke_with_vagrant_libvirt_and_ansible

Vagrant setup for building a RKE cluster with three controlplane nodes using the vagrant libvirt provider and ansible for provisioning of the VMs

This setup creates (by default) three controlplane VMs and one worker node using rke. The numbers can be changed in the `Vagrantfile`, see below.

Default OS is openSUSE Leap 15.2, but that can be changed in the Vagrantfile. Same holds true for the sizing of the machines.

In contrast to [this Vagrant setup](https://github.com/johanneskastl/rke_with_vagrant_libvirt) this one uses Ansible, which makes it easier to change instance numbers.

## Vagrant

1. You need vagrant obviously. And libvirt. And rke. And Ansible.
2. Fetch the box, per default this is `opensuse/Leap-15.2.x86_64`, using `vagrant box add opensuse/Leap-15.2.x86_64`.
3. Make sure the machine where you want to run this on has enough RAM to allow running the three VMs. If not, adjust the sizes in the `Vagrantfile`.
4. Run `vagrant up`
5. Run `kubectl --kubeconfig kube_config_cluster.yml get nodes` and you should see your three controlplane nodes as well as the worker nodes.
6. Party!

## Changing the number of worker nodes

To change the number of worker nodes, you need to modify the `Vagrantfile` and tweak two lines:

1. Setting the number of workers (in this example to `2`):

```
  ###################################################################################
  # define number of workers
  W = 2
```

2. Adding the additional worker nodes to the `ansible_groups` line:
```
            ansible.groups = {
              "rkeservers"  => [ "rkeserver1", "rkeserver2", "rkeserver3" ],
              "rkeworkers"  => [ "rkeworker1", "rkeworker2" ]
            }
```

Of course you need to make sure your host system has enough resources...

## Changing the number of controlplane nodes

To change the number of controlplane nodes, you can modify the `Vagrantfile` similar to adjusting the worker node number described above.

1. Setting the number of servers (in this example to `1`):

```
  ###################################################################################
  # define number of controlplane nodes
  M = 5
```

2. Adding the additional controlplane nodes to the `ansible_groups` line:
```
            ansible.groups = {
              "rkeservers"  => [ "rkeserver1", "rkeserver2", "rkeserver3", "rkeserver4", "rkeserver5" ],
              "rkeworkers"  => [ "rkeworker1" ]
            }
```

Of course you need to make sure your host system has enough resources...

## Adjusting size in the Vagrantfile

In case you want or need to change the VM sizing, change the following lines in the `Vagrantfile`:
```
        lv.memory = 2048
        lv.cpus = 2
```

## Creating your own ssh keys

This setup uses a ssh private key that is delivered with this repository. This is a NO-GO for productive use cases, and this key should be considered insecure and for testing purposes only.

In case you want to create your own key, run the following commands:

```bash
rm ssh_key_rke ssh_key_rke.pub
ssh-keygen -t ecdsa -f ssh_key_rke -C "rke_with_vagrant_libvirt" -P ""
```

Please make sure you do not accidentally commit your new ssh key files somehow (they are not excluded via `.gitignore`)...

## Cleaning up

You can remove your vagrant VMs using `vagrant destroy`, but you need to manually remove the following files before starting again:
```bash
cluster.yml
cluster.rkestate
kube_config_cluster.yml
```

Otherwise rke will try to restore the state saved in the `cluster.rkestate` file. And will fail horribly...

In case you are lazy you can just use Ansible to clean up for you. Please not that this WILL NOT ASK FOR CONFIRMATION!

This will delete all rke-related files as well as the SSH keys!

```
ansible-playbook ansible/playbook-DELETE_everything_to_start_from_scratch.yml
```
