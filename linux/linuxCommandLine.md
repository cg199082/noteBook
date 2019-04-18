# linux常用命令

## 1.centos开机界面多个选项
centos更新后不会自动删除旧的内核，所以启动时就会出现多个内核选项，以下为删除方式：

* 查看当前版本
```
# uname -a
Linux localhost.localdomain 3.10.0-229.20.1.el7.x86_64 #1 SMP Tue Nov 3 19:10:07 UTC 201 GNU/Linux
```
* 查看系统中全部内核rpm包
```
# rpm -qa | grep kernel
kernel-3.10.0-229.14.1.el7.x86_64
kernel-3.10.0-229.el7.x86_64
abrt-addon-kerneloops-2.1.11-22.el7.centos.0.1.x86_64
kernel-tools-libs-3.10.0-229.20.1.el7.x86_64
kernel-3.10.0-229.20.1.el7.x86_64
kernel-tools-3.10.0-229.20.1.el7.x86_64
```
* 删除旧的内核rpm包
```
yum remove kernel-3.10.0-229.14.1.el7
```
* 重启系统
```
 # reboot 
 ```