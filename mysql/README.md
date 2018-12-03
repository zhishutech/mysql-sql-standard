# MySQL安装规范
该文档解释归：知数堂 http://zhishutang.com

## 系统安装规范

### bois中
1. 禁用系统的numa
2. 关闭节能模式& 关闭NUMA、C-states、C1E
3. CPU，内存处在高性能模式

### Raid控制器
1. 使用WB模式，可能选择force wb
2. 优先选择Raid10，次优先选择Raid5 
3. 关闭Raid卡预读机制

## 系统优化
1. 更改文件句柄和进程数
2. 内核优化 /etc/sysctl.conf  

    ```
    vm.swappiness <= 5（也可以设置为0）
    vm.dirty_ratio <= 20
    vm.dirty_background_ratio <= 10
    net.ipv4.tcp_max_syn_backlog = 819200
    net.core.netdev_max_backlog = 400000
    net.core.somaxconn = 4096
    net.ipv4.tcp_tw_reuse=1
    net.ipv4.tcp_tw_recycle=0
    ```
    
3. 禁用selinux ： /etc/sysconfig/selinux  更改SELINUX=disabled.
4. iptables如果不使用可以关闭。可是需要打开MySQL需要的端口号

## 文件系统优化
1. 推荐使用XFS文件系统
2. MySQL数据分区独立 ，例如挂载点为: /data
3. mount参数 defaults, noatime, nodiratime, nobarrier 如/etc/fstab：

    ```
    /dev/mapper/vg_data-lv_data /data                   xfs     defaults,noatime,nodiratime,nobarrier        1 2
    ```

4. io调度
   SAS      ： deadline
   
   SSD&PCI-E： noop


## MySQL版本选择
特别注意，不要使用yum，apt-get直接安装。 建议从官方下载指定的二进制版本安装。 
对于安装版本犹豫不决的，推荐加入QQ群：581702903 咨询。
现在推荐： MySQL-5.7.22 （2018.4月）

## MySQL安装路径
1.  基本准备 

    * 下载mysql
   
    ``` 
    cd /data
    wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.22-linux-glibc2.12-x86_64.tar.gz
    ```
  
    * MySQL软件放置 :

    ```  
    mkdir /opt/mysql
    cd /opt/mysql/
    tar zxvf /data/mysql/mysql-5.7.22-linux-glibc2.12-x86_64.tar.gz
    ```

    * basedir：  /usr/local/mysql 
     
    ```
    ln -s /opt/mysql/mysql-xxx /usr/local/mysql
    ```

2.  创建数据库相关目录：

    ```   
    mkdir /data/mysql/ -p
    mkdir /data/mysql/mysql3306/{data,logs,tmp} -p
    ```

3. 创建mysql用户
    
    ```   
    groupadd mysql
    useradd -g mysql -s /sbin/nologin -d /usr/local/mysql -MN mysql
    ```

4. 更改权限

    ```   
    chown -R mysql:mysql /data/mysql/mysql3306
    chown -R mysql:mysql /opt/mysql/
    chown -R mysql:mysql /usr/local/mysql
    ```
   
5. 配置文参考 /data/mysql/mysql3306/my3306.cnf 
   参考URL
   
6. 初始化
     
    ```  
    /usr/local/mysql/bin/mysqld --defaults-file=/data/mysql/mysql3306/my3306.cnf  --initialize
    ```
   
   确认没有错误提示，从error log中找到初始化的密码。
  
    ``` 
    grep "password" /data/mysql/mysql3306/data/error.log
    ```
   
7. 第一次启动MySQL

    ``` 
    /usr/local/mysql/bin/mysqld --defaults-file=/data/mysql/mysql3306/my3306.cnf &
    ```
 进入mysql更改密码。
  
    ```
    /usr/local/mysql/bin/mysql -S /tmp/mysql3306.sock -p 
    ```
  输入刚才查看到的密码。
  
    ```
    mysql>alter user user() identified by 'zstzst';
    ```
  
  *恭喜到此MySQL安装毕*

## MySQL启动及关闭
1. 启动mysql
   
    ```
    /usr/local/mysql/bin/mysqld --defaults-file=/data/mysql/mysql3306/my3306.cnf & 
    ```

2. 关闭mysql


* 利用mysqladmin关闭

```
mysqladmin -S /tmp/mysql3306.sock -p shutdown   
```
输入密码关闭

* 另一种在命令行关闭，在MySQL输入shutdown关闭，需要有shutdown权限

```
/usr/local/mysql/bin/mysql -S /tmp/mysql3306.sock -p 
    
mysql>shutdown;
```


