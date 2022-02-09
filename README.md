this repository is a project repository to keep ansible automation for rise.

make sure the variable file is placed in /root/variable_file.txt on both NW and HANA LPARs. Content should be like below.
```
sahitya-suse-15:~ # cat variable_file.txt
disks_configuration: {'counts': [2,2,1], 'names': [data,log,shared], 'paths': [/hana/data,/hana/log,/hana/shared], 'disks_wwn': [600507681082018bc8000000000057e4,600507681082018bc8000000000057e5,600507681082018bc8000000000057e6,600507681082018bc8000000000057e7,600507681082018bc8000000000057e8]}
sahitya-suse-15:~ #
```
To call ansible playbook do

ansible_playbook playbook.yaml -e "SAP_SOLUTION=HANA" 

SAP_SOLUTION can be HANA or NETWEAVER depending on which LPAR you are running the ansible script. This varaible is used to setup saptune solution.
