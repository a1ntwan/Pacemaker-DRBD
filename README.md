This is my tribute to LPIC 304 - simple implementations of **Nginx + Wordpress + MariaDB HA Pacemaker Two-Node Cluster with DRBD**:
***
1) Vagrant + Virtualbox provider with fence_ssh or fence_vbox stonith agents;
2) Vagrant + libvirt provider with fence_virsh or external/libvirt stonith agents;
***
The playbook used to pre-configure nodes doesn't set the exact type of configuration you need, so configuring firewalls,
locales, routes, DNS and MAC is still your own responsibility!

Check logs during special pauses located in the resources task to verify resources are added correctly and don't panic 
if some resources are started in a stopped state, feel free to re-enable them.

You may need extra vagrant plugins for libvirt:
```
vagrant plugin install vagrant-libvirt
vagrant plugin install vagrant-mutate 
```
I also want to pay my respect to [this guy](https://sleeplessbeastie.eu/2021/05/10/how-to-define-multiple-disks-inside-vagrant-using-virtualbox-provider/) for helping with a Vagrant configuration
for Virtualbox. It turned out that adding extra block storage may cause so much pain using this provider.

These playbooks have been tested on **Centos7** and **Ubuntu 20.04**.
