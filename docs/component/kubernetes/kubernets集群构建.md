[TOC]

**本系列文档内容涵盖 (但不限于) 以下技术领域：**

- **KubeSphere**
- **Kubernetes**
- **Ansible**
- **自动化运维**
- **CNCF 技术栈**

## **1. 本文简介**

Kubernetes v1.24.0 在 2022-05-03 正式发布了，v1.25.0在2022-08-24正式发布。不再支持 Docker 作为容器运行时，纯纯的使用 Containerd 安装部署。

在公网安装稍微简单点，在私网安装特别麻烦。特别是在使用欧拉的版本尤其是麻烦。因此，本文介绍了在私有云离线安装方式。本文内容配置只适合于学习测试环境，生产环境**绝对**不能用。生产环境需要做特定的配置

本文使用安装外部3节点etcd，3控制节点和5个业务节点，分布三地。

> **本文知识点**

- 定级：**入门级**
- Kubernetes 1.25
- Containers 常用操作
- Ansible 日常使用

> **演示服务器配置**

| 主机名    | 操作系统类型版本   | IP             | CPU  | 内存 | 用途                 |
| --------- | ------------------ | -------------- | ---- | ---- | -------------------- |
| --        | wsl2               | 172.25.41.102  | --   | --   | Ansible 运维控制节点 |
| shmaster1 | Euler2R9 (x86_64)  | 10.160.144.93  | 2    | 5    | master/etcd          |
| shmaster2 | Euler2R9 (x86_64)  | 10.160.144.96  | 2    | 5    | master/etcd          |
| shmaster3 | Euler2R9 (x86_64)  | 10.160.144.111 | 2    | 5    | master/etcd          |
| bjwork1   | Euler2R9 (x86_64)  | 10.23.30.206   | 8    | 8    | worker               |
| bjwork2   | Euler2R9 (x86_64)  | 10.23.30.208   | 8    | 8    | worker               |
| cdwork1   | Euler2R9 (x86_64)  | 10.148.136.62  | 8    | 16   | worker               |
| cdwork2   | Euler2R9 (x86_64)  | 10.148.136.63  | 8    | 16   | worker               |
| cdwork3   | Euler2R10 (x86_64) | 10.148.151.120 | 8    | 16   | worker               |

------

## **2. Ansible 配置**

Ansible的安装不在此文说明

### **2.1. 切换到 ansible 代码目录**

```text
[root@zdevops-master dev]# cd /data/ansible/ansible-zdevops/inventories/dev
[root@zdevops-master dev]# source /opt/ansible2.8/bin/activate
```

### **2.2. 编辑 hosts 配置文件**

> **01-编辑 k8s.hosts**

```text
[master]
k8s-master-0 ansible_ssh_host=192.168.9.61  host_name=k8s-master-0
k8s-master-1 ansible_ssh_host=192.168.9.62  host_name=k8s-master-1
k8s-master-2 ansible_ssh_host=192.168.9.63  host_name=k8s-master-2

[k8s_worker]
k8s-worker-0 ansible_ssh_host=192.168.9.65 host_name=k8s-worker-0

[servers:children]
k8s_master
k8s_worker

[servers:vars]
ansible_connection=paramiko
ansible_ssh_user=root
ansible_ssh_pass=F@ywwpTj4bJtYwzpwCqD
```

------

## **3. K8S 服务器初始化**

### **3.1. 检测服务器连通性**

```text
# 利用 ansible 检测服务器的连通性

(ansible2.8) [root@zdevops-master dev]# ansible servers -m ping -i k8s-hosts 
/opt/ansible2.8/lib/python2.7/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
  from cryptography.exceptions import InvalidSignature

k8s-master-0 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
k8s-master-2 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
k8s-worker-0 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
k8s-master-1 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

### **3.2. 初始化服务器配置**

```text
# 利用 ansible-playbook 初始化服务器配置
(ansible2.8) [root@zdevops-master dev]# ansible-playbook ../../playbooks/init-base.yaml -i k8s-hosts 
/opt/ansible2.8/lib/python2.7/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
  from cryptography.exceptions import InvalidSignature

PLAY [初始化服务器配置.] *************************************************************************************************************

TASK [01-停止并禁用firewalld服务.] **************************************************************************************************
changed: [k8s-master-1]
changed: [k8s-worker-0]
changed: [k8s-master-0]
changed: [k8s-master-2]

TASK [02-配置主机名.] *************************************************************************************************************
changed: [k8s-master-1]
changed: [k8s-worker-0]
changed: [k8s-master-2]
changed: [k8s-master-0]

TASK [03-配置/etc/hosts.] ******************************************************************************************************
changed: [k8s-master-1]
changed: [k8s-master-0]
changed: [k8s-master-2]
changed: [k8s-worker-0]

TASK [04-配置时区.] **************************************************************************************************************
ok: [k8s-master-1]
ok: [k8s-worker-0]
ok: [k8s-master-0]
ok: [k8s-master-2]

TASK [05-升级操作系统.] ************************************************************************************************************
skipping: [k8s-master-0]
skipping: [k8s-master-1]
skipping: [k8s-master-2]
skipping: [k8s-worker-0]

TASK [05-升级操作系统后如果需要重启，则重启服务器.] **********************************************************************************************
skipping: [k8s-master-0]
skipping: [k8s-master-1]
skipping: [k8s-master-2]
skipping: [k8s-worker-0]

TASK [05-等待服务器完成重启.] *********************************************************************************************************
skipping: [k8s-master-0]
skipping: [k8s-master-1]
skipping: [k8s-master-2]
skipping: [k8s-worker-0]

PLAY [安装配置chrony服务器.] ********************************************************************************************************

TASK [01-安装chrony软件包.] *******************************************************************************************************
changed: [k8s-master-1] => (item=[u'chrony'])
changed: [k8s-worker-0] => (item=[u'chrony'])
changed: [k8s-master-2] => (item=[u'chrony'])
changed: [k8s-master-0] => (item=[u'chrony'])

TASK [02-配置chrony.conf.] *****************************************************************************************************
changed: [k8s-master-0]
changed: [k8s-master-1]
changed: [k8s-worker-0]
changed: [k8s-master-2]

TASK [03-确认chrony服务启动并实现开机自启.] ***********************************************************************************************
changed: [k8s-master-1]
changed: [k8s-master-2]
changed: [k8s-worker-0]
changed: [k8s-master-0]

PLAY RECAP *******************************************************************************************************************
k8s-master-0               : ok=0    changed=7    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
k8s-master-1               : ok=0    changed=7    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
k8s-master-2               : ok=0    changed=7    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
k8s-worker-0                 : ok=0    changed=7    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
```

### **3.3. 挂载数据盘**

```text
# 利用 ansible-playbook 初始化主机数据盘
# 注意 -e data_disk_path="/data" 指定挂载目录

(ansible2.8) [root@zdevops-master dev]# ansible-playbook ../../playbooks/init-disk.yaml -i k8s-hosts -e data_disk_path="/data"
/opt/ansible2.8/lib/python2.7/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
  from cryptography.exceptions import InvalidSignature


PLAY [初始化磁盘.] ****************************************************************************************************

TASK [01-数据磁盘分区.] ************************************************************************************************
changed: [k8s-master-1]
changed: [k8s-master-0]
changed: [k8s-worker-0]
changed: [k8s-master-2]

TASK [02-格式化数据磁盘.] ***********************************************************************************************
changed: [k8s-master-1]
changed: [k8s-worker-0]
changed: [k8s-master-0]
changed: [k8s-master-2]

TASK [03-挂载数据盘.] *************************************************************************************************
changed: [k8s-master-0]
changed: [k8s-master-2]
changed: [k8s-worker-0]
changed: [k8s-master-1]

PLAY RECAP *******************************************************************************************************
k8s-master-0               : ok=3    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-master-1               : ok=3    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-master-2               : ok=3    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker-0               : ok=3    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### **3.4. 验证数据盘的挂载**

```text
# 利用 ansible 验证数据盘是否格式化并挂载

(ansible2.8) [root@zdevops-master dev]#  ansible -i k8s-hosts servers -m shell -a 'df -h'
/opt/ansible2.8/lib/python2.7/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
  from cryptography.exceptions import InvalidSignature

k8s-master-2 | CHANGED | rc=0 >>
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 7.9G     0  7.9G   0% /dev
tmpfs                    7.9G     0  7.9G   0% /dev/shm
tmpfs                    7.9G  8.8M  7.9G   1% /run
tmpfs                    7.9G     0  7.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root   37G  1.5G   36G   4% /
/dev/sda1               1014M  168M  847M  17% /boot
/dev/sdb1                200G   33M  200G   1% /data
tmpfs                    1.6G     0  1.6G   0% /run/user/0

k8s-master-1 | CHANGED | rc=0 >>
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 7.9G     0  7.9G   0% /dev
tmpfs                    7.9G     0  7.9G   0% /dev/shm
tmpfs                    7.9G  8.8M  7.9G   1% /run
tmpfs                    7.9G     0  7.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root   37G  1.5G   36G   4% /
/dev/sda1               1014M  168M  847M  17% /boot
/dev/sdb1                200G   33M  200G   1% /data
tmpfs                    1.6G     0  1.6G   0% /run/user/0

k8s-worker-0 | CHANGED | rc=0 >>
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 7.9G     0  7.9G   0% /dev
tmpfs                    7.9G     0  7.9G   0% /dev/shm
tmpfs                    7.9G  8.8M  7.9G   1% /run
tmpfs                    7.9G     0  7.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root   37G  1.5G   36G   4% /
/dev/sda1               1014M  168M  847M  17% /boot
/dev/sdb1                200G   33M  200G   1% /data
tmpfs                    1.6G     0  1.6G   0% /run/user/0

k8s-master-0 | CHANGED | rc=0 >>
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 7.9G     0  7.9G   0% /dev
tmpfs                    7.9G     0  7.9G   0% /dev/shm
tmpfs                    7.9G  8.8M  7.9G   1% /run
tmpfs                    7.9G     0  7.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root   37G  1.5G   36G   4% /
/dev/sda1               1014M  168M  847M  17% /boot
tmpfs                    1.6G     0  1.6G   0% /run/user/0
/dev/sdb1                200G   33M  200G   1% /data


# 利用 ansible 验证数据盘是否配置自动挂载

(ansible2.8) [root@zdevops-master dev]# ansible -i k8s-hosts servers -m shell -a 'tail -1  /etc/fstab'
/opt/ansible2.8/lib/python2.7/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
  from cryptography.exceptions import InvalidSignature
k8s-master-2 | CHANGED | rc=0 >>
/dev/sdb1 /data xfs defaults 0 0

k8s-master-1 | CHANGED | rc=0 >>
/dev/sdb1 /data xfs defaults 0 0

k8s-master-0 | CHANGED | rc=0 >>
/dev/sdb1 /data xfs defaults 0 0

k8s-worker-0 | CHANGED | rc=0 >>
/dev/sdb1 /data xfs defaults 0 0
```

------

## **4. K8S 节点通用配置**

### **4.1. 系统基础配置**

> **01-关闭 swap**

```text
[root@k8s-master-0 ~]# swapoff -a
```

> **02-确认 MAC 地址和 product_uuid 对于每个节点都是唯一的**

```text
# MAC
[root@k8s-master-0 ~]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:85:67:89 brd ff:ff:ff:ff:ff:ff
[root@k8s-master-0 ~]# cat /sys/class/dmi/id/product_
cat: /sys/class/dmi/id/product_: No such file or directory

# product_uuid
[root@k8s-master-0 ~]# cat /sys/class/dmi/id/product_uuid 
4205AB12-9DD3-9174-D108-6CB2B1FC169Dxxxxxxxxxx [root@k8s-master-0 ~]# ip link1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:002: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000    link/ether 00:50:56:85:67:89 brd ff:ff:ff:ff:ff:ff[root@k8s-master-0 ~]# cat /sys/class/dmi/id/product_cat: /sys/class/dmi/id/product_: No such file or directory[root@k8s-master-0 ~]# cat /sys/class/dmi/id/product_uuid 4205AB12-9DD3-9174-D108-6CB2B1FC169Dip linkcat /sys/class/dmi/id/product_uuid
```

> **03-关闭 SELinux(系统初始化时已关闭)**

```text
# Set SELinux in permissive mode (effectively disabling it)
[root@k8s-master-0 ~]# setenforce 0
[root@k8s-master-0 ~]# sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

### **4.2. 安装容器运行时 (Container runtimes)-containerd**

> **01-安装配置前提条件**

```text
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

# 必须执行，否则后面init集群的时候会报错
[root@k8s-master-0 ~]# modprobe overlay
[root@k8s-master-0 ~]# modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF


# Apply sysctl params without reboot
[root@k8s-master-0 ~]# sysctl --system
* Applying /usr/lib/sysctl.d/00-system.conf ...
net.bridge.bridge-nf-call-ip6tables = 0
net.bridge.bridge-nf-call-iptables = 0
net.bridge.bridge-nf-call-arptables = 0
* Applying /usr/lib/sysctl.d/10-default-yama-scope.conf ...
kernel.yama.ptrace_scope = 0
* Applying /usr/lib/sysctl.d/50-default.conf ...
kernel.sysrq = 16
kernel.core_uses_pid = 1
kernel.kptr_restrict = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.promote_secondaries = 1
net.ipv4.conf.all.promote_secondaries = 1
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
* Applying /etc/sysctl.d/99-sysctl.conf ...
* Applying /etc/sysctl.d/k8s.conf ...
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
* Applying /etc/sysctl.conf ...

# 使用下面的命令从指定的配置文件加载也可以
# [root@k8s-master-0 ~]# sysctl -p /etc/sysctl.d/k8s.conf 
```

> **02-安装 Containerd**

```text
# 配置软件源
[root@k8s-master-0 ~]# yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
Loaded plugins: fastestmirror
adding repo from: http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
grabbing file http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo

[root@k8s-master-0 ~]# ls /etc/yum.repos.d/
CentOS-Base.repo  CentOS-Debuginfo.repo  CentOS-Media.repo    CentOS-Vault.repo          docker-ce.repo
CentOS-CR.repo    CentOS-fasttrack.repo  CentOS-Sources.repo  CentOS-x86_64-kernel.repo


# 查找可用的 containerd 包
[root@k8s-master-0 ~]# yum search containerd
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
============================================ N/S matched: containerd =============================================
containerd.io.x86_64 : An industry-standard container runtime

  Name and summary matches only, use "search all" for everything.

# 安装 containerd
[root@k8s-master-0 ~]# yum install containerd.io -y
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
Resolving Dependencies
--> Running transaction check
---> Package containerd.io.x86_64 0:1.6.4-3.1.el7 will be installed
--> Processing Dependency: container-selinux >= 2:2.74 for package: containerd.io-1.6.4-3.1.el7.x86_64
--> Running transaction check
---> Package container-selinux.noarch 2:2.119.2-1.911c772.el7_8 will be installed
--> Processing Dependency: policycoreutils-python for package: 2:container-selinux-2.119.2-1.911c772.el7_8.noarch
--> Running transaction check
---> Package policycoreutils-python.x86_64 0:2.5-34.el7 will be installed
--> Processing Dependency: setools-libs >= 3.3.8-4 for package: policycoreutils-python-2.5-34.el7.x86_64
--> Processing Dependency: libsemanage-python >= 2.5-14 for package: policycoreutils-python-2.5-34.el7.x86_64
--> Processing Dependency: audit-libs-python >= 2.1.3-4 for package: policycoreutils-python-2.5-34.el7.x86_64
--> Processing Dependency: python-IPy for package: policycoreutils-python-2.5-34.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.4)(64bit) for package: policycoreutils-python-2.5-34.el7.x86_64
--> Processing Dependency: libqpol.so.1(VERS_1.2)(64bit) for package: policycoreutils-python-2.5-34.el7.x86_64
--> Processing Dependency: libcgroup for package: policycoreutils-python-2.5-34.el7.x86_64
--> Processing Dependency: libapol.so.4(VERS_4.0)(64bit) for package: policycoreutils-python-2.5-34.el7.x86_64
--> Processing Dependency: checkpolicy for package: policycoreutils-python-2.5-34.el7.x86_64
--> Processing Dependency: libqpol.so.1()(64bit) for package: policycoreutils-python-2.5-34.el7.x86_64
--> Processing Dependency: libapol.so.4()(64bit) for package: policycoreutils-python-2.5-34.el7.x86_64
--> Running transaction check
---> Package audit-libs-python.x86_64 0:2.8.5-4.el7 will be installed
---> Package checkpolicy.x86_64 0:2.5-8.el7 will be installed
---> Package libcgroup.x86_64 0:0.41-21.el7 will be installed
---> Package libsemanage-python.x86_64 0:2.5-14.el7 will be installed
---> Package python-IPy.noarch 0:0.75-6.el7 will be installed
---> Package setools-libs.x86_64 0:3.3.8-4.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==================================================================================================================
 Package                        Arch           Version                             Repository                Size
==================================================================================================================
Installing:
 containerd.io                  x86_64         1.6.4-3.1.el7                       docker-ce-stable          33 M
Installing for dependencies:
 audit-libs-python              x86_64         2.8.5-4.el7                         base                      76 k
 checkpolicy                    x86_64         2.5-8.el7                           base                     295 k
 container-selinux              noarch         2:2.119.2-1.911c772.el7_8           extras                    40 k
 libcgroup                      x86_64         0.41-21.el7                         base                      66 k
 libsemanage-python             x86_64         2.5-14.el7                          base                     113 k
 policycoreutils-python         x86_64         2.5-34.el7                          base                     457 k
 python-IPy                     noarch         0.75-6.el7                          base                      32 k
 setools-libs                   x86_64         3.3.8-4.el7                         base                     620 k

Transaction Summary
==================================================================================================================
Install  1 Package (+8 Dependent packages)

Total download size: 35 M
Installed size: 130 M
Downloading packages:
(1/9): container-selinux-2.119.2-1.911c772.el7_8.noarch.rpm                                |  40 kB  00:00:00     
(2/9): audit-libs-python-2.8.5-4.el7.x86_64.rpm                                            |  76 kB  00:00:00     
(3/9): libcgroup-0.41-21.el7.x86_64.rpm                                                    |  66 kB  00:00:00     
(4/9): checkpolicy-2.5-8.el7.x86_64.rpm                                                    | 295 kB  00:00:00     
(5/9): libsemanage-python-2.5-14.el7.x86_64.rpm                                            | 113 kB  00:00:00     
(6/9): python-IPy-0.75-6.el7.noarch.rpm                                                    |  32 kB  00:00:00     
(7/9): policycoreutils-python-2.5-34.el7.x86_64.rpm                                        | 457 kB  00:00:00     
(8/9): setools-libs-3.3.8-4.el7.x86_64.rpm                                                 | 620 kB  00:00:00     
warning: /var/cache/yum/x86_64/7/docker-ce-stable/packages/containerd.io-1.6.4-3.1.el7.x86_64.rpm: Header V4 RSA/SHA512 Signature, key ID 621e9f35: NOKEY
Public key for containerd.io-1.6.4-3.1.el7.x86_64.rpm is not installed
(9/9): containerd.io-1.6.4-3.1.el7.x86_64.rpm                                              |  33 MB  00:00:21     
------------------------------------------------------------------------------------------------------------------
Total                                                                             1.6 MB/s |  35 MB  00:00:21     
Retrieving key from https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
Importing GPG key 0x621E9F35:
 Userid     : "Docker Release (CE rpm) <docker@docker.com>"
 Fingerprint: 060a 61c5 1b55 8a7f 742b 77aa c52f eb6b 621e 9f35
 From       : https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : setools-libs-3.3.8-4.el7.x86_64                                                                1/9 
  Installing : libcgroup-0.41-21.el7.x86_64                                                                   2/9 
  Installing : audit-libs-python-2.8.5-4.el7.x86_64                                                           3/9 
  Installing : python-IPy-0.75-6.el7.noarch                                                                   4/9 
  Installing : libsemanage-python-2.5-14.el7.x86_64                                                           5/9 
  Installing : checkpolicy-2.5-8.el7.x86_64                                                                   6/9 
  Installing : policycoreutils-python-2.5-34.el7.x86_64                                                       7/9 
  Installing : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch                                             8/9 
setsebool:  SELinux is disabled.
  Installing : containerd.io-1.6.4-3.1.el7.x86_64                                                             9/9 
  Verifying  : checkpolicy-2.5-8.el7.x86_64                                                                   1/9 
  Verifying  : libsemanage-python-2.5-14.el7.x86_64                                                           2/9 
  Verifying  : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch                                             3/9 
  Verifying  : python-IPy-0.75-6.el7.noarch                                                                   4/9 
  Verifying  : containerd.io-1.6.4-3.1.el7.x86_64                                                             5/9 
  Verifying  : policycoreutils-python-2.5-34.el7.x86_64                                                       6/9 
  Verifying  : audit-libs-python-2.8.5-4.el7.x86_64                                                           7/9 
  Verifying  : libcgroup-0.41-21.el7.x86_64                                                                   8/9 
  Verifying  : setools-libs-3.3.8-4.el7.x86_64                                                                9/9 

Installed:
  containerd.io.x86_64 0:1.6.4-3.1.el7                                                                            

Dependency Installed:
  audit-libs-python.x86_64 0:2.8.5-4.el7                      checkpolicy.x86_64 0:2.5-8.el7                     
  container-selinux.noarch 2:2.119.2-1.911c772.el7_8          libcgroup.x86_64 0:0.41-21.el7                     
  libsemanage-python.x86_64 0:2.5-14.el7                      policycoreutils-python.x86_64 0:2.5-34.el7         
  python-IPy.noarch 0:0.75-6.el7                              setools-libs.x86_64 0:3.3.8-4.el7                  

Complete!
```

> **03-配置 containerd**

```text
# 备份默认配置文件，并生成一份更全的默认配置文件
[root@k8s-master-0 ~]# cp /etc/containerd/config.toml /etc/containerd/config.toml.ori
[root@k8s-master-0 ~]# containerd  config default > /etc/containerd/config.toml

# 替换默认的 sandbox_image
# sandbox_image = "k8s.gcr.io/pause:3.6" 为 "registry.aliyuncs.com/google_containers/pause:3.6"
[root@k8s-master-0 ~]# sed -i 's#k8s.gcr.io#registry.aliyuncs.com/google_containers#g' /etc/containerd/config.toml

# 配置 systemd cgroup driver
# 修改下面配置中的 SystemdCgroup = false 为true
#[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
#  ...
#  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
#    SystemdCgroup = true
[root@k8s-master-0 ~]# sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml 
    
# 配置 containerd的存储路径
# 修改 root = "/var/lib/containerd"为root = "/data/containerd"
[root@k8s-master-0 ~]# sed -i 's#root = "/var/lib/containerd"#root = "/data/containerd"#g' /etc/containerd/config.toml

# 强烈建议修改后，cat /etc/containerd/config.toml，查看修改结果是否正确
```

> **04-启动 containerd 并设置开机自启**

```text
[root@k8s-master-0 ~]# systemctl daemon-reload
[root@k8s-master-0 ~]# systemctl enable --now containerd
Created symlink from /etc/systemd/system/multi-user.target.wants/containerd.service to /usr/lib/systemd/system/containerd.service.
```

> **05-验证 containerd**

```text
# 查看服务状态
[root@k8s-master-0 ~]# systemctl status containerd
● containerd.service - containerd container runtime
   Loaded: loaded (/usr/lib/systemd/system/containerd.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2022-05-17 16:09:43 CST; 1min 21s ago
     Docs: https://containerd.io
 Main PID: 17217 (containerd)
   CGroup: /system.slice/containerd.service
           └─17217 /usr/bin/containerd

May 17 16:09:43 k8s-master-0 containerd[17217]: time="2022-05-17T16:09:43.155226321+08:00" level=error msg=...fig"
May 17 16:09:43 k8s-master-0 containerd[17217]: time="2022-05-17T16:09:43.155410134+08:00" level=info msg="...ent"
May 17 16:09:43 k8s-master-0 containerd[17217]: time="2022-05-17T16:09:43.155526261+08:00" level=info msg="...ate"
May 17 16:09:43 k8s-master-0 containerd[17217]: time="2022-05-17T16:09:43.155628084+08:00" level=info msg=s...trpc
May 17 16:09:43 k8s-master-0 containerd[17217]: time="2022-05-17T16:09:43.155691927+08:00" level=info msg="...tor"
May 17 16:09:43 k8s-master-0 containerd[17217]: time="2022-05-17T16:09:43.155723856+08:00" level=info msg=s...sock
May 17 16:09:43 k8s-master-0 containerd[17217]: time="2022-05-17T16:09:43.155737042+08:00" level=info msg="...cer"
May 17 16:09:43 k8s-master-0 containerd[17217]: time="2022-05-17T16:09:43.155809298+08:00" level=info msg="...ult"
May 17 16:09:43 k8s-master-0 containerd[17217]: time="2022-05-17T16:09:43.155837948+08:00" level=info msg="...ver"
May 17 16:09:43 k8s-master-0 containerd[17217]: time="2022-05-17T16:09:43.155867943+08:00" level=info msg="...57s"
Hint: Some lines were ellipsized, use -l to show in full.

# 查看进程和端口
[root@k8s-master-0 ~]# ps -ef | grep containerd
root     17217     1  0 16:09 ?        00:00:00 /usr/bin/containerd
root     17264 10585  0 16:13 pts/0    00:00:00 grep --color=auto containerd

[root@k8s-master-0 ~]# ss -ntlup | grep containerd
tcp    LISTEN     0      128    127.0.0.1:34412                 *:*                   users:(("containerd",pid=17217,fd=13))

# 查看存储路径的内容
[root@k8s-master-0 ~]# ls /data/containerd/
io.containerd.content.v1.content  io.containerd.runtime.v2.task        io.containerd.snapshotter.v1.overlayfs
io.containerd.metadata.v1.bolt    io.containerd.snapshotter.v1.btrfs   tmpmounts
io.containerd.runtime.v1.linux    io.containerd.snapshotter.v1.native
```

### **4.3. 安装 kubeadm, kubelet, kubectl**

> **01-配置软件源**

```text
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

> **02-安装软件包并设置开机自启**

```text
# 由于1.24刚发布，官方只有一个版本的包，有多个版本的时候，可以查询后指定版本安装
# yum list kubelet --showduplicate
# yum install kubelet-1.24.0-0
# ps: 由于官网未开放同步方式, 可能会有索引gpg检查失败的情况, 这时请用 yum install -y --nogpgcheck kubelet kubeadm kubectl 安装

[root@k8s-master-0 ~]# yum install kubelet kubeadm kubectl --nogpgcheck -y
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
Resolving Dependencies
--> Running transaction check
---> Package kubeadm.x86_64 0:1.24.0-0 will be installed
--> Processing Dependency: kubernetes-cni >= 0.8.6 for package: kubeadm-1.24.0-0.x86_64
--> Processing Dependency: cri-tools >= 1.19.0 for package: kubeadm-1.24.0-0.x86_64
---> Package kubectl.x86_64 0:1.24.0-0 will be installed
---> Package kubelet.x86_64 0:1.24.0-0 will be installed
--> Processing Dependency: socat for package: kubelet-1.24.0-0.x86_64
--> Processing Dependency: conntrack for package: kubelet-1.24.0-0.x86_64
--> Running transaction check
---> Package conntrack-tools.x86_64 0:1.4.4-7.el7 will be installed
--> Processing Dependency: libnetfilter_cttimeout.so.1(LIBNETFILTER_CTTIMEOUT_1.1)(64bit) for package: conntrack-tools-1.4.4-7.el7.x86_64
--> Processing Dependency: libnetfilter_cttimeout.so.1(LIBNETFILTER_CTTIMEOUT_1.0)(64bit) for package: conntrack-tools-1.4.4-7.el7.x86_64
--> Processing Dependency: libnetfilter_cthelper.so.0(LIBNETFILTER_CTHELPER_1.0)(64bit) for package: conntrack-tools-1.4.4-7.el7.x86_64
--> Processing Dependency: libnetfilter_queue.so.1()(64bit) for package: conntrack-tools-1.4.4-7.el7.x86_64
--> Processing Dependency: libnetfilter_cttimeout.so.1()(64bit) for package: conntrack-tools-1.4.4-7.el7.x86_64
--> Processing Dependency: libnetfilter_cthelper.so.0()(64bit) for package: conntrack-tools-1.4.4-7.el7.x86_64
---> Package cri-tools.x86_64 0:1.23.0-0 will be installed
---> Package kubernetes-cni.x86_64 0:0.8.7-0 will be installed
---> Package socat.x86_64 0:1.7.3.2-2.el7 will be installed
--> Running transaction check
---> Package libnetfilter_cthelper.x86_64 0:1.0.0-11.el7 will be installed
---> Package libnetfilter_cttimeout.x86_64 0:1.0.0-7.el7 will be installed
---> Package libnetfilter_queue.x86_64 0:1.0.2-2.el7_2 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==================================================================================================================
 Package                             Arch                Version                    Repository               Size
==================================================================================================================
Installing:
 kubeadm                             x86_64              1.24.0-0                   kubernetes              9.5 M
 kubectl                             x86_64              1.24.0-0                   kubernetes              9.9 M
 kubelet                             x86_64              1.24.0-0                   kubernetes               20 M
Installing for dependencies:
 conntrack-tools                     x86_64              1.4.4-7.el7                base                    187 k
 cri-tools                           x86_64              1.23.0-0                   kubernetes              7.1 M
 kubernetes-cni                      x86_64              0.8.7-0                    kubernetes               19 M
 libnetfilter_cthelper               x86_64              1.0.0-11.el7               base                     18 k
 libnetfilter_cttimeout              x86_64              1.0.0-7.el7                base                     18 k
 libnetfilter_queue                  x86_64              1.0.2-2.el7_2              base                     23 k
 socat                               x86_64              1.7.3.2-2.el7              base                    290 k

Transaction Summary
==================================================================================================================
Install  3 Packages (+7 Dependent packages)

Total download size: 66 M
Installed size: 288 M
Downloading packages:
(1/10): conntrack-tools-1.4.4-7.el7.x86_64.rpm                                             | 187 kB  00:00:00     
(2/10): 4d300a7655f56307d35f127d99dc192b6aa4997f322234e754f16aaa60fd8906-cri-tools-1.23.0- | 7.1 MB  00:00:06     
(3/10): dda11ee75bc7fcb01e32512cefb8f686dc6a7383516b8b0828adb33761fe602e-kubeadm-1.24.0-0. | 9.5 MB  00:00:08     
(4/10): 0c7a02e05273d05ea82ca13546853b65fbc257dd159565ce6eb658a0bdf31c9f-kubectl-1.24.0-0. | 9.9 MB  00:00:08     
(5/10): libnetfilter_cthelper-1.0.0-11.el7.x86_64.rpm                                      |  18 kB  00:00:00     
(6/10): socat-1.7.3.2-2.el7.x86_64.rpm                                                     | 290 kB  00:00:00     
(7/10): libnetfilter_queue-1.0.2-2.el7_2.x86_64.rpm                                        |  23 kB  00:00:00     
(8/10): libnetfilter_cttimeout-1.0.0-7.el7.x86_64.rpm                                      |  18 kB  00:00:00     
(9/10): 363f3fbfa8b89bb978e2d089e52ba59847f143834f8ea1b559afa864d8c5c011-kubelet-1.24.0-0. |  20 MB  00:00:17     
(10/10): db7cb5cb0b3f6875f54d10f02e625573988e3e91fd4fc5eef0b1876bb18604ad-kubernetes-cni-0 |  19 MB  00:00:16     
------------------------------------------------------------------------------------------------------------------
Total                                                                             2.1 MB/s |  66 MB  00:00:31     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : libnetfilter_cthelper-1.0.0-11.el7.x86_64                                                     1/10 
  Installing : socat-1.7.3.2-2.el7.x86_64                                                                    2/10 
  Installing : libnetfilter_cttimeout-1.0.0-7.el7.x86_64                                                     3/10 
  Installing : cri-tools-1.23.0-0.x86_64                                                                     4/10 
  Installing : kubectl-1.24.0-0.x86_64                                                                       5/10 
  Installing : libnetfilter_queue-1.0.2-2.el7_2.x86_64                                                       6/10 
  Installing : conntrack-tools-1.4.4-7.el7.x86_64                                                            7/10 
  Installing : kubernetes-cni-0.8.7-0.x86_64                                                                 8/10 
  Installing : kubelet-1.24.0-0.x86_64                                                                       9/10 
  Installing : kubeadm-1.24.0-0.x86_64                                                                      10/10 
  Verifying  : conntrack-tools-1.4.4-7.el7.x86_64                                                            1/10 
  Verifying  : kubernetes-cni-0.8.7-0.x86_64                                                                 2/10 
  Verifying  : kubeadm-1.24.0-0.x86_64                                                                       3/10 
  Verifying  : libnetfilter_queue-1.0.2-2.el7_2.x86_64                                                       4/10 
  Verifying  : kubectl-1.24.0-0.x86_64                                                                       5/10 
  Verifying  : cri-tools-1.23.0-0.x86_64                                                                     6/10 
  Verifying  : libnetfilter_cttimeout-1.0.0-7.el7.x86_64                                                     7/10 
  Verifying  : socat-1.7.3.2-2.el7.x86_64                                                                    8/10 
  Verifying  : libnetfilter_cthelper-1.0.0-11.el7.x86_64                                                     9/10 
  Verifying  : kubelet-1.24.0-0.x86_64                                                                      10/10 

Installed:
  kubeadm.x86_64 0:1.24.0-0            kubectl.x86_64 0:1.24.0-0            kubelet.x86_64 0:1.24.0-0           

Dependency Installed:
  conntrack-tools.x86_64 0:1.4.4-7.el7                    cri-tools.x86_64 0:1.23.0-0                            
  kubernetes-cni.x86_64 0:0.8.7-0                         libnetfilter_cthelper.x86_64 0:1.0.0-11.el7            
  libnetfilter_cttimeout.x86_64 0:1.0.0-7.el7             libnetfilter_queue.x86_64 0:1.0.2-2.el7_2              
  socat.x86_64 0:1.7.3.2-2.el7                           

Complete!
```

> **03-启动 kubelet 服务并设置开机自启**

```text
[root@k8s-master-0 ~]# systemctl enable --now kubelet
Created symlink from /etc/systemd/system/multi-user.target.wants/kubelet.service to /usr/lib/systemd/system/kubelet.service.
```

### **4.4 安装 Container Runtime Interface (CRI) CLI-crictl**

> **01-安装 crictl**

**该配置为可选项，仅供参考，二进制安装受限于网络环境有可能安装不成功，建议使用上面安装 kubernetes 时安装的 cri-tools**

```text
VERSION="v1.23.0"
curl -L https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-${VERSION}-linux-amd64.tar.gz --output crictl-${VERSION}-linux-amd64.tar.gz
tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-amd64.tar.gz
```

> **02-配置 crictl**

```text
# 配置 runtime-endpoint
[root@k8s-master-0 ~]# crictl config --set runtime-endpoint=unix:///run/containerd/containerd.sock 

# 如果不配置使用默认值，会有下面的报错
[root@k8s-master-0 ~]# crictl images
WARN[0000] image connect using default endpoints: [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead. 
ERRO[0002] connect endpoint 'unix:///var/run/dockershim.sock', make sure you are running as root and the endpoint has been started: context deadline exceeded 
IMAGE               TAG                 IMAGE ID            SIZE
```

> **03-验证测试**

```text
[root@k8s-master-0 ~]# crictl images
IMAGE               TAG                 IMAGE ID            SIZE

[root@k8s-master-0 ~]# crictl pods
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME
```

### **4.5. 命令脚本汇总**

**初始化节点时，可以直接复制执行**

```text
#!/bin/bash
# k8s-node-common.sh

# init
swapoff -a
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system

# containerd
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install containerd.io -y
cp /etc/containerd/config.toml /etc/containerd/config.toml.ori
containerd  config default > /etc/containerd/config.toml
sed -i 's#k8s.gcr.io#registry.aliyuncs.com/google_containers#g' /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sed -i 's#root = "/var/lib/containerd"#root = "/data/containerd"#g' /etc/containerd/config.toml
systemctl daemon-reload
systemctl enable --now containerd

# kubernetes
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
yum install kubelet kubeadm kubectl --nogpgcheck -y
systemctl enable --now kubelet


# Verify
crictl config --set runtime-endpoint=unix:///run/containerd/containerd.sock
crictl images
crictl pods
systemctl status kubelet
```

------

## **5. k8s 单控集群部署**

### **5.1. 采用配置文件初始化集群**

**可选参考，本文不执行，本文采用命令行直接配置参数执行的方式**

```text
[root@k8s-master-0 ~]# kubeadm config print init-defaults > kubeadmn-config.yaml
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 1.2.3.4
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  name: node
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: 1.24.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}

# 可以使用上面的命令生成一个配置文件，编辑修改后，再执行下面的命令初始化集群
kubeadm init --config=kubeadm-config.yaml --upload-certs | tee kubeadm-init.log
```

### **5.2. 命令行直接配置初始化集群**

> **01-初始化集群**

```text
# --pod-network-cidr 192.168.0.0/16 使用Calico网络插件的时候需要这么配置，如果跟现有网络冲突，请自行修改。
# --image-repository registry.aliyuncs.com/google_containers 受限于网络原因，指定image的仓库地址。
# 也可以提前将需要的image使用kubeadm config images pull，下载回来。

[root@k8s-master-0 ~]# kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version v1.24.0 --image-repository registry.aliyuncs.com/google_containers
[init] Using Kubernetes version: v1.24.0
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master-0 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.9.61]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master-0 localhost] and IPs [192.168.9.61 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master-0 localhost] and IPs [192.168.9.61 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 12.504486 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s-master-0 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node k8s-master-0 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: ueiemf.wfjtb2fmaju9dfmb
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.9.61:6443 --token hha3hi.v6g4ggfop3a27nv1 \
        --discovery-token-ca-cert-hash sha256:99a69b81a2bc48332ea9e4a1df3645f101a7bbcd0f3a76b9f0a8cba2bb1486fb
```

> **02-查看安装后的底层变化**

```text
[root@k8s-master-0 ~]# crictl images
IMAGE                                                             TAG                 IMAGE ID            SIZE
registry.aliyuncs.com/google_containers/coredns                   v1.8.6              a4ca41631cc7a       13.6MB
registry.aliyuncs.com/google_containers/etcd                      3.5.3-0             aebe758cef4cd       102MB
registry.aliyuncs.com/google_containers/kube-apiserver            v1.24.0             529072250ccc6       33.8MB
registry.aliyuncs.com/google_containers/kube-controller-manager   v1.24.0             88784fb4ac2f6       31MB
registry.aliyuncs.com/google_containers/kube-proxy                v1.24.0             77b49675beae1       39.5MB
registry.aliyuncs.com/google_containers/kube-scheduler            v1.24.0             e3ed7dee73e93       15.5MB
registry.aliyuncs.com/google_containers/pause                     3.6                 6270bb605e12e       302kB
registry.aliyuncs.com/google_containers/pause                     3.7                 221177c6082a8       311kB
[root@k8s-master-0 ~]# crictl pods
POD ID              CREATED             STATE               NAME                                   NAMESPACE           ATTEMPT             RUNTIME
77c4482fa7d43       28 seconds ago      Ready               kube-proxy-k59zz                       kube-system         0                   (default)
4ece96938cec8       55 seconds ago      Ready               etcd-k8s-master-0                      kube-system         0                   (default)
fc80ca83de6bf       55 seconds ago      Ready               kube-controller-manager-k8s-master-0   kube-system         0                   (default)
a83747a21aa47       55 seconds ago      Ready               kube-scheduler-k8s-master-0            kube-system         0                   (default)
0746ddf45df26       55 seconds ago      Ready               kube-apiserver-k8s-master-0            kube-system         0                   (default)
```

### **5.3. kubectl 管理环境配置**

> **01-配置管理 config**

```text
[root@k8s-master-0 ~]# mkdir -p $HOME/.kube
[root@k8s-master-0 ~]# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@k8s-master-0 ~]# chown $(id -u):$(id -g) $HOME/.kube/config
```

> **02-配置 kubectl Bash 自动补齐**

```text
[root@k8s-master-0 ~]# yum install bash-completion -y
[root@k8s-master-0 ~]# echo 'source <(kubectl completion bash)' >>~/.bashrc
[root@k8s-master-0 ~]# echo 'alias k=kubectl' >>~/.bashrc
[root@k8s-master-0 ~]# echo 'complete -F __start_kubectl k' >>~/.bashrc
```

> **03-验证 kubectl**

```text
[root@k8s-master-0 ~]# source ~/.bashrc 
[root@k8s-master-0 ~]# k get nodes
NAME           STATUS     ROLES           AGE   VERSION
k8s-master-0   NotReady   control-plane   15m   v1.24.0
```

### **5.4. 添加 Pod 网络插件-Calico**

> **01-安装 Tigera Calico Operator**

```text
[root@k8s-master-0 ~]# kubectl create -f https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml
namespace/tigera-operator created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/apiservers.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/imagesets.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/installations.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/tigerastatuses.operator.tigera.io created
Warning: policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
podsecuritypolicy.policy/tigera-operator created
serviceaccount/tigera-operator created
clusterrole.rbac.authorization.k8s.io/tigera-operator created
clusterrolebinding.rbac.authorization.k8s.io/tigera-operator created
deployment.apps/tigera-operator created
```

> **02-通过创建必要的自定义资源安装 Calico**

```text
[root@k8s-master-0 ~]# kubectl create -f https://projectcalico.docs.tigera.io/manifests/custom-resources.yaml
installation.operator.tigera.io/default created
apiserver.operator.tigera.io/default created
```

**官方默认 custom-resources.yaml 参考 , 注意 IP 地址范围，如果跟现有环境网络冲突，需要修改配置文件更换地址**

```text
# This section includes base Calico installation configuration.
# For more information, see: https://projectcalico.docs.tigera.io/v3.23/reference/installation/api#operator.tigera.io/v1.Installation
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    # Note: The ipPools section cannot be modified post-install.
    ipPools:
    - blockSize: 26
      cidr: 192.168.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()

---

# This section configures the Calico API server.
# For more information, see: https://projectcalico.docs.tigera.io/v3.23/reference/installation/api#operator.tigera.io/v1.APIServer
apiVersion: operator.tigera.io/v1
kind: APIServer 
metadata: 
  name: default 
spec: {}
```

> **03-观察 Pods 创建过程 (默认会失败)**

```text
[root@k8s-master-0 ~]# watch kubectl get pods -n calico-system

Every 2.0s: kubectl get pods -n calico-system                                                                      Wed May 18 09:40:53 2022

NAME                                       READY   STATUS              RESTARTS   AGE
calico-kube-controllers-68884f975d-9x48h   0/1     Pending             0          4s
calico-node-fn4w9                          0/1     Init:0/2            0          4s
calico-typha-86b5ddf878-mtd2v              0/1     ContainerCreating   0          4s
```

> **04-删除 master 节点的 Taint，保证 calico 相关 pod 能成功创建**

```text
[root@k8s-master-0 ~]# kubectl taint nodes $(hostname) node-role.kubernetes.io/master:NoSchedule-
[root@k8s-master-0 ~]# kubectl taint nodes $(hostname) node-role.kubernetes.io/control-plane:NoSchedule-
```

> **05-再次观察 Pods 创建过程**

```text
[root@k8s-master-0 ~]# watch kubectl get pods -n calico-system

Every 2.0s: kubectl get pods -n calico-system                                                                      Wed May 18 10:03:30 2022

NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-68884f975d-9x48h   1/1     Running   0          22m
calico-node-fn4w9                          1/1     Running   0          22m
calico-typha-86b5ddf878-mtd2v              1/1     Running   0          22m
```

> **06-确认 node 状态**

```text
[root@k8s-master-0 ~]# kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
k8s-master-0   Ready    control-plane   27m   v1.24.0   192.168.9.61   <none>        CentOS Linux 7 (Core)   3.10.0-1160.59.1.el7.x86_64   containerd://1.6.4
```

### **5.5. 添加 Worker 节点**

> **01-参考 <<K8S 节点通用配置 >>，配置 Worker 节点**

**配置过程略，请参考上文**

> **02-Join 到 k8s 集群**

```text
# Join 到集群
# 执行初始化集群时最后显示的 join 命令，如果忘记了可以执行 kubeadm token create --print-join-command --ttl=0 打印出命令

[root@k8s-master-1 ~]# kubeadm join 192.168.9.61:6443 --token hha3hi.v6g4ggfop3a27nv1 \
        --discovery-token-ca-cert-hash sha256:99a69b81a2bc48332ea9e4a1df3645f101a7bbcd0f3a76b9f0a8cba2bb1486fb

[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

# 查看 Join 到集群后，节点的状态
[root@k8s-master-1 ~]# crictl images
IMAGE                                                TAG                 IMAGE ID            SIZE
docker.io/calico/cni                                 v3.23.1             90d97aa939bbf       111MB
docker.io/calico/pod2daemon-flexvol                  v3.23.1             01dda8bd1b91e       8.67MB
registry.aliyuncs.com/google_containers/kube-proxy   v1.24.0             77b49675beae1       39.5MB
registry.aliyuncs.com/google_containers/pause        3.6                 6270bb605e12e       302kB

[root@k8s-master-1 ~]# crictl pods
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME
54112207923a3       3 minutes ago       Ready               calico-node-mj5db   calico-system       0                   (default)
58725b6e761da       3 minutes ago       Ready               kube-proxy-fvvjl    kube-system         0                   (default)
```

> **03-在 master 节点验证**

```text
[root@k8s-master-0 ~]# kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
k8s-master-0   Ready    control-plane   71m     v1.24.0   192.168.9.61   <none>        CentOS Linux 7 (Core)   3.10.0-1160.59.1.el7.x86_64   containerd://1.6.4
k8s-master-1   Ready    <none>          2m41s   v1.24.0   192.168.9.62   <none>        CentOS Linux 7 (Core)   3.10.0-1160.59.1.el7.x86_64   containerd://1.6.4
```

### **5.6. 测试验证**

> **01-创建测试资源**

```text
# 创建deployment
[root@k8s-master-0 ~]# kubectl create deployment nginx-test --image=nginx
deployment.apps/nginx-test created

# 创建NodePort类型的服务
[root@k8s-master-0 ~]# kubectl expose deployment nginx-test --port 80 --target-port=80  --type=NodePort --name=nginx-test-external
service/nginx-test-external exposed
```

> **02-验证创建结果**

```text
[root@k8s-master-0 ~]# kubectl get deployment -o wide
NAME         READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS   IMAGES   SELECTOR
nginx-test   1/1     1            1           2m5s   nginx        nginx    app=nginx-test

[root@k8s-master-0 ~]# kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP              NODE           NOMINATED NODE   READINESS GATES
nginx-test-847f5bc47c-n9dpw   1/1     Running   0          6s    192.168.196.2   k8s-master-1   <none>           <none>

[root@k8s-master-0 ~]# curl 192.168.196.2
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

[root@k8s-master-0 ~]# kubectl get svc -o wide
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE   SELECTOR
kubernetes            ClusterIP   10.96.0.1       <none>        443/TCP        91m   <none>
nginx-test-external   NodePort    10.105.107.17   <none>        80:31632/TCP   64s   app=nginx-test

# 测试集群内部和集群外部服务访问，显示结果是一致的。
[root@k8s-master-0 ~]# curl 10.105.107.17
[root@k8s-master-0 ~]# curl 192.168.9.62:31632
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### **5.7. 命令脚本汇总**

> **K8s-master.sh**

```text
#!/bin/bash
# K8s-master.sh

# kubeadm-init.sh
kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version v1.24.0 --image-repository registry.aliyuncs.com/google_containers

# kube config
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
yum install bash-completion -y
echo 'source <(kubectl completion bash)' >>~/.bashrc
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -F __start_kubectl k' >>~/.bashrc

# Calico
kubectl create -f https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml
kubectl create -f https://projectcalico.docs.tigera.io/manifests/custom-resources.yaml
kubectl taint nodes $(hostname) node-role.kubernetes.io/master:NoSchedule-
kubectl taint nodes $(hostname) node-role.kubernetes.io/control-plane:NoSchedule-

# 验证
kubectl get nodes -o wide
```

> **K8s-worker.sh**

```text
# 加入集群，下面的是示例，请用init时的实际语句替换
# kubeadm join 192.168.9.61:6443 --token hha3hi.v6g4ggfop3a27nv1 \
        --discovery-token-ca-cert-hash sha256:99a69b81a2bc48332ea9e4a1df3645f101a7bbcd0f3a76b9f0a8cba2bb1486fb
```

------

## **6. K8S 高可用集群部署**

### **6.1. 高可用方案选型**

> **01-实验环境采用 Kubernetes 官方介绍的方案 [Kubeadm HA topology - stacked etcd](https://link.zhihu.com/?target=https%3A//kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/)**

![img](https://pic4.zhimg.com/80/v2-d0708fd3db62200a0d7e400a81386d17_720w.jpg)



> **02-Load Balancer 选择 [官方方案](https://link.zhihu.com/?target=https%3A//github.com/kubernetes/kubeadm/blob/main/docs/ha-considerations.md%23options-for-software-load-balancing) 中的 kube-vip**

### **6.2. 初始化集群节点**

**参考 <<k8s 服务器初始化 >> 和 << K8S 节点通用配置 >>，配置所有节点。**

### **6.3. kube-vip 安装配置 (静态 pod)**

**所有 master 节点 (control-plane) 配置**

> **01-生成 Manifest(ARP)**

```text
# INTERFACE=ens160 网卡名称需要根据实际环境替换
# KVVERSION=v0.4.4 kube-vip 版本
# VIP=192.168.9.60 VIP 需要根据实际环境替换
# alias kube-vip="ctr run --rm --net-host ghcr.io/kube-vip/kube-vip:$KVVERSION vip /kube-vip"

#KVVERSION=$(curl -sL https://api.github.com/repos/kube-vip/kube-vip/releases | jq -r ".[0].name")
[root@k8s-master-0 ~]# export KVVERSION=v0.4.4
[root@k8s-master-0 ~]# export VIP=192.168.9.60
[root@k8s-master-0 ~]# export INTERFACE=ens160
[root@k8s-master-0 ~]# ctr image pull docker.io/plndr/kube-vip:$KVVERSION
docker.io/plndr/kube-vip:v0.4.4:                                                  resolved       |++++++++++++++++++++++++++++++++++++++| 
index-sha256:f59745f64f8e02158bc5aa70fb24e123e3f3891ca9bb86f8c259076e41a81924:    exists         |++++++++++++++++++++++++++++++++++++++| 
manifest-sha256:6c06a1a6c917afb9cfbd5c3e8714c941037bcd0b461e7ef2f21203c9cd22b713: exists         |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:53dcb733c4018ed99b211857fdf656f8ab22a07ffbd6b5cc9974d95c200749a3:    done           |++++++++++++++++++++++++++++++++++++++| 
config-sha256:10a711aa9888db2d9173101453ad14aa0b9b4b256486620a6e3de90b1751365b:   exists         |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:1ff217c64e92386b9d908ccc8978aa472db7eb1b1ffa4cb7abc33eecf799210b:    exists         |++++++++++++++++++++++++++++++++++++++| 
elapsed: 9.8 s                                                                    total:  11.7 M (1.2 MiB/s)                                       
unpacking linux/amd64 sha256:f59745f64f8e02158bc5aa70fb24e123e3f3891ca9bb86f8c259076e41a81924...
done: 1.012189313s

[root@k8s-master-0 ~]# alias kube-vip="ctr run --rm --net-host docker.io/plndr/kube-vip:$KVVERSION vip /kube-vip"

# 在终端命令行执行
kube-vip manifest pod \
    --interface $INTERFACE \
    --vip $VIP \
    --controlplane \
    --services \
    --arp \
    --leaderElection | tee /etc/kubernetes/manifests/kube-vip.yaml


apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  name: kube-vip
  namespace: kube-system
spec:
  containers:
  - args:
    - manager
    env:
    - name: vip_arp
      value: "true"
    - name: port
      value: "6443"
    - name: vip_interface
      value: ens160
    - name: vip_cidr
      value: "32"
    - name: cp_enable
      value: "true"
    - name: cp_namespace
      value: kube-system
    - name: vip_ddns
      value: "false"
    - name: svc_enable
      value: "true"
    - name: vip_leaderelection
      value: "true"
    - name: vip_leaseduration
      value: "5"
    - name: vip_renewdeadline
      value: "3"
    - name: vip_retryperiod
      value: "1"
    - name: vip_address
      value: 192.168.9.60
    image: ghcr.io/kube-vip/kube-vip:v0.4.4
    imagePullPolicy: Always
    name: kube-vip
    resources: {}
    securityContext:
      capabilities:
        add:
        - NET_ADMIN
        - NET_RAW
    volumeMounts:
    - mountPath: /etc/kubernetes/admin.conf
      name: kubeconfig
  hostAliases:
  - hostnames:
    - kubernetes
    ip: 127.0.0.1
  hostNetwork: true
  volumes:
  - hostPath:
      path: /etc/kubernetes/admin.conf
    name: kubeconfig
status: {}
```

> **02-验证**

**开配置完静态 pod 并不会创建，等 k8s 集群 init 以后才会有类似下面的输出**

```text
# 查看ens160网卡配置
[root@k8s-master-0 ~]# ip add show ens160
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:85:67:89 brd ff:ff:ff:ff:ff:ff
    inet 192.168.9.61/24 brd 192.168.9.255 scope global noprefixroute ens160
       valid_lft forever preferred_lft forever
    inet 192.168.9.60/32 scope global ens160
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:fe85:6789/64 scope link 
       valid_lft forever preferred_lft forever

# 查看pod
[root@k8s-master-0 ~]# crictl pods | grep kube-vip
0158385d6d14a       3 minutes ago       Ready               kube-vip-k8s-master-0                  kube-system         0                   (default)
```

### **6.4. 初始化 k8s 集群**

**第一个 master 节点 (control-plane) 配置**

> **01-初始化集群**

```text
# --pod-network-cidr 192.168.0.0/16 使用Calico网络插件的时候需要这么配置，如果跟现有网络冲突，请自行修改。
# --image-repository 受限于网络原因，指定image的仓库地址。
# 也可以提前将需要的image使用kubeadm config images pull，下载回来。
# --control-plane-endpoint "192.168.9.60:6443" kube-vip配置的vip地址和6443端口

[root@k8s-master-0 ~]# kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version v1.24.0 --image-repository registry.aliyuncs.com/google_containers --control-plane-endpoint "192.168.9.60:6443" --upload-certs
[init] Using Kubernetes version: v1.24.0
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master-0 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.9.61 192.168.9.60]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master-0 localhost] and IPs [192.168.9.61 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master-0 localhost] and IPs [192.168.9.61 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 14.017737 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
b32dc2ad50700e4b6b55da7d07725bc439f28abe107a35b5d5acd215bace4224
[mark-control-plane] Marking the node k8s-master-0 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node k8s-master-0 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: b2oizh.k4xs41xkvn4vze4o
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 192.168.9.60:6443 --token b2oizh.k4xs41xkvn4vze4o \
        --discovery-token-ca-cert-hash sha256:66aa526cdfbfee2d3fe523d4bec97cf00fca644f55565e104ccc8b456cd2dba0 \
        --control-plane --certificate-key b32dc2ad50700e4b6b55da7d07725bc439f28abe107a35b5d5acd215bace4224

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.9.60:6443 --token b2oizh.k4xs41xkvn4vze4o \
        --discovery-token-ca-cert-hash sha256:66aa526cdfbfee2d3fe523d4bec97cf00fca644f55565e104ccc8b456cd2dba0 
[root@k8s-master-0 ~]# 
```

### **6.5. kubectl 管理环境配置**

**第一个 master 节点 (control-plane) 配置**

> **01-配置管理 config**

```text
[root@k8s-master-0 ~]# mkdir -p $HOME/.kube
[root@k8s-master-0 ~]# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@k8s-master-0 ~]# chown $(id -u):$(id -g) $HOME/.kube/config
```

> **02-配置 kubectl Bash 自动补齐**

```text
[root@k8s-master-0 ~]# yum install bash-completion -y
[root@k8s-master-0 ~]# echo 'source <(kubectl completion bash)' >>~/.bashrc
[root@k8s-master-0 ~]# echo 'alias k=kubectl' >>~/.bashrc
[root@k8s-master-0 ~]# echo 'complete -F __start_kubectl k' >>~/.bashrc
```

> **03-验证 kubectl**

```text
[root@k8s-master-0 ~]# source ~/.bashrc 
[root@k8s-master-0 ~]# k get nodes
NAME           STATUS     ROLES           AGE   VERSION
k8s-master-0   NotReady   control-plane   15m   v1.24.0
```

### **6.6. 添加 Pod 网络插件-Calico**

**第一个 master 节点 (control-plane) 配置**

> **01-安装 Tigera Calico Operator**

```text
[root@k8s-master-0 ~]# kubectl create -f https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml
```

> **02-通过创建必要的自定义资源安装 Calico**

```text
[root@k8s-master-0 ~]# kubectl create -f https://projectcalico.docs.tigera.io/manifests/custom-resources.yaml
installation.operator.tigera.io/default created
apiserver.operator.tigera.io/default created
```

**官方默认 custom-resources.yaml 参考 , 注意 IP 地址范围，如果跟现有环境网络冲突，需要修改配置文件更换地址**

> **03-删除 master 节点的 Taint，保证 calico 相关 pod 能成功创建。**

```text
[root@k8s-master-0 ~]# kubectl taint nodes $(hostname) node-role.kubernetes.io/master:NoSchedule-
[root@k8s-master-0 ~]# kubectl taint nodes $(hostname) node-role.kubernetes.io/control-plane:NoSchedule-
```

> **04-确认 node 状态**

```text
[root@k8s-master-0 ~]# kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
k8s-master-0   Ready    control-plane   27m   v1.24.0   192.168.9.61   <none>        CentOS Linux 7 (Core)   3.10.0-1160.59.1.el7.x86_64   containerd://1.6.4
```

### **6.7. 添加其他控制平面 (master) 节点**

> **01-参考 6.3 配置 kube-vip**
>
> **02-添加 control-plane node**

```text
# 以 k8s-master-1 演示
[root@k8s-master-1 ~]# kubeadm join 192.168.9.60:6443 --token b2oizh.k4xs41xkvn4vze4o \
        --discovery-token-ca-cert-hash sha256:66aa526cdfbfee2d3fe523d4bec97cf00fca644f55565e104ccc8b456cd2dba0 \
        --control-plane --certificate-key 048fde800500bc200995e57f447e71a6f5aebf525f6b21c3d1e1be5cd7bd280b

[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks before initializing the new control plane instance
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[download-certs] Downloading the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master-2 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.9.62 192.168.9.60]
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master-2 localhost] and IPs [192.168.9.62 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master-2 localhost] and IPs [192.168.9.62 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[certs] Using the existing "sa" key
[kubeconfig] Generating kubeconfig files
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[check-etcd] Checking that the etcd cluster is healthy
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...
[etcd] Announced new etcd member joining to the existing etcd cluster
[etcd] Creating static Pod manifest for "etcd"
[etcd] Waiting for the new etcd member to join the cluster. This can take up to 40s
The 'update-status' phase is deprecated and will be removed in a future release. Currently it performs no operation
[mark-control-plane] Marking the node k8s-master-2 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node k8s-master-2 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule node-role.kubernetes.io/control-plane:NoSchedule]

This node has joined the cluster and a new control plane instance was created:

* Certificate signing request was sent to apiserver and approval was received.
* The Kubelet was informed of the new secure connection details.
* Control plane label and taint were applied to the new node.
* The Kubernetes control plane instances scaled up.
* A new etcd member was added to the local/stacked etcd cluster.

To start administering your cluster from this node, you need to run the following as a regular user:

        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config

Run 'kubectl get nodes' to see this node join the cluster.
```

> **03-certificates 过期的处理方法 (可选，非必需)**

**如果不是在初始化集群后立刻添加的其他 master 节点，certificates key 会过期，采用下面的命令获取新 key**

```text
[root@k8s-master-0 ~]# kubeadm init phase upload-certs --upload-certs
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
048fde800500bc200995e57f447e71a6f5aebf525f6b21c3d1e1be5cd7bd280b
```

### **6.8. 添加 Worker 节点**

> **01-添加 worker node**

```text
# 以 k8s-worker-0 演示
[root@k8s-worker-0 ~]# kubeadm join 192.168.9.60:6443 --token b2oizh.k4xs41xkvn4vze4o \
        --discovery-token-ca-cert-hash sha256:66aa526cdfbfee2d3fe523d4bec97cf00fca644f55565e104ccc8b456cd2dba0 

[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

### **6.9. 测试验证**

> **01-查看 nodes**

```text
[root@k8s-master-0 ~]# k get nodes
NAME           STATUS   ROLES           AGE     VERSION
k8s-master-0   Ready    control-plane   15h     v1.24.0
k8s-master-1   Ready    control-plane   14m     v1.24.0
k8s-master-2   Ready    control-plane   10m     v1.24.0
k8s-worker-0   Ready    <none>          3m22s   v1.24.0
```

> **02-创建测试资源**

```text
# 创建deployment
[root@k8s-master-0 ~]# kubectl create deployment nginx-test --image=nginx
deployment.apps/nginx-test created

# 创建NodePort类型的服务
[root@k8s-master-0 ~]# kubectl expose deployment nginx-test --port 80 --target-port=80  --type=NodePort --name=nginx-test-external
service/nginx-test-external exposed
```

> **03-验证创建结果**

```text
[root@k8s-master-0 ~]# kubectl get deployment -o wide
NAME         READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES   SELECTOR
nginx-test   0/1     1            0           14s   nginx        nginx    app=nginx-test

[root@k8s-master-0 ~]# kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP               NODE           NOMINATED NODE   READINESS GATES
nginx-test-847f5bc47c-9t77b   1/1     Running   0          34s   192.168.29.129   k8s-worker-0   <none>           <none>

[root@k8s-master-0 ~]# kubectl get svc -o wide
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE   SELECTOR
kubernetes            ClusterIP   10.96.0.1       <none>        443/TCP        15h   <none>
nginx-test-external   NodePort    10.97.202.183   <none>        80:32617/TCP   52s   app=nginx-test
```

> **04-查看 etcd 状态**

```text
[root@k8s-master-0 ~]# crictl exec `crictl ps | grep etcd | awk '{print $1}'` etcdctl --cert /etc/kubernetes/pki/etcd/peer.crt --key /etc/kubernetes/pki/etcd/peer.key --cacert /etc/kubernetes/pki/etcd/ca.crt --endpoints https://192.168.9.61:2379 endpoint health --cluster
https://192.168.9.61:2379 is healthy: successfully committed proposal: took = 27.310514ms
https://192.168.9.63:2379 is healthy: successfully committed proposal: took = 34.384575ms
https://192.168.9.62:2379 is healthy: successfully committed proposal: took = 35.74179ms
```

### **6.10. 命令脚本汇总**

> **K8s-master-first.sh**

```text
#!/bin/bash
# K8s-master-first.sh

# kube-vip
#KVVERSION=$(curl -sL https://api.github.com/repos/kube-vip/kube-vip/releases | jq -r ".[0].name")
export KVVERSION=v0.4.4
export VIP=192.168.9.60
export INTERFACE=ens160
ctr image pull docker.io/plndr/kube-vip:$KVVERSION
#alias kube-vip="ctr run --rm --net-host ghcr.io/kube-vip/kube-vip:$KVVERSION vip /kube-vip"
alias kube-vip="ctr run --rm --net-host docker.io/plndr/kube-vip:$KVVERSION vip /kube-vip"
kube-vip manifest pod \
    --interface $INTERFACE \
    --vip $VIP \
    --controlplane \
    --services \
    --arp \
    --leaderElection | tee /etc/kubernetes/manifests/kube-vip.yaml


# kubeadm init
kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version v1.24.0 --image-repository registry.aliyuncs.com/google_containers --control-plane-endpoint "192.168.9.60:6443" --upload-certs

# kube config
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
yum install bash-completion -y
echo 'source <(kubectl completion bash)' >>~/.bashrc
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -F __start_kubectl k' >>~/.bashrc

# Calico
kubectl create -f https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml
kubectl create -f https://projectcalico.docs.tigera.io/manifests/custom-resources.yaml
kubectl taint nodes $(hostname) node-role.kubernetes.io/master:NoSchedule-
kubectl taint nodes $(hostname) node-role.kubernetes.io/control-plane:NoSchedule-

# 验证
kubectl get nodes -o wide
```

> **K8s-master-other.sh**

```text
#!/bin/bash
# K8s-master-other.sh

# kube-vip
#KVVERSION=$(curl -sL https://api.github.com/repos/kube-vip/kube-vip/releases | jq -r ".[0].name")
export KVVERSION=v0.4.4
export VIP=192.168.9.60
export INTERFACE=ens160
ctr image pull docker.io/plndr/kube-vip:$KVVERSION
#alias kube-vip="ctr run --rm --net-host ghcr.io/kube-vip/kube-vip:$KVVERSION vip /kube-vip"
alias kube-vip="ctr run --rm --net-host docker.io/plndr/kube-vip:$KVVERSION vip /kube-vip"
kube-vip manifest pod \
    --interface $INTERFACE \
    --vip $VIP \
    --controlplane \
    --services \
    --arp \
    --leaderElection | tee /etc/kubernetes/manifests/kube-vip.yaml

# 加入集群，下面的是示例，请用init时的实际语句替换
#kubeadm join 192.168.9.60:6443 --token fghmf1.zl29bwrlg8brcop8 \
#        --discovery-token-ca-cert-hash sha256:045747aa25253136f686daa6fca94985a4482ca39f568abbe36ee0dec55c48a7 \
#        --control-plane --certificate-key 35f6367feafc6755a7ab72b8a2f03321ac2545937d45ea84650de3a8835e341e
```

> **K8s-worker.sh**

```text
# 加入集群，下面的是示例，请用init时的实际语句替换
#kubeadm join 192.168.9.60:6443 --token b2oizh.k4xs41xkvn4vze4o \
#        --discovery-token-ca-cert-hash sha256:66aa526cdfbfee2d3fe523d4bec97cf00fca644f55565e104ccc8b456cd2dba0 
```

------

## **7. 总结**

本文详细展示了 Kubernetes1.24 使用 Containerd 部署单 Master 和三 Master 集群的部署过程，仅供读者在学习测试环境使用。

------

> **参考文档**

- [https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/](https://link.zhihu.com/?target=https%3A//kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/)
- **[Getting started / Production environment / Container runtimes](https://link.zhihu.com/?target=https%3A//kubernetes.io/docs/setup/production-environment/container-runtimes/)**
- **[Getting started / Production environment / Installing Kubernetes with deployment tools](https://link.zhihu.com/?target=https%3A//kubernetes.io/docs/setup/production-environment/tools/)**
- [https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md](https://link.zhihu.com/?target=https%3A//github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md)
- [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/](https://link.zhihu.com/?target=https%3A//kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
- [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/](https://link.zhihu.com/?target=https%3A//kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/)
- [https://kube-vip.chipzoller.dev/docs/installation/static/](https://link.zhihu.com/?target=https%3A//kube-vip.chipzoller.dev/docs/installation/static/)

> **Get 文档**

- Github [https://github.com/devops/z-notes](https://link.zhihu.com/?target=https%3A//github.com/devops/z-notes)
- Gitee [https://gitee.com/zdevops/z-notes](https://link.zhihu.com/?target=https%3A//gitee.com/zdevops/z-notes)

> **Get 代码**

- Github [https://github.com/devops/ansible-zdevops](https://link.zhihu.com/?target=https%3A//github.com/devops/ansible-zdevops)
- Gitee [https://gitee.com/zdevops/ansible-zdevops](https://link.zhihu.com/?target=https%3A//gitee.com/zdevops/ansible-zdevops)

> **B 站**

- **[老 Z 手记](https://link.zhihu.com/?target=https%3A//space.bilibili.com/1039301316)**[https://space.bilibili.com/1039301316](https://link.zhihu.com/?target=https%3A//space.bilibili.com/1039301316)

> **版权声明**

- 所有内容均属于原创，整理不易，感谢收藏，转载请标明出处。

> **About Me**

- 昵称：老 Z
- 坐标：山东济南
- 职业：运维架构师 / 高级运维工程师 =**运维**
- 微信：zdevops
- 关注的领域：云计算 / 云原生技术运维，自动化运维
- 技能标签：OpenStack、Ansible、K8S、Python、Go、CNCF