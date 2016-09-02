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

