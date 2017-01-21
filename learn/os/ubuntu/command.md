##新建用户:
*   在Ubuntu13.10下创建一个新的用户：
*   Step1：添加新用户
*   useradd -r -m -s /bin/bash xifei
*   Step2:配置新用户密码
*   passwd 用户名
*   Step3：给新添加的用户增加ROOT权限
visudo/etc/sudoers
然后添加：
xifei ALL=(ALL:ALL) ALL

另外，如果直接用useradd添加用户的话，可能出现没有home下的文件夹，以及shell无法自动补全的显现。出现此问题只要修改/etc/passwd下的/bin/sh为/bin/bash即可。


##ssh连接就断开
Just comment this line in file “/etc/pam.d/sshd”:
注释
session required        pam_loginuid.so
www.jb51.net/article/47764.htm

##批量刪除spark work日志

find /log_data/spark_work_data/ -name 'app-2015*'  | xargs rm -rf

find /log_data/spark_work_data/ -name 'app-201601*'  | xargs rm -rf

find /log_data/spark_work_data/ -name 'app-201602*'  | xargs rm -rf

find /log_data/spark_work_data/ -name 'app-201603*'  | xargs rm -rf



##防火墙

vim /etc/sysconfig/iptables

-A INPUT -p tcp -m state --state NEW -m tcp --dport 9160 -j ACCEPT