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
```
[TBDS]  
name=TBDS  
baseurl=http://10.151.136.14/tbds-mirror-multicluster  
path=/  
enabled=1    
gpgcheck=0  
```    
  
### 2. 安装lhotse runner
1. yum clean all
2. yum install -y lhotse-runners
3. 安装确认   
 确认目录/usr/local/lhotse_runners 存在

### 3. 安装jdk1.8
1. yum install -y jdk1.8.0_111  

### 4. 确认 pstree -p pid 命令可用

### 5. 安装httpd
1. yum install -y httpd
2. 修改httpd 配置 (/etc/httpd/conf/httpd.conf)   
  注释  Listen 80   
3. 新建配置文件  /etc/httpd/conf.d/runner.conf  
添加如下内容：
```
Listen 56986  
ScriptAlias /runner/ /usr/local/lhotse_runners/getlog/  
<Directory "/usr/local/lhotse_runners/getlog/">  
 AllowOverride None  
 Require all granted  
 Options +ExecCGI  
 Order allow,deny  
 Allow from all  
</Directory>  
```

4. 修改httpd 相关目录权限  
  chown -R hdfs:hadoop /etc/httpd/logs/;chown -R hdfs:hadoop /usr/sbin/httpd;chown -R hdfs:hadoop /run/httpd  
5. 启动httpd  
  su hdfs -c "/usr/sbin/httpd"

### 6. 修改runner 配置
相关内容可以参考 控制集群 runner 节点  

**a. 修改 lhotse_base.properties 文件**  
```
base_ip = 控制集群 base 节点hostname  
base_port =  高级配置 lhotse-base-env 配置的 lhotse.base.port属性值   
ftp_user=hdfs  
ftp_password=ftp@Tbds.com  
ftp_path=/  
doAndRedo_url=http://lhotse-yh-webCgi.tencent-distribute.com:8081/LService/RedoAndDoState  
type.list=37,38,39,66,67,68,69,70,72,75,92,98,99,100,104,105,118,119,120  
discovery.service.zk.conn.str=控制集群 zk 地址  
discovery.cluster.name=tdw
cluster_type=集群类型（目前支持tbds,zhx_cdh;tbds集群可以不配置该选项）    
```
__demo 文件内容 参考__   
```
base_ip = tbds-10-219-7-148  
base_port = 9930  
ftp_user=hdfs  
ftp_password=ftp@Tbds.com  
ftp_path=/  
doAndRedo_url=http://lhotse-yh-webCgi.tencent-distribute.com:8081/LService/RedoAndDoState  
type.list=37,38,39,66,67,68,69,70,72,75,92,98,99,100,104,105,118,119,120,123  
discovery.service.zk.conn.str=tbds-100-76-22-204:2181,tbds-10-254-96-17:2181,tbds-10-234-135-212:2181  
discovery.cluster.name=tdw  
cluster_type=zhx_cdh    
```
----------------------
**b. 添加新的配置文件 discover.properties**
```
discovery.service.client.fixedtime.sync.interval.millis=60000  
discovery.exception.feedback.blacklist.policy.checktime.seconds=60  
discovery.exception.feedback.blacklist.policy.checkcount.threshoud=3  
discovery.service.zk.conn.timeout.millis=1200000  
discovery.service.zk.session.timeout.millis=600000  
discovery.service.zk.conn.str=为服务发现的 zk 地址(控制集群 zk 地址)   
discovery.cluster.name= 通常 默认值都是tdw  
```
__demo 内容参考如下：__
```
>discovery.service.client.fixedtime.sync.interval.millis=60000  
discovery.exception.feedback.blacklist.policy.checktime.seconds=60  
discovery.exception.feedback.blacklist.policy.checkcount.threshoud=3  
discovery.service.zk.conn.timeout.millis=1200000  
discovery.service.zk.session.timeout.millis=600000  
discovery.service.zk.conn.str=tbds-100-76-22-204:2181,tbds-10-254-96-17:2181,tbds-10-234-135-212:2181  
discovery.cluster.name=tdw  
```
----------------------
**c. 添加host name**  

编辑runner 节点的 /etc/hosts 文件,将 控制集群的 ip host 添加到改文件  
类似  
```
10.151.135.217 tbds-10-151-135-217  
10.254.99.17 tbds-10-254-99-17  
10.254.99.27 tbds-10-254-99-27  
10.151.141.151 tbds-10-151-141-151  
10.151.141.152 tbds-10-151-141-152  
10.151.135.224 tbds-10-151-135-224  
```

### 7. 启动runner 
a. 切到目录  
/usr/local/lhotse_runners

b. hdfs 用户执行  
./start_jar.sh

c. 将hdfs 用户添加到sudo组  
echo "hdfs ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

### 8. 添加服务  
将新建的runner添加到集群配置中  

操作如下图
![新建服务](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/987474dbe5a61b20c944dc8daecb65bfd90f7ebb83ca311a339e61ad5b2ed793cc1d104bab9dab48d98af20b05f4622a?pictype=scale&from=30113&version=2.0.0.2&uin=821244074&fname=20170918221108.png&size=1024)

添加对应节点的runner
![添加runner](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/d57629254b68f4cefade48c75236d5aceb0d7b68760c55639bd80880015cc7b8abcc28d5c0b65e94fe74e2898b632b3d?pictype=scale&from=30113&version=2.0.0.2&uin=821244074&fname=20170918221620.png&size=1024)
