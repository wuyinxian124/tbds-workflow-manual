# cdh 环境手动安装

cdh 手动安装runner过程  

## 
<font color=#DC143C>
__务必选择一个没有安装httpd 的节点__
</font>   
##

### 1. 添加yum 源

1. touch TBDS.repo（/etc/yum.repos.d）
2. 添加如下内容：  
>     [TBDS]  
>     name=TBDS  
>     baseurl=http://10.151.136.14/tbds-mirror-multicluster  
>     path=/  
>     enabled=1    
>     gpgcheck=0  
    
  
### 2. 安装lhotse runner
1. yum clean all
2. yum install -y lhotse-runners
3. 安装确认   
 确认目录/usr/local/lhotse_runners 存在

### 3. 安装jdk1.8
1. yum install -y jdk1.8.0_111  


### 4. 安装httpd
1. yum install -y httpd
2. 修改httpd 配置 (/etc/httpd/conf/httpd.conf)   
  注释  Listen 80   
3. 新建配置文件  /etc/httpd/conf.d/runner.conf  
添加如下内容：
>
Listen 56986  
ScriptAlias /runner/ /usr/local/lhotse_runners/getlog/  
<Directory "/usr/local/lhotse_runners/getlog/">  
 AllowOverride None  
 Require all granted  
 Options +ExecCGI  
 Order allow,deny  
 Allow from all  
<//Directory>  **<font color=#DC143C>这里多了一个反斜线</font>**

_  
4. 修改httpd 相关目录权限  
  chown -R hdfs:hadoop /etc/httpd/logs/;chown -R hdfs:hadoop /usr/sbin/httpd;chown -R hdfs:hadoop /run/httpd  
5. 启动httpd  
  su hdfs -c "/usr/sbin/httpd"

### 4. 修改runner 配置
相关内容可以参考 控制集群 runner 节点  

**a. 修改lhotse_base.properties 文件**  

___
  base_ip = 控制集群 base hostname  
  base_port = lhotse.base.port  
  ftp_user=hdfs  
  ftp_password=ftp@Tbds.com  
  ftp_path=/  
  doAndRedo_url=http://lhotse-yh-webCgi.tencent-distribute.com:8081/LService/RedoAndDoState  
  type.list=37,38,39,66,67,68,69,70,72,75,92,98,99,100,104,105,118,119,120  （默认就行，除非有自定义任务）
  discovery.service.zk.conn.str=服务发现 zk 地址  
  discovery.cluster.name=tdw    
___  

demo 文件内容 参考   
___
>  base_ip = tbds-10-254-99-17  
>  base_port = 9930  
>  ftp_user=hdfs  
>  ftp_password=ftp@Tbds.com  
>  ftp_path=/  
>  doAndRedo_url=http://lhotse-yh-webCgi.tencent-distribute.com:8081/LService/RedoAndDoState  
>  type.list=37,38,39,66,67,68,69,70,72,75,92,98,99,100,104,105,118,119,120  
>  discovery.service.zk.conn.str=tbds-10-151-135-217:2181,tbds-10-151-141-151:2181,tbds-10-151-141-152:2181  
>  discovery.cluster.name=tdw    
___  


**b. 添加新的配置文件 discovery.properties**

discovery.service.zk.conn.str 为服务发现的 zk 地址。
___
> discovery.service.client.fixedtime.sync.interval.millis=60000
discovery.exception.feedback.blacklist.policy.checktime.seconds=60
discovery.exception.feedback.blacklist.policy.checkcount.threshoud=3
discovery.service.zk.conn.timeout.millis=1200000
discovery.service.zk.session.timeout.millis=600000
discovery.service.zk.conn.str=tbds-10-151-135-217:2181,tbds-10-151-141-151:2181,tbds-10-151-141-152:2181
>  discovery.cluster.name=tdw
___

**c. 添加host name**  

编辑runner 节点的 /etc/hosts 文件,将 控制集群的 ip host 添加到改文件  
类似  
___
>  10.151.135.217 tbds-10-151-135-217  
10.254.99.17 tbds-10-254-99-17  
10.254.99.27 tbds-10-254-99-27  
10.151.141.151 tbds-10-151-141-151  
10.151.141.152 tbds-10-151-141-152  
>  10.151.135.224 tbds-10-151-135-224  
___
  
 
## 5. 启动runner 
a. 切到目录  
/usr/local/lhotse_runners

b. hdfs 用户执行  
./start_jar.sh