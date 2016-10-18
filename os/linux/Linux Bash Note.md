## 1. Linux 权限管理
### 1.1 普通用户 （Centos 6.5）
普通用户（非root）执行命令权限不够
> 用 sudo

仍然没有权限，需要添加 sudoers

```
查看 sudoers 文件权限
# ls -l /etc/sudoers
修改文件权限
# chmod 777 /etc/sudoers
# vim /etc/sudoers
修改文件，添加一行
## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL
shadow  ALL=(ALL)       ALL
保存后恢复权限
# chmod 440 /etc/sudoers
再去普通用户下尝试
```

### 1.2 更改用户密码
```
 $ passwd
OR:
 # passwd [user]
```
只有 root 用户可以修改别人的密码

### 1.3 四个概念
- owner, 文件拥有者
可以通过 ls -l 命令查看
- real user ID
实际进行的执行者
- effective user ID
用于检验进程执行时所获的访问权限
- saved set-user-ID

不太明白
[real user ID, effective user ID,saved set-user-ID 与 setuid()，seteuid()](http://www.groad.net/bbs/thread-3743-1-1.html)

### 1.4 chmod 命令
三种用户：
- 拥有者
- 组 （Group）
- 其他 （other）

权限：
- 一般权限, Read(4), Write(2), X(1), non(0)
- 特殊权限，变更文件或者目录
![Linux 文件的用户权限分析图](http://man.linuxde.net/wp-content/uploads/2013/11/chmod.gif)

### 1.5 systemd
Unix System V(System Five): one of the first commercial versions of the Unix OS. Abbreviated to SysV.

systemd is an improvement of SysV init system.