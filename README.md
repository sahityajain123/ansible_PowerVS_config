<b>LPAR Configuration for SAP HANA and SAP NetWeaver using Ansible</b>


<b>Introduction</b>

The ansible automation, simplifies PowerVM LPAR configuration for installing SAP HANA and SAP Netweaver. This automation currently supports SLES and RHEL environments. At present this automation supports, configurating pre-requisite for installation of HANA and Netweaver. They can be installed on same LPARs or different LPARS. 
This automation has 3 modules, which are independent of each other, and can be run individually. For complete configuration, both roles should be run together, which is described in this documentation below.

1)	Preparing Operating System
2)	Creating Filesystems for SAP installations.
3)	Configuring SWAP spaces. 


<b>Preparing Operating System</b>

SLES: 
This module role is named as prepareos_sles. In this module, automation for discovering of disk using rescan_scsi_disk.sh is used, which will discover newly attached disks, if any  in LPAR after it has been started.  Later we enable multipathd daemon. We also enable NFS and rpcbind daemon for SAP installations. 

We also do network configuration, where we set network MTU to 9000 and make tso on, for all network devices which are not eth0, lo or device without and ipv4 IP associated to it. This is done in such a way, that after LPAR is rebooted by its user, the setting remain persistent. 

We also do saptune setting in the role, which is executed based on SAP_SOLUTION value passed when ansible playbook is executed. SAP_SOLUTION can be HANA or NETWEAVER depending on which LPAR you are running the ansible script.

RHEL:
This module role is named as prepareos_rhel.  In this module, automation for discovering of disk using rescan_scsi_disk.sh is used, which will discover newly attached disks, if any  in LPAR after it has been started.  Later we enable multipathd daemon. We also enable NFS and rpcbind daemon for SAP installations. 

We also do network configuration, where we set network MTU to 9000 and make tso on, for all network devices which are not eth0, lo or device without and ipv4 IP associated to it. This is done in such a way, that after LPAR is rebooted by its user, the setting remain persistent. 

We also run, pre installed SAP roles, as part of this automation. 


<b>Creating Filesystems for SAP installations</b>

This module is similar for both SLES and RHEL.
This module role is named as prepare_fs_sles. In this module, automation for configuring filesystems, for SAP installation is done, using multiple ansible modules for logical volume. We have a task wrapper_terraform.yml, which is used to handle the variable passed to execute this role, via terraform. It is written in such a way that role can be ran exclusively, with no need to run wrapper_terraform.yml task.
In this module, we do pvcreate, followed by vgcreate, lvcreate, and file system creation, in same order. We use dictionary list, as variable as shown below

```
disks_configuration: 
{
"counts": [2,2,1], 
"names": [data,log,shared], 
"paths": [/hana/data,/hana/log,/hana/shared], 
"disks_wwn": [600507681082018bc8000000000057e4,600507681082018bc8000000000057e8,600507681082018bc8000000000057e5,600507681082018bc8000000000057e6,600507681082018bc8000000000057e7]}
}
```
This is how we pass the variable using terraform. In similar way we do for standalone role, which will be  like.
```
file_system_list: 
{
"counts": 2, 
"names": data, 
"paths": /hana/data, 
"disks_wwn": 600507681082018bc8000000000057e4,600507681082018bc8000000000057e8
}
```
The standalone role, can run for one filesystem at a time. If you want to create multiple filesystems in one execution, you should use the variable as defined in disks_configuration variable. 

Let me explain the variable now. 

count: count of disks to be added to  volume group. This will also be used as number of stripes, in lvcreate command. By default Stripe Size is 64k, unless stripe_size variable is defined and passed as extra variable.

names: this variable is used to distinguish between data provided. It is also used to name volume group as [name]vg and logical volume as [name]lv. We also run prepare_fs_sles role based on names provided. 

paths: path where filesystem created need to be mounted. 

disks_wwn: wwn ID of disks to be used for filesystem creation. You can get wwn using multipath -ll command. 

All these variable values should be written in order, as automation works accordingly. 

We also perform cleanup as optional , and do validation of passed values, for correctness. 

<b>Executing Automation</b>

To call the complete automation, ansible playbook playbook_sles.yml file need to be used. It can be called as below
```
ansible-playbook playbook_sles.yml -e '{"SAP_SOLUTION":HANA, "swap_disk_wwn":"600507681082018bc8000000000057e1", disks_configuration: {"counts": [2,2,1], "names": [data,log,shared], "paths": [/hana/data,/hana/log,/hana/shared], "disks_wwn": [600507681082018bc8000000000057e4,600507681082018bc8000000000057e8,600507681082018bc8000000000057e5,600507681082018bc8000000000057e6,600507681082018bc8000000000057e7]}}'
```
To run automation, to only create filesystems, and not to apply prepareos_sles role, execute

```
ansible-playbook playbook_sles.yml -e '{disks_configuration: {"counts": [2,2,1], "names": [data,log,shared], "paths": [/hana/data,/hana/log,/hana/shared], "disks_wwn": [600507681082018bc8000000000057e4,600507681082018bc8000000000057e8,600507681082018bc8000000000057e5,600507681082018bc8000000000057e6,600507681082018bc8000000000057e7]}}'
```

To run automation, to create only 1 filesystem, using fs_creation_sles role. This will not run wrapper_terraform task.

```
ansible-playbook  playbook_sles.yml  -e '{"file_system_list": {"counts": "5", names": "hana", "paths": "/hana", "wwn": "600507681082018bc8000000000057e4,600507681082018bc8000000000057e8,600507681082018bc8000000000057e5,600507681082018bc8000000000057e6,600507681082018bc8000000000057e7" } }'
```

To run only swap space creation role
```
ansible-playbook playbook_sles.yml -e "swap_disk_wwn"="600507681082018bc8000000000057e1"
```

To just run, prepareos_sles role, 
```
ansible-playbook playbook_sles.yml -e "SAP_SOLUTION:HANA”
```

Similarly, you can run RHEL modules, by changing playbook name as playbook_rhel.yml.

END of Document





