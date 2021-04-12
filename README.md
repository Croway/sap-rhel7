# SAP on RHEL7
HANA &amp; S/4HANA setup on rhel7

These are the steps that I had to execute in order to install HANA and S/4HANA instances on a single machine, I have used RedHat roles for this, but before executing them you have to make sure that required software has been downloaded and packages are installed on RHEL7.

The static machine I’m using has RHEL7 installed, so I had to install additional packages that are not needed on RHEL8.

## Preconfiguration steps
* Make sure you have RHEL subscription
* Make sure you have a mounted partition with 1TB HDD
* Make sure you can login as root
* Our production line environment is a static machine with 40GB RAM, 1TB HDD and 40 CPUs
* Search repos for your RHEL version with `subscription-manager repos --list | grep ansible` and `subscription-manager repos --list | grep sap`
* Enable repos ansible rpms, sap rpms (netweaver) and sap-hana rpms (sap hana)
* `subscription-manager repos --enable=rhel-7-server-ansible-2-rpms`
* `subscription-manager repos --enable=rhel-sap-hana-for-rhel-7-server-rpms`
* `subscription-manager repos --enable=rhel-sap-for-rhel-7-server-rpms`
* install sap roles and ansible `yum install ansible rhel-system-roles-sap` (not mandatory, you can use github projects and install them via ansible-galaxy)
* Add centos repos by adding the following entry (if not present) in `/etc/yum.repos.d/redhat.repo`
```
[centos]
name=CentOS-7
baseurl=http://ftp.heanet.ie/pub/centos/7/os/x86_64/
enabled=1
gpgcheck=1
gpgkey=http://ftp.heanet.ie/pub/centos/7/os/x86_64/RPM-GPG-KEY-CentOS-7
```
* Add missing/renamed packages
* * `yum install python2-blivet3` (on RHEL7)
* * `yum install python2-jmespath` (on RHEL7)
* Add missing RPM `compat-sap-c++-9`
* * download from https://access.redhat.com/downloads/content/rhel---7/x86_64/4609/compat-sap-c++-9/9.1.1-2.3.el7/x86_64/fd431d51/package
* * install with `rpm -U compat-sap-c++-9-9.1.1-2.3.el7.x86_64.rpm`

## Download required software
* Files and folder structure to download from https://launchpad.support.sap.com/#/softwarecenter (if you want to downlaod another version, make sure version number is the same from the softwarecenter, otherwise errors will appear during installation process)
* In order to download these files I recommend you to use `SAP download manager` from https://support.sap.com/en/my-support/software-downloads.html, I have just `ssh -X` to my remote host and downloaded everything there, since it will require a lot of space (~50GB) and time
```
[root@remote-server playbook]# tree /software
├── HANA_installation
│   └── IMDB_SERVER20_048_5-80002031.SAR
├── S4HANA_installation
│   ├── igsexe_9-80003187.sar
│   ├── igshelper_17-10010245.sar
│   ├── IMDB_CLIENT20_007_26-80002082.SAR
│   ├── S4CORE104_INST_EXPORT_1
│   ├── S4CORE104_INST_EXPORT_10
│   ├── S4CORE104_INST_EXPORT_10.zip
│   ├── S4CORE104_INST_EXPORT_11
│   ├── S4CORE104_INST_EXPORT_11.zip
│   ├── S4CORE104_INST_EXPORT_12
│   ├── S4CORE104_INST_EXPORT_12.zip
│   ├── S4CORE104_INST_EXPORT_13
│   ├── S4CORE104_INST_EXPORT_13.zip
│   ├── S4CORE104_INST_EXPORT_14
│   ├── S4CORE104_INST_EXPORT_14.zip
│   ├── S4CORE104_INST_EXPORT_15
│   ├── S4CORE104_INST_EXPORT_15.zip
│   ├── S4CORE104_INST_EXPORT_16
│   ├── S4CORE104_INST_EXPORT_16.zip
│   ├── S4CORE104_INST_EXPORT_17
│   ├── S4CORE104_INST_EXPORT_17.zip
│   ├── S4CORE104_INST_EXPORT_18
│   ├── S4CORE104_INST_EXPORT_18.zip
│   ├── S4CORE104_INST_EXPORT_19
│   ├── S4CORE104_INST_EXPORT_19.zip
│   ├── S4CORE104_INST_EXPORT_1.zip
│   ├── S4CORE104_INST_EXPORT_2
│   ├── S4CORE104_INST_EXPORT_20
│   ├── S4CORE104_INST_EXPORT_20.zip
│   ├── S4CORE104_INST_EXPORT_21
│   ├── S4CORE104_INST_EXPORT_21.zip
│   ├── S4CORE104_INST_EXPORT_22
│   ├── S4CORE104_INST_EXPORT_22.zip
│   ├── S4CORE104_INST_EXPORT_23
│   ├── S4CORE104_INST_EXPORT_23.zip
│   ├── S4CORE104_INST_EXPORT_24
│   ├── S4CORE104_INST_EXPORT_24.zip
│   ├── S4CORE104_INST_EXPORT_25
│   ├── S4CORE104_INST_EXPORT_25.zip
│   ├── S4CORE104_INST_EXPORT_2.zip
│   ├── S4CORE104_INST_EXPORT_3
│   ├── S4CORE104_INST_EXPORT_3.zip
│   ├── S4CORE104_INST_EXPORT_4
│   ├── S4CORE104_INST_EXPORT_4.zip
│   ├── S4CORE104_INST_EXPORT_5
│   ├── S4CORE104_INST_EXPORT_5.zip
│   ├── S4CORE104_INST_EXPORT_6
│   ├── S4CORE104_INST_EXPORT_6.zip
│   ├── S4CORE104_INST_EXPORT_7
│   ├── S4CORE104_INST_EXPORT_7.zip
│   ├── S4CORE104_INST_EXPORT_8
│   ├── S4CORE104_INST_EXPORT_8.zip
│   ├── S4CORE104_INST_EXPORT_9
│   ├── S4CORE104_INST_EXPORT_9.zip
│   ├── SAPEXE_300-80004393.SAR
│   ├── SAPEXEDB_300-80004392.SAR
│   ├── SAPHANACLIENT
│   ├── SAPHOSTAGENT.SAR
│   └── SWPM20SP08_3-80003424.SAR
├── SAPCAR
│   ├── SAPCAR_1010-70006178.EXE
└── SAPHOSTAGENT
    └── saphostagentrpm_44-20009394.rpm
```
* Add execution permission to SAPCAR `chmod +x /software/SAPCAR/SAPCAR_1010-70006178.EXE` (`chmod +x` to a .exe files :D)
## Ansible playbook
We can write our own ansible playbook that will use ansible roles from redhat `https://github.com/redhat-sap`, this is the folder structure:
```
[root@remote-server playbook]# tree .
├── hosts
├── host_vars
│   └── localhost.yml
└── plays
    ├── requirements.yml
    └── sap-deploy.yml
```
* Install required roles via `ansible-galaxy install -r requirements.yml -p roles`
* Install the ansible playbook via `ansible-playbook -i hosts plays/sap-deploy.yml`

Good luck!

Useful links:
* https://blogs.sap.com/2020/04/08/automating-your-sap-hana-and-s-4hana-by-sap-deployments-using-ansible-part-4/
* https://blogs.sap.com/2020/11/03/automating-the-installation-of-sap-s-4hana-and-sap-hana-on-ibm-power-systems-using-red-hat-ansible/
* https://www.youtube.com/watch?v=hfqVozIUH4w
