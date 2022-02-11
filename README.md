this repository is a project repository to keep ansible automation for creation of HANA and NW filesystems. This also contains prepareos_sles module..which setup Power system LPAR for HANA and NW. 

To call ansible playbook do
```
ansible_playbook playbook.yaml -e "SAP_SOLUTION=HANA disks_configuration: {'counts': [2,2,1], 'names': [data,log,shared], 'paths': [/hana/data,/hana/log,/hana/shared], 'disks_wwn': [600507681082018bc8000000000057e4,600507681082018bc8000000000057e5,600507681082018bc8000000000057e6,600507681082018bc8000000000057e7,600507681082018bc8000000000057e8]}" 
```
SAP_SOLUTION can be HANA or NETWEAVER depending on which LPAR you are running the ansible script. This varaible is used to setup saptune solution.

disks_configuration variable  should be as shown in command line, where we capture WWNs for disk, counts of disk for each filesystems. Names will be used for vg and lv creation and paths are mount points.
