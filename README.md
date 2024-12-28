
PVFS Lab Setup (Server and Client Implementation)

## 1. Prepare Server and Client

- Server Configuration:
        - Add extra HDD.
        - Set hostname .

  ```yml
  sudo hostnamectl set-hostname ofs-srv-1
  ```

- Client Configuration:
    - Set hostname to ofs-client-1.
 
  ```yml
  sudo hostnamectl set-hostname ofs-client-1
  ```


## 2. Hosts File Configuration (For Both Server and Client)

  - Update /etc/hosts on both the server and client to add entries for each host.             - Replace 192.168.1.x with the actual IP addresses:

```yml
192.168.1.x ofs-srv-1
192.168.1.x ofs-client-1
```

<br>


#  *************Implementation setup ********************


## Server Implementation:

### 1. YUM Repository Configuration:

   - Run the following to configure the YUM repositories:

```yml

cd /etc/yum.repos.d/
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-* 
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

```


### 2. Disable Firewall:

  - Stop and disable firewalld:

```yml
systemctl stop firewalld
systemctl disable firewalld
```

### 3. Install Dependencies:

  - Install necessary packages:

```yml
yum install -y nano
yum -y install epel-release
```

### 4. Disable SELinux:

Edit SELinux configuration to disable it:
bash

```yml

nano /etc/selinux/config

# Change SELINUX=enforcing to SELINUX=disabled

```

### 5. Add ELRepo for Kernel Installation:

  - Add the ELRepo repository:

```yml
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
echo "[elrepo]
name=ELRepo.org Community Enterprise Linux Repository - el8
baseurl=http://elrepo.org/linux/elrepo/el8/\$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org

[elrepo-kernel]
name=ELRepo.org Community Enterprise Linux Kernel Repository - el8
baseurl=http://elrepo.org/linux/kernel/el8/\$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org" | sudo tee /etc/yum.repos.d/elrepo.repo

```

### 6. Install Latest Kernel and Reboot:

  - Install the latest kernel:

```yml

yum -y --enablerepo=elrepo-kernel install kernel-ml
reboot
```

### 7. Install OrangeFS Server:

  - Install OrangeFS server and required dependencies:

```yml
yum -y install orangefs orangefs-server
```

### 8. Format and Mount Extra HDD:

  - Format the extra HDD and mount it:

```yml
mkfs.ext4 /dev/nvme0n2
mount -t ext4 /dev/nvme0n2 /mnt/ofsmnt/
```

### 9. Configure OrangeFS:

  - Generate and configure OrangeFS settings:

```yml
pvfs2-genconfig /etc/orangefs/orangefs.conf
```

  - When prompted, enter the following details:
        - Protocol type: tcp
        - Port number: 3334
        - Directory name: /mnt/ofsmnt
        - Hostnames: ofs-srv-1


### 10. Start OrangeFS Server:

  - Start the OrangeFS server:

```yml
pvfs2-server -f /etc/orangefs/orangefs.conf
```

### 11. Auto-mount Configuration:

```yml

nano /etc/pvfs2tab

tcp://ofs-srv-1:3334/orangefs /mnt/ofsmnt pvfs2

```

### 12. Enable and Start OrangeFS Server:

  - Enable and start the OrangeFS server:

```yml
systemctl enable orangefs-server
systemctl start orangefs-server
systemctl status orangefs-server
```

### 13. Test Connection to Server:

Verify the connection to the server:

```yml

pvfs2-ping -m /mnt/ofsmnt

```


<br>
<br>




## Client Implementation:

### 1. YUM Repository Configuration:

  - Perform the same repository configuration as on the server:

```yml

cd /etc/yum.repos.d/
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-* 
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*

```

### 2. Install Dependencies:

  - Install necessary packages:

```yml

yum -y install orangefs orangefs-fuse

```

### 3. Load OrangeFS Module:

  - Load the OrangeFS module:

```yml
modprobe orangefs
```

### 4. Mount OrangeFS Filesystem:

  - Create the mount directory and configure /etc/pvfs2tab:

```yml

mkdir /mnt/ofsmnt

nano /etc/pvfs2tab

tcp://ofs-srv-1:3334/orangefs /mnt/ofsmnt pvfs2

```

### 5. Verify Connection to Server:

  - Test the connection to the OrangeFS server:

```yml

pvfs2-ping -m /mnt/ofsmnt

```

### 6. Mount OrangeFS on Client:


  - Mount the OrangeFS filesystem on the client:

```yml

pvfs2fuse /mnt/ofsmnt/ -o fs_spec=tcp://ofs-srv-1:3334/orangefs

```

### 7. Verify Mount and Disk Space:

  - Check disk space:

```yml
df -h
```

### 8. Test File Creation:

     - Create a test file to ensure proper functionality:

```ynl
dd if=/dev/zero of=1GB-file bs=1MB count=1024
```


<br>


## Final Verification:

- Both the server (ofs-srv-1) and client (ofs-client-1) should now be successfully configured to communicate via OrangeFS.

- Test by accessing /mnt/ofsmnt on both nodes and ensuring that the filesystem is mounted and operational.





