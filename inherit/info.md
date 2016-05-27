以下是保留的是网址:

1. [孔浩学习空间](http://pan.baidu.com/s/1eQILyKi)    
    提取码：5ode

2. 安装mysql 5.7
    [mysql-yum](http://blog.csdn.net/horace20/article/details/26516689)
    在安装完后会生成一个默认密码，在以下文件中
    grep 'temporary password' /var/log/mysqld.log

    SET PASSWORD = PASSWORD('123456');
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'Carl10086(ysz)' WITH GRANT OPTION ;
    FLUSH PRIVILEGES;
    CREATE DATABASE `carl` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;


3. 如何写一份好的简历：

    [简历模板](https://github.com/geekcompany/ResumeSample)
    [如何写一份好的简历](http://blog.devtang.com/2013/12/22/how-to-write-resume-for-it-company/)
