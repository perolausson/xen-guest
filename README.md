xen-guest
=========

This is a role for creating guests on a Xen hypervisor which can then immediately be used to perform Ansible actions on, in the same Ansible playbook run.

The role uses xen-tools/xen-create-image to create new guests and it is possible to specify the method used.

This means you can install debian/ubuntu or centos/fedora guests but the latter is dependent on support in xen-tools. Unfortunately the latest centos support is centos-6.

There are issues to be aware of which has not been solved, such as feature levels in filesystems you create are dependent on the feature levels in your hypervisor.

For example, creating older releases might not boot up because the filesystem created by xen-create-image is newer and has features an older Linux distribution doesn't know about.

What happens when you run through the role?

 - copy across SSH authorised keys so that we can log into the new guest without passwords (password is however displayed during the run)
 - clean up and delete any old guest with the same name if they exist and overwrite is specified
 - create the guest image
 - create the guest
 - start the guest and wait for it to come online
 - remove/add SSH fingerprints for the old/new guest
 - add the guest to the in-memory inventory
 - prepare the host for Ansible operations (install python-minimal)

TODO:

* Add permanent inventory updates, currently only in-memory updates are done after the new guest has been prepared for Ansible use.

Requirements
------------

Ansible dig (dnspython from pip and dig from your distribution; Ansible 1.9+)
A working Xen hypervisor installation with xen-tools installed
There is a DHCP server which also updates DNS with any new hosts

Role Variables
--------------

    Variable	Default		Description  
    ==============	======		========================================
    size 		1g 		the disk size the new guest should have
    memory 		512m		the amount of memory ...
    cpu		1		the number of virtual cpus ...
    lvm		rootvg		the name of the volume group on the hypervisor that filesystems should be created from
    method		debootstrap	the method used ("debootstrap" for ubuntu, "rinse" for rpm based)
    dist            xenial          Ubuntu Xenial
    overwrite	false		overwrite and replace an existing guest with the same name
    domain		.lan		the name of the domain that new guests should be a member of
  
The following should be changed if you have non-standard locations for Xen:
  
    xen_etc_dir	    /etc/xen	       	      Where guest config files are to be created
    xen_skeleton_dir  /etc/xen-tools/skel/root  The directory where skeleton files are placed for the root user

Dependencies
------------

None

Example Playbook for creating a Xenial guest and install apache on it
---------------------------------------------------------------------
    
     ---
      - hosts: forester   # change this to the host where your Xen hypervisor is running
        roles:
          - role: xen-guest
            name: zeus2
            overwrite: true
   
      - hosts: zeus2.lan
        tasks:

          - name: Ping the new guest
            ping:

          - name: Setup
            setup: filter=ansible_eth[0-2]

          - name: Install Apache2
            package:
              name: apache2
              state: latest

Abbreviated example run:

    TASK [xen-guest : debug] **************************************************************************************************************************************************************
    ok: [forester] => {
        "msg": "Guest root password is 2S4rgXtANh5bv2MUv96Cmnu"
    }
    ....

    TASK [xen-guest : Prepare host for Ansible] *******************************************************************************************************************************************
    changed: [forester -> localhost]
    
    PLAY [zeus2.lan] **********************************************************************************************************************************************************************
    
    TASK [Gathering Facts] ****************************************************************************************************************************************************************
    ok: [zeus2.lan]

    TASK [Ping the new guest] *************************************************************************************************************************************************************
    ok: [zeus2.lan]

    TASK [Setup] **************************************************************************************************************************************************************************
    ok: [zeus2.lan]

    TASK [Install Apache2] ****************************************************************************************************************************************************************
    changed: [zeus2.lan]

    PLAY RECAP ****************************************************************************************************************************************************************************
    forester                   : ok=22   changed=10   unreachable=0    failed=0   
    zeus2.lan                  : ok=4    changed=1    unreachable=0    failed=0  

License
-------

BSD

Author Information
------------------

Per Olausson. No warranties given or implied.
