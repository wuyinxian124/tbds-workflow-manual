**概要**

在tbds集群以外的节点安装runner服务，需要用到该文档做参考。

安装步骤包括：
1. 添加yum源
2. 添加hdfs 用户
3. 安装runner服务
4. 安装httpd服务
5. 修改httpd配置
6. 修改runner配置
7. 修改base设置（该步骤以下，只针对升级系统适用）
8. 重启base
9. 重启runner

### 一、添加yum 源
假设需要安装runner服务的节点是10.0.0.1
1. 进入10.0.0.1 节点，并切到root 用户
2. 编辑 /etc/yum.repos.d/TBDS.repo文件
3. 在打开的文件中添加如下内容：  
<code>[TBDS]  
name=TBDS  
baseurl=http://10.151.136.14/tbds-mirror-4.0.3.1-docker/  
path=/  
enabled=1  
gpgcheck=0</code>  
4. 更新本地缓存
yum clean all

### 二、添加用户
runner服务 依赖hdfs 用户,并需要将hdfs 用户添加到sudo 组
1. 添加hdfs  
<code>useradd hdfs</code>

2. 添加hdfs到sudo 组  
编辑 /etc/sudoers 文件 ，在最后一行添加  
<code>hdfs ALL=(ALL) NOPASSWD:ALL</code>

### 三、安装runner服务
root 用户操作  
yum install -y lhotse-runners

### 四、安装httpd服务
root 用户操作  
yum install -y httpd

### 五、修改httpd配置  
**使用hdfs用户做如下操作**  
1. 确认文件 /etc/httpd/conf/httpd.conf 包括  
ps:如果不包括，添加该行  
<code>
IncludeOptional conf.d/*.conf
</code>

2. 编辑文件 /etc/httpd/conf.d/runner.conf（不存在新建）  
插入如下内容  
<code>
Listen lhotse-runner 配置是 cgi.port 属性值    
ScriptAlias /runner/ /usr/local/lhotse_runners/getlog/  
</code>
例子如下：  
>Listen 56986    
ScriptAlias /runner/ /usr/local/lhotse_runners/getlog/  
3. 修改httpd 项目目录权限  
>chown -R hdfs:hadoop /etc/httpd/logs/;  
chown -R hdfs:hadoop /usr/sbin/httpd;  
chown -R hdfs:hadoop /run/httpd  

### 六、修改runner配置  
**使用hdfs用户做如下操作**  
切到 /usr/local/lhotse_runners/cfg 目录，编辑如下文件
1. discover.properties   
将discovery.service.zk.conn.str 属性修改为tbds 集群中 zk host name list  
<br>效果如下:  
>discovery.service.zk.conn.str=tbds-10-151-140-233:2181,tbds-10-254-100-141:2181,tbds-10-254-99-28:2181   
<br>其他属性值不变化  
2. lhotse_base.properties  
编辑该文件 修改如下内容：<br>  
<code>
base_ip = taskSchduler hostname  
base_port = taskSchduler 对应 lhotse.base.port 属性值  
ftp_user=登陆用户，默认是 hdfs  
ftp_password= 密码默认是 ftp@Tbds.com  
ftp_path=默认是根路径 /  
doAndRedo_url=默认值http://lhotse-yh-webCgi.tencent-distribute.com:8081/LService/RedoAndDoState  
type.list=37,38,39,66,67,68,69,70,72,75,92,98,99,100,104,105,106,118,119,120  
discovery.service.zk.conn.str=tbds 集群 zk host:port 集合  
discovery.cluster.name=默认值 tdw  
</code>
<br>**例子如下：**  
>base_ip = tbds-10-254-99-18  
base_port = 9930  
ftp_user=hdfs  
ftp_password=ftp@Tbds.com  
ftp_path=/  
doAndRedo_url=http://lhotse-yh-webCgi.tencent-distribute.com:8081/LService/RedoAndDoState  
type.list=37,38,39,66,67,68,69,70,72,75,92,98,99,100,104,105,106,118,119,120  
discovery.service.zk.conn.str=tbds-10-151-140-233:2181,tbds-10-254-100-141:2181,tbds-10-254-99-28:2181  
discovery.cluster.name=tdw  

---
**该文档接下来的部分只对在原集群上升级系统适用，对于新安装的环境不适用**  

*ps:*  tdbank  给江苏消防的环境，需要执行下面的步骤。  

### 七、 修改base 设置  
1. 添加sql server 导入hive 任务类型    
登陆portal(或者8080端口)修改lhots-runner 配置type.list属性，在其中添加106 任务类型,并保存。    
![添加新任务类型](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/0317fd959cc95c7d20198aa2443b77a1c15a11fa3c77eb0b24bf9675441ed24e5a2d6d6c7459f64aa6df4572e589786b?pictype=scale&from=30113&version=2.0.0.2&uin=821244074&fname=1508908165%281%29.jpg&size=1024) 

### 八、 重启base
在portal （或者8080端口）重启taskSchduler  

### 九、 启动tbds集群以外节点的runner服务  
**hdfs 用户操作**
1. 启动runner 服务  
切到/usr/local/lhotse_runners 目录  
启动runner服务  ./start_jar.sh  
2. 启动httpd 服务  
执行/usr/sbin/httpd  
3. 确认httpd，runner启动  
执行命令：netstat -pan|grep httpd  有相关输出,确认httpd 启动ok.  
执行命令： jps ,有TaskRunnerLoader 相关进程信息，确认runner启动ok.    
