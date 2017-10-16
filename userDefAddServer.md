该文档用于支持多集群版本用户，在自定义任务类型中添加源服务器和目标服务器。  

**ps**: 目前不能支持在前台页面中直接添加服务器配置，需要在后台手动操作，相关的操作流程如下：

### 1. 更新 master-runner-1.0-SNAPSHOT-jar-with-dependencies.jar  
__在每一个runner节点执行如下操作__  
###### 1.1  切到/usr/local/lhotse_runners/jar 目录   
> cd /usr/local/lhotse_runners/jar  
###### 1.2  切换到hdfs 用户(需要先切到root 用户)  
> su hdfs
###### 1.3  更新/usr/local/lhotse_runners/jar/master-runner-1.0-SNAPSHOT-jar-with-dependencies.jar  
###### 1.4  查看/usr/local/lhotse_runners/jar 目录下是否存在 目录名大于120文件夹（不包含120）  
如果存在121,122 等文件夹，需要更新121 ，122 等文件夹下的master-runner-1.0-SNAPSHOT-jar-with-dependencies.jar  文件  

### 2. 开发自定义任务类型的执行代码
请参考 devUseDef.md

### 3. 新建一个任务类型（自定义）
在portal页面新建一个需要添加服务器配置的任务类型

### 4. db 更新
######4.1 通过查看自定义任务类型的返回结果，确认第2步新建的任务类型类型id   
![](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/f3738bc7d270b438a010004671dfd8a06e98f4abbc7591037f089896e833a9f8a148cc731ca6bef3fb90e3314fef731a?pictype=scale&from=30113&version=2.0.0.2&uin=821244074&fname=1507798739.png&size=1024)  
这里我们假设新建的任务类型id 为 122  
######4.2 登陆metadb 数据库，添加服务器类型  
**4.2.1 登陆metadb**  
> mysql -h{metadbIP} -ulhotse -plhotse@Tbds.com lhotse_open;(如：mysql -h10.234.135.21 -ulhotse -plhotse@Tbds.com lhotse_open;)  

**4.2.2 执行更新操作（这里我们假设需要添加的源服务器是hive,目标服务器是hdfs** 
>update lb_task_type   set source_server_type="hive" ,target_server_type="hdfs" where type_id=122;  

#####4.3 登陆tbds 数据库，添加服务器类型
**4.3.1 登陆tbds**  
>mysql -hportalIP -uroot -pportal@Tbds.com tbds; 

**4.3.2 执行更新操作**
>update task_type_info  set source_server_type="hive", target_server = "hdfs" where task_type_id = 122 \G

____
目前支持的服务器类型有(小写):  
hive,hdfs,hbase,mysql,oracle,postgre,ftp  
