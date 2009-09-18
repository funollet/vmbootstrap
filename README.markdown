vmbootstrap                 
===========                 

Creates and installs a new instance of a virtual machine (guest) in the KVM
node that runs the script. This includes:                                  

  - Configure a *libvirt* instance.
  - Create filesystems.          
  - Debootstrap a Debian system. 
  - Make the guest bootable: install a kernel and configure Grub.
  - Configure the guest so it can be accessed by SSH.            

**vmbootstrap** configures the VM just to the point of being usable with ssh. Any
other configuration is left to deployment tools ([Capistrano][capistrano], [Fabric][fabric]) or 
configuration management tools ([Puppet][puppet], [Chef][chef], [Cfengine][cfengine]).                       

Right now only KVM is supported for virtualization and only Debian guests
can be installed.                                                        

The guest is installed with *deborphan*. This makes the configuration easier to
tweak than with a tool based on presseding *debian-installer*.

This script is largely inspired by:

  - [Debian DSA Install KVM procedure][dsa_install]
  - *newvserver* on *vserver-utils* Debian package 


Filesystem layout
-----------------

Every guest's partition has a LV on the KVM node. No partitions neither LVM is
done inside the guest. That makes easy to grow disks and resize/fsck          
filesystems from the KVM node.                                                

The only exception is for `/boot`: the disk has a partition so we can install Grub
on the MBR.                                                                     

So, if we have a VG called 'host' and the new machine is called 'guest', there will
be two LV on the host:                                                             

    /dev/host/guest-boot
    /dev/host/guest-root
                        
Guest's /etc/fstab will look like this:

    /dev/vda1   /boot   ext3   defaults   0 0
    /dev/vdb    /root   ext3   defaults   0 0


Network and default values
--------------------------

The *libvirt* instance has the network configured as 'bridge'.

Default values are infered from the KVM host. All of them can be customized in a
configuration file (`~/.vmbootstrap`).


Utilities
---------

Some steps this scripts does are useful too while doing VM maintenance, so command-line options are provided for invoking them:

  - Mount `/boot` and `/` (under `/mnt/<guest-name>/`).
  - Umount.
  - Re-install *Grub*.


Alternatives
------------

[virt-install][virt-manager] can do most of these tasks but must be preseed.

[vmbuilder][vmbuilder] will become a very nice option once it supports Debian
hosts. Does a similar subset of tasks but doesn't use *deborphan*.

Bernd Zeimetz wrote a [vmbuilder extension that supports Debian Etch][vmbuilder-bzed] 
(but not yet Lenny). David Wendt was working on [support for Debian as a GSoC-2009][vmbuilder-dwnedt].

[Cobbler][cobbler] has a lot of (useful) features but installation on Debian doesn't look straighforward.

--------
Jordi Funollet  <jordi.f@ati.es>
Fri, 18 Sep 2009 00:25:26 +0200


[fabric]: http://fabfile.org/
[capistrano]: http://www.capify.org/
[puppet]: http://reductivelabs.com/products/puppet/
[chef]: http://wiki.opscode.com/display/chef/Home
[cfengine]: http://www.cfengine.org/

[dsa_install]: http://dsa.debian.org/howto/install-kvm/
[virt-manager]: http://virt-manager.et.redhat.com/
[vmbuilder]: https://launchpad.net/vmbuilder
[vmbuilder-bzed]: https://bugs.launchpad.net/ubuntu/+source/vm-builder/+bug/235562
[vmbuilder-dwnedt]: http://wiki.debian.org/VMBuilder
[cobbler]: https://fedorahosted.org/cobbler/
