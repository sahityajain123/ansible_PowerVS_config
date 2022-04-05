# POWER LPAR Configuration for SAP HANA and SAP NetWeaver using Ansible


## Introduction

This ansible collection simplifies PowerVS LPAR configuration for installing SAP HANA and SAP Netweaver on SLES and RHEL environments. It doesn't Install SAP HANA or NETWEAVER applications but, prepares the OS with correct configurations for HANA/Netweaver Installations for best performance. They can be executed on same LPAR or different LPARS. 
This automation has 3 modules, which are independent of each other and can be run individually.

1)	**Preparing Operating System for SAP installations.**
2)	**Creating Filesystems for SAP installations.**
3)	**Configuring SWAP spaces.**

### Ansible Roles Summary

<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Input Variable Name</th>
			<th>Variable description</th>
			<th>Variable Values</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan=2><b>prepare_sles_sap</b><br /></td>
            <td rowspan=1><b>1. SAP_SOLUTION</b></td>
            <td rowspan=1>Saptune is executed based on this value</td>
            <td rowspan=1><b>HANA or NETWEAVER</b></td>
        </tr>
        <tr>
            <td><b>2. host_ip</b></td>
            <td>Management private IPv4 address</td>
            <td>e.g.: 192.168.1.1</td>
        </tr>
         <tr>
            <td rowspan=3><b>prepare_rhel_sap</b><br /></td>
            <td rowspan=1><b>1. SAP_SOLUTION</b></td>
            <td rowspan=1>Saptune is executed based on this value</td>
            <td rowspan=1><b>HANA or NETWEAVER</b></td>
        </tr>
        <tr>
            <td><b>2. host_ip</b></td>
            <td>Management IPv4 address</td>
            <td>eg: 192.168.1.1</td>
        </tr>
		 <tr>
            <td><b>3. rhel_subscription : { <br />"username":"",<br />"password":"" ,<br />"release":""<br />}</td>
            <td>RHEL subscription information. It is a dictionary</td>
            <td>e.g.: { <br />"username":"",<br />"password":"" ,<br />"release":""<br />}</td>
        </tr>
		<tr>
         <td rowspan=3><b>fs_creation</b><br /></td>
            <td rowspan=1><b>1.a. disks_congiguration: { "counts": [ ], names": [ ],"paths": [ ],"wwns": [ ] }<br />1.b. disks_configuration: [ { "name": "", "path":"", "wwns": }...]</b></td>
            <td>Disks configuration value to create and mount filesystems. Supports 2 data structures. First data structure is a single dictionary. Second data structure is a list of dictionaries.</td>
            <td rowspan=1>see <b>example a</b> and <b>example b</b> below</td>
        </tr>
        <tr>
            <td><b>2. terraform_wrapper</b></td>
            <td>This flag is bool. If input is of data structure type a then this flag has to be set True. </td>
            <td><b>eg: True or False. Default is False</b></td>
        </tr>
		<tr>
            <td><b>3. stripe_size</b></td>
            <td>This is optional, stripe size for disks  </td>
            <td><b>Default is 64K</b></td>
        </tr>
		<tr>
            <td><b>swap_creation</b></td>
            <td><b>swap_disk_wwn</b></td>
            <td>wwn id of swap disk</td>
		    <td><b>wwn value</b></td>
        </tr>
    </tbody>
</table>

## Roles Description
### 1. Preparing Operating System for SAP installations

This role is different for **SLES and 
** and hence should be selected as per operating system in use.
 
#### 1.1 prepare_sles_sap: 

This role performs the following tasks:
- Enables **multipathd** daemon
- Enables **NFS** Service
- Enables **rpcbind** daemon
- Sets **MTU** value to **9000 for SAP network interfaces**
- **TSO** is enabled for SAP network interfaces
- **SAPTUNE SOLUTION** for **HANA or NETWEAVER or both** is applied based on parameter passed.

All these setting remain persistent across reboot.

#### 1.2 prepare_rhel_sap:

This role performs the following tasks:
- Enables **multipathd** daemon
- Enables **NFS** Service
- Enables **rpcbind** daemon
- Sets **MTU** value to **9000 for SAP network interfaces**
- **TSO** is enabled for SAP network interfaces
- **Activates RHEL subscription**

**Pre installed SAP roles part of OS (1.sap-preconfigure 2.sap-hana-preconfigure 3.sap-netweaver-preconfigure)** are also executed for configuring the LPAR for HANA or NETWEAVER installation.
Link : 

All these setting remain persistent across reboot.

### 2. Creating Filesystems for SAP installations

This role is same for both SLES and RHEL.

#### 2.1 fs_creation:

This role performs the following tasks:
- **Creates filesystems** with user defined **stripe size** using ansible **builtin** modules **pvcreate, vgcreate, lvcreate and mkfs**
- **Mounts** the filesystems on provided **mount points**
- **Adds an entry to /etc/fstab** for **automount** on reboot.
- **Optional** :Converts the input data structure from **terraform to a general data structure** (Terraform output support)

A separate **task** called **terraform_wrapper.yml** is used to handle the variable values passed **from terraform output** to execute this role, via **terraform**.

The input variable **disks_configuration** for this role supports 2 data structures. When using Terraform data structure, one more variable called **terraform_wrapper** needs to be set as **True**.Only then terraform_wrapper.yml will convert the terraform data structure in **example a** to normal data structure in **example b** below. 

**example a**.**Terraform** Data Structure for **disks_configuration** variable value example:
```
disks_configuration: 
{
counts: [2,2,1], 
names: [data,log,shared], 
paths: [/hana/data,/hana/log,/hana/shared], 
wwns: [600507681082018bc8000000000057e4,600507681082018bc8000000000057e8,600507681082018bc8000000000057e5,600507681082018bc8000000000057e6,600507681082018bc8000000000057e7]}
}
```

**example b**. Data structure for **disks_configuration** variable value example:
```
disks_configuration: [
{
name: data, 
path: /hana/data, 
wwns: 600507681082018bc8000000000057e4,600507681082018bc8000000000057e8
},
{
name: log, 
path: /hana/log, 
wwns: 600507681082018bc8000000000057d9,600507681082018bc8000000000057ed7
},
{
name: shared, 
path: /hana/shared, 
wwns: 600507681082018bc8000000000057f1
}
.
.
.
]
```


### 3. Configuring SWAP spaces

This role Configures Swap space on LPAR. 
#### 3.1 swap_creation

This role performs the following tasks:
- Removes previous swap device configured
- Creates a new swap device on disk provided with swap_disk_wwn variable. 


## Execution examples

1. To run only **prepare_sles_sap** role, 

```
ansible-playbook playbook_sles.yml -e '{SAP_SOLUTION: "HANA", host_ip: "192.168.1.1" }'
```

2. To run only **prepare_rhel_sap** role, 

```
ansible-playbook playbook_rhel.yml -e '{SAP_SOLUTION: "NETWEAVER", rhel_subscription: { username: "XYZ",password: "ABC", release: "8.2"}, host_ip: "192.168.1.1"}}'
```

3. To run Only **fs_creation** role to create filesystems using **data structure example a** above for disks_configuration:

```
ansible-playbook playbook_sles.yml -e '{terraform_wrapper: True, disks_configuration: {counts:[8,8,1,1], names:[data,log,shared,usrsap], paths:[/hana/data,/hana/log,/hana/shared,/usr/sap], wwns:[6005076810810261F800000000004094,6005076810810261F800000000004096,6005076810810261F80000000000409D,6005076810810261F8000000000040A3,6005076810810261F80000000000409A,6005076810810261F8000000000040A0,6005076810810261F8000000000040A4,6005076810810261F800000000004097,6005076810810261F800000000004098,6005076810810261F80000000000409E,6005076810810261F80000000000409B,6005076810810261F80000000000409F,6005076810810261F8000000000040A2,6005076810810261F8000000000040A1,6005076810810261F800000000004095,6005076810810261F800000000004093,6005076810810261F80000000000409C,6005076810810261F800000000004099]}
```

4. To run Only **fs_creation** role to create filesystems using **data structure example b** above for disks_configuration:

```
ansible-playbook playbook_sles.yml -e '{disks_configuration: [{ name: log, path: /hana/log, wwns: 6005076810810261F800000000004098,6005076810810261F80000000000409E,6005076810810261F80000000000409B,6005076810810261F80000000000409F,6005076810810261F8000000000040A2,6005076810810261F8000000000040A1,6005076810810261F800000000004095,6005076810810261F800000000004093},{ name: shared, path: /hana/shared, wwns: 6005076810810261F80000000000409C},{ name: usrsap, path: /usr/sap, wwns: 6005076810810261F800000000004099}]}'
```

5. To run only **swap_creation** role:
```
ansible-playbook playbook_sles.yml -e '{swap_disk_wwn: 6005076810810261F80000000000409H}'
```

6. To run all roles for **RHEL( prepare_rhel_sap, fs_creation and swap_creation)** using **data structure example a** above for disks_configuration:

```
ansible-playbook playbook_rhel.yml -e '{SAP_SOLUTION: "NETWEAVER", rhel_subscription: { username: "XYZ",password: "ABC", release: "8.2"}, host_ip: "192.168.1.1", terraform_wrapper: True, disks_configuration: {counts:[8,8,1,1], names:[data,log,shared,usrsap], paths:[/hana/data,/hana/log,/hana/shared,/usr/sap], wwns:[6005076810810261F800000000004094,6005076810810261F800000000004096,6005076810810261F80000000000409D,6005076810810261F8000000000040A3,6005076810810261F80000000000409A,6005076810810261F8000000000040A0,6005076810810261F8000000000040A4,6005076810810261F800000000004097,6005076810810261F800000000004098,6005076810810261F80000000000409E,6005076810810261F80000000000409B,6005076810810261F80000000000409F,6005076810810261F8000000000040A2,6005076810810261F8000000000040A1,6005076810810261F800000000004095,6005076810810261F800000000004093,6005076810810261F80000000000409C,6005076810810261F800000000004099], swap_disk_wwn: 6005076810810261F80000000000409H}'
```

7. To run all roles for **SLES( prepare_sles_sap, fs_creation and swap_creation)** using **data structure example b** above for disks_configuration:

```
ansible-playbook playbook_sles.yml -e '{ SAP_SOLUTION: "NETWEAVER", host_ip: "192.168.1.1", disks_configuration: [{ name: log, path: /hana/log, wwns: 6005076810810261F800000000004098,6005076810810261F80000000000409E,6005076810810261F80000000000409B,6005076810810261F80000000000409F,6005076810810261F8000000000040A2,6005076810810261F8000000000040A1,6005076810810261F800000000004095,6005076810810261F800000000004093},{ name: shared, path: /hana/shared, wwns: 6005076810810261F80000000000409C},{ name: usrsap, path: /usr/sap, wwns: 6005076810810261F800000000004099}], swap_disk_wwn: 6005076810810261F80000000000409H }'
```

Similarly, only RHEL modules can be executed by changing playbook name to playbook_rhel.yml, which is part of this collection.

<b>Requirements, Dependencies and Testing</b>


### Operating System requirements

Designed for Linux operating systems, e.g. RHEL and SLES.

This role has not been tested and amended for SAP NetWeaver Application Server instantiations on IBM AIX or Windows Server.

Assumptions for executing this role include:
- Registered OS License and OS Package repositories are available (from the relevant content delivery network of the OS vendor)

### Python requirements

Python 3 from the execution/controller host.

### Testing on execution/controller host

**Tests with Ansible Core release versions:**
- Ansible Core 2.9.19 community edition
- 

**Tests with Python release versions:**
- Python 3.6.8


### Testing on target/remote host

**Tests with Operating System release versions:**
- RHEL 8.2 for SAP
- SLES 15 for SAP
- SLES 12 for SAP


## License

- [Apache 2.0](./LICENSE)

## Contributors

Contributors to the Ansible Roles within this Ansible Collection, are shown within [/docs/contributors](./docs/CONTRIBUTORS.md).
