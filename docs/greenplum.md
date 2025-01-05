# Create a Greenplum instance on Rocky 9
This process is run on an Intel i7 with 16Gb memory running Rocky 9.

## Prepare machine
### Prepare OS
#### Disable selinux
- In `/etc/selinux/config` set `SELINUX=disabled`

#### Set limits
- In `/etc/sysctl.conf` set:
```
vm.overcommit_memory = 2
vm.overcommit_ratio = 95 
net.ipv4.ip_local_port_range = 10000 65535 
kernel.sem = 250 2048000 200 8192
```
Enable this with `sudo sysctl --system`

- In `/etc/security/limits.conf` set: 
```
soft nofile 524288
hard nofile 524288
soft nproc 131072
hard nproc 131072
```
### Enable passwordless sudo
- `sudo visudo`
```
## Allows people in group wheel to run all commands
#%wheel ALL=(ALL)       ALL

## Same thing without a password
%wheel ALL=(ALL)       NOPASSWD: ALL
```

### Create user gpadmin
- `sudo groupadd gpadmin`

- `sudo useradd gpadmin -r -m -g gpadmin`

- `sudo passwd gpadmin` and set the password.

#### Create ssh key 
- `su - gpadmin`

- `ssh-keygen -t rsa -b 4096` and copy to host `ssh-copy-id -i .ssh/id_rsa gpadmin@<hostname>`

Reboot to apply all above changes.

## Build greenplum from source
### Install dependencies
- `sudo dnf groupinstall "Development Tools"`
- `sudo dnf install python-devel python-pip epel-release perl readline-devel krb5-devel apr-devel libevent-devel libxml2-devel curl-devel bzip2-devel xerces-c-devel`
- `pip install psutil psycopg2 psycopg2-binary`
- `git clone https://github.com/greenplum-db/gpdb-archive`

#### Build libyaml
```
wget https://github.com/yaml/libyaml/archive/refs/tags/0.2.5.tar.gz
tar -xvzf 0.2.5.tar.gz
cd libyaml-0.2.5
./bootstrap
./configure
make
sudo make install
export LDFLAGS="-L/usr/local/lib"
export CPPFLAGS="-I/usr/local/include"
```

### Build greenplum
- `cd gpdb-archive`
- `./configure --with-perl --with-python --with-libxml --with-gssapi --prefix=/usr/local/gpdb`
- `make -j8`
- `sudo make install`

## Greenplum post installation
### Add greenplum path to `.bashrc` of the `gpadmin` user.
- As user gpadmin, `echo ". /usr/local/gpdb/greenplum_path.sh" >> .bashrc`

### Create a host file with all your greenplum nodes in it (for me this is just one node)
- As user gpadmin, `echo "<hostname> > hostfile`

### Create data directory
- As user gpadmin, `mkdir ./data`

- `vim .bashrc`
``` 
export MASTER_DATA_DIRECTORY=/home/gpadmin/data/gpsne-1
```
- `source .bashrc`
  
## Configure greenplum cluster
### Copy sample configuration to $HOME
- As gpadmin, `cp $GPHOME/docs/cli_help/gpconfigs/gpinitsystem_singlenode ~`

### Modify config file
- `vim gpinitsystem_singlenode` and change:
```
MACHINE_LIST_FILE=./hostfile
declare -a DATA_DIRECTORY=(/home/gpadmin/data /home/gpadmin/data /home/gpadmin/data) # I created 3 shards, create as many as you need.
COORDINATOR_HOSTNAME=<hostname>
COORDINATOR_DIRECTORY=/home/gpadmin/data
```
## Create cluster
- As gpadmin, `gpinitsystem -c gpinitsystem_singlenode`

Greenplum should now be running
```
gpadmin    10780       1  0 12:19 ?        00:00:00 /usr/local/gpdb/bin/postgres -D /home/gpadmin/data/gpsne0 -c gp_role=execute
gpadmin    10781       1  0 12:19 ?        00:00:00 /usr/local/gpdb/bin/postgres -D /home/gpadmin/data/gpsne1 -c gp_role=execute
gpadmin    10782       1  0 12:19 ?        00:00:00 /usr/local/gpdb/bin/postgres -D /home/gpadmin/data/gpsne2 -c gp_role=execute
gpadmin    10783   10780  0 12:19 ?        00:00:00 postgres:  6000, logger process   
gpadmin    10784   10781  0 12:19 ?        00:00:00 postgres:  6001, logger process   
gpadmin    10785   10782  0 12:19 ?        00:00:00 postgres:  6002, logger process   
gpadmin    10789   10780  0 12:19 ?        00:00:00 postgres:  6000, checkpointer   
gpadmin    10790   10780  0 12:19 ?        00:00:00 postgres:  6000, background writer   
gpadmin    10791   10780  0 12:19 ?        00:00:00 postgres:  6000, walwriter   
gpadmin    10792   10780  0 12:19 ?        00:00:00 postgres:  6000, autovacuum launcher   
gpadmin    10793   10780  0 12:19 ?        00:00:00 postgres:  6000, stats collector   
gpadmin    10794   10780  0 12:19 ?        00:00:00 postgres:  6000, logical replication launcher   
gpadmin    10795   10781  0 12:19 ?        00:00:00 postgres:  6001, checkpointer   
gpadmin    10796   10781  0 12:19 ?        00:00:00 postgres:  6001, background writer   
gpadmin    10797   10781  0 12:19 ?        00:00:00 postgres:  6001, walwriter   
gpadmin    10798   10781  0 12:19 ?        00:00:00 postgres:  6001, autovacuum launcher   
gpadmin    10799   10781  0 12:19 ?        00:00:00 postgres:  6001, stats collector   
gpadmin    10800   10781  0 12:19 ?        00:00:00 postgres:  6001, logical replication launcher   
gpadmin    10801   10782  0 12:19 ?        00:00:00 postgres:  6002, checkpointer   
gpadmin    10802   10782  0 12:19 ?        00:00:00 postgres:  6002, background writer   
gpadmin    10803   10782  0 12:19 ?        00:00:00 postgres:  6002, walwriter   
gpadmin    10804   10782  0 12:19 ?        00:00:00 postgres:  6002, autovacuum launcher   
gpadmin    10805   10782  0 12:19 ?        00:00:00 postgres:  6002, stats collector   
gpadmin    10806   10782  0 12:19 ?        00:00:00 postgres:  6002, logical replication launcher   
gpadmin    10809       1  0 12:19 ?        00:00:00 /usr/local/gpdb/bin/postgres -D /home/gpadmin/data/gpsne-1 -c gp_role=dispatch
gpadmin    10810   10809  0 12:19 ?        00:00:00 postgres:  5432, master logger process   
gpadmin    10812   10809  0 12:19 ?        00:00:00 postgres:  5432, checkpointer   
gpadmin    10813   10809  0 12:19 ?        00:00:00 postgres:  5432, background writer   
gpadmin    10814   10809  0 12:19 ?        00:00:00 postgres:  5432, walwriter   
gpadmin    10815   10809  0 12:19 ?        00:00:00 postgres:  5432, autovacuum launcher   
gpadmin    10816   10809  0 12:19 ?        00:00:00 postgres:  5432, stats collector   
gpadmin    10817   10809  0 12:19 ?        00:00:00 postgres:  5432, dtx recovery process   
gpadmin    10818   10809  0 12:19 ?        00:00:00 postgres:  5432, ftsprobe process   
gpadmin    10833   10809  0 12:19 ?        00:00:00 postgres:  5432, logical replication launcher
```

### After cluster creation add data directory to environment.
- As gpadmin, `echo "COORDINATOR_DATA_DIRECTORY=/home/gpadmin/data/gpsne-1" >> .bashrc`

## Test cluster
- As gpadmin, `createdb test`

- `psql test`
```
psql (12.12)
Type "help" for help.

test=# \timing
Timing is on.

test=# create table t1 as select generate_series(1,1000000) as colA distributed by (colA);
SELECT 1000000
Time: 683.351 ms

test=# select count(*) from t1;
  count  
---------
 1000000
(1 row)
Time: 164.377 ms

test=# select sum(colA) from t1;
     sum      
--------------
 500000500000
(1 row)
Time: 89.030 ms
```

### EPAS 17
```
[enterprisedb@onion ~]$ psql edb
psql (17.2.0)
Type "help" for help.

edb=# \timing
Timing is on.

edb=# create table t1 as select generate_series(1,1000000);
SELECT 1000000
Time: 1483.358 ms (00:01.483)

edb=# select count(*) from t1;
  count  
---------
 1000000
(1 row)
Time: 67.455 ms

edb=# select sum(generate_series) from t1;
     sum      
--------------
 500000500000
(1 row)
Time: 61.911 ms
```
## Remove the system
Normally you should be able to remove the system using `gpdeletesystem`. If you get the error `gpadmin-[CRITICAL]:-Error deleting system: FATAL:  System was started in single node mode - only utility mode connections are allowed`, then just stop the cluster using `gpstop` and remove the directoes and files under the `./data` directory. There must be a better way, but i haven't figured that out yet.

# Query CSV files from storage
The intent is to be able to query the many CSV files i have on a CIFS volume using a foreign data wrapper.

## Mount the SMB share on the server
### Install Samba if not already installed
```
[ton@onion ~]$ sudo dnf install cifs-utils samba samba-common samba-client
Last metadata expiration check: 2:52:41 ago on Thu 02 Jan 2025 10:06:42 AM CET.
Package cifs-utils-7.0-5.el9.x86_64 is already installed.
Package samba-common-4.20.2-2.el9_5.noarch is already installed.
Package samba-client-4.20.2-2.el9_5.x86_64 is already installed.
Dependencies resolved.
=============================================================================================================
 Package                            Architecture       Version                      Repository          Size
=============================================================================================================
Installing:
 samba                              x86_64             4.20.2-2.el9_5               baseos             939 k
Installing dependencies:
 libnetapi                          x86_64             4.20.2-2.el9_5               baseos             143 k
 samba-common-tools                 x86_64             4.20.2-2.el9_5               baseos             483 k
 samba-dcerpc                       x86_64             4.20.2-2.el9_5               baseos             716 k
 samba-ldb-ldap-modules             x86_64             4.20.2-2.el9_5               baseos              27 k
 samba-libs                         x86_64             4.20.2-2.el9_5               baseos             124 k

Transaction Summary
=============================================================================================================
Install  6 Packages

Total download size: 2.4 M
Installed size: 8.6 M
Is this ok [y/N]: y
Downloading Packages:
(1/6): samba-libs-4.20.2-2.el9_5.x86_64.rpm                                  353 kB/s | 124 kB     00:00    
(2/6): samba-ldb-ldap-modules-4.20.2-2.el9_5.x86_64.rpm                      414 kB/s |  27 kB     00:00    
(3/6): samba-common-tools-4.20.2-2.el9_5.x86_64.rpm                          979 kB/s | 483 kB     00:00    
(4/6): samba-4.20.2-2.el9_5.x86_64.rpm                                       1.7 MB/s | 939 kB     00:00    
(5/6): samba-dcerpc-4.20.2-2.el9_5.x86_64.rpm                                2.5 MB/s | 716 kB     00:00    
(6/6): libnetapi-4.20.2-2.el9_5.x86_64.rpm                                   358 kB/s | 143 kB     00:00    
-------------------------------------------------------------------------------------------------------------
Total                                                                        2.1 MB/s | 2.4 MB     00:01     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                     1/1 
  Installing       : libnetapi-4.20.2-2.el9_5.x86_64                                                     1/6 
  Installing       : samba-libs-4.20.2-2.el9_5.x86_64                                                    2/6 
  Installing       : samba-dcerpc-4.20.2-2.el9_5.x86_64                                                  3/6 
  Installing       : samba-ldb-ldap-modules-4.20.2-2.el9_5.x86_64                                        4/6 
  Installing       : samba-common-tools-4.20.2-2.el9_5.x86_64                                            5/6 
  Installing       : samba-4.20.2-2.el9_5.x86_64                                                         6/6 
  Running scriptlet: samba-4.20.2-2.el9_5.x86_64                                                         6/6 
  Verifying        : samba-libs-4.20.2-2.el9_5.x86_64                                                    1/6 
  Verifying        : samba-common-tools-4.20.2-2.el9_5.x86_64                                            2/6 
  Verifying        : samba-4.20.2-2.el9_5.x86_64                                                         3/6 
  Verifying        : samba-ldb-ldap-modules-4.20.2-2.el9_5.x86_64                                        4/6 
  Verifying        : samba-dcerpc-4.20.2-2.el9_5.x86_64                                                  5/6 
  Verifying        : libnetapi-4.20.2-2.el9_5.x86_64                                                     6/6 

Installed:
  libnetapi-4.20.2-2.el9_5.x86_64                            samba-4.20.2-2.el9_5.x86_64                     
  samba-common-tools-4.20.2-2.el9_5.x86_64                   samba-dcerpc-4.20.2-2.el9_5.x86_64              
  samba-ldb-ldap-modules-4.20.2-2.el9_5.x86_64               samba-libs-4.20.2-2.el9_5.x86_64                

Complete!
```
### Create directory where to mount the SMB share.
```
[ton@onion ~]$ mkdir /mnt/usbdrive
```
### Create a Samba credentials file.
```
[ton@onion ~]$ sudo cat - > /root/.smbcredentials_USB
username=ton
password=<password>
^Z
[ton@onion ~]$ chmod 600 /root/.smbcredentials_USB
```
### Test mount the share, the add it to `/etc/fstab`.
```
[ton@onion ~]$ sudo mount -t cifs //192.168.0.10/USB /mnt/usbdrive -o credentials=/root/.smbcredentials_USB
[ton@onion ~]$ cd /mnt/usbdrive/
[ton@onion usbdrive]$ ls
 Breaches        iPhone          Ringtones         Videos
'Curious Moon'   ISOs            ROMS              Webs
 Downloads      'Motos Coches'   Shared            Work
 Drivers         Personal        Software         'xBox 360'
 eBooks          Photos         'Surabhi Pandey'
 Hacking         Radio           UFO

[ton@onion ~]$ umount /mnt/usbdrive
[ton@onion ~]$ echo "//192.168.0.10/USB /mnt/usbdrive cifs ro,auto,credentials=/root/.smbcredentials_USB 0 0" >> /etc/fstab
[ton@onion ~]$ sudo systemctl daemon-reload
[ton@onion ~]$ sudo mount -a -v
/                        : ignored
/boot                    : already mounted
/boot/efi                : already mounted
/home                    : already mounted
none                     : ignored
Host "192.168.0.10" resolved to the following IP addresses: 192.168.0.10
mount.cifs kernel mount options: ip=192.168.0.10,unc=\\192.168.0.10\USB,user=ton,pass=********
/mnt/usbdrive            : successfully mounted
[ton@onion ~]$ ls /mnt/usbdrive
 Breaches        Downloads   eBooks    iPhone  'Motos Coches'   Photos   Ringtones   Shared    'Surabhi Pandey'   Videos   Work
'Curious Moon'   Drivers     Hacking   ISOs     Personal        Radio    ROMS        Software   UFO               Webs    'xBox 360'
```

## Not that the server is prepared, let's configure the database.
### Create the `file_fdw` extension.
```
✗ ~ psql -h onion -p 5432 -U gpadmin breaches
psql (17.2 (Homebrew), server 12.12)
Type "help" for help.

breaches=# \dx
                                List of installed extensions
      Name       | Version |   Schema   |                    Description                    
-----------------+---------+------------+---------------------------------------------------
 gp_exttable_fdw | 1.0     | pg_catalog | External Table Foreign Data Wrapper for Greenplum
 gp_toolkit      | 1.6     | gp_toolkit | various GPDB administrative views/functions
 plpgsql         | 1.0     | pg_catalog | PL/pgSQL procedural language
(3 rows)

breaches=# create extension file_fdw;
CREATE EXTENSION

breaches=# \dx
                                List of installed extensions
      Name       | Version |   Schema   |                    Description                    
-----------------+---------+------------+---------------------------------------------------
 file_fdw        | 1.0     | public     | foreign-data wrapper for flat file access
 gp_exttable_fdw | 1.0     | pg_catalog | External Table Foreign Data Wrapper for Greenplum
 gp_toolkit      | 1.6     | gp_toolkit | various GPDB administrative views/functions
 plpgsql         | 1.0     | pg_catalog | PL/pgSQL procedural language
(4 rows)
```
### Create the server and the table definition.
```
breaches=# CREATE SERVER csv_server FOREIGN DATA WRAPPER file_fdw;
CREATE SERVER
breaches=# CREATE FOREIGN TABLE accounts (email TEXT, password TEXT) SERVER csv_server OPTIONS (filename '/mnt/usbdrive/Breaches/120k.txt',delimiter ':',null 'NULL', header 'false');
CREATE FOREIGN TABLE
breaches=# select * from accounts;
                       email                        |                              password                              
----------------------------------------------------+--------------------------------------------------------------------
 00_gerard00_@yahoo.com.ph                          | 001gerard
 01010101@meralco.com.ph                            | nghao
 012089_neil@yahoo.com.ph                           | music6
 015_jericho@yahoo.com.ph                           | jericho
```

## Next is to find a way to query ALL files in the directory structure.
