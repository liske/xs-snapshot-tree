xs-snapshot-tree
================

About
-----

*Citrix XenServer* uses the *Cluster Logical Volume Manager* (CLVM) to provide
virtual disk images (VDIs) for virtual machines (VMs).

To create snapshots, *XenServer* uses the snapshot and copy-on-write feature
of CLVM. Removing snapshots requires a coalescing process to reclaim the
storage. The online coalescing process has known limitations (i.e. it
is limited by the storage repository's (SR) free disk space). Citrix provides
an offline coalescing tool, it requires to shutdown or suspend the VM
which is beeing coalesced.

The *xs-snapshot-tree* script retrieves the current state of VHDs/VDIs on CLVM
based SRs and shows there dependencies and storage usage. This might help to
deside which VMs should be offline coalesced to reclaim free storage.


Usage
------

Copy and run *xs-snapshot-tree* onto the *XenServer* host:

```console
[root@xenserver ~]# ./xs-snapshot-tree 
Scanning VDIs...
Scanning SRs...

Storage Repositories:

 0: *
 1: SR-00
 2: SR-01

Select SR to analyze [0]: 
```

The selected SR(s) we be analyzed and print the VDI tree of the SR(s).
Different chaining levels are idented by spaces.

Example:
```console
+ [e196e13b55f8-c25f-4f74-9aca-a84a54c3]
           is-a-snapshot => false
              name-label => base copy
           size-physical => 89.72GiB
            size-virtual => 270.00GiB
             snapshot-of => <not in database>
           vhd is hidden => True
  + [fd7e5b1b6209-d717-4e03-b725-5a0a01ed]
             is-a-snapshot => false
                name-label => base copy
             size-physical => 11.35GiB
              size-virtual => 270.00GiB
               snapshot-of => <not in database>
             vhd is hidden => True
    + [d2a270cf3c35-ce47-4405-9079-c0d2b233]
               is-a-snapshot => false
                  name-label => base copy
               size-physical => 34.31GiB
                size-virtual => 270.00GiB
                 snapshot-of => <not in database>
               vhd is hidden => True
      - [1e907c142661-52ed-4960-8818-64cd547a]
                 is-a-snapshot => true
                 size-physical => 8.00MiB
                  size-virtual => 270.00GiB
                   snapshot-of => 7eb627b22033-eba2-4d72-9ebe-b8c651f7
                 vhd is hidden => False
                 vm-name-label => w2008r2-before-upgrade
      - [7eb627b22033-eba2-4d72-9ebe-b8c651f7]
                 is-a-snapshot => false
                 size-physical => 270.54GiB
                  size-virtual => 270.00GiB
                   snapshot-of => <not in database>
                 vhd is hidden => False
                 vm-name-label => w2008r2.local
```

This example shows a single VM (named *w2008r2.local*) which has one valid
snapshot (named *w2008r2-before-upgrade*) and a bunch of hidden uncoalesced
old snapshots occupying 135&nbsp;GiB of disk space.
