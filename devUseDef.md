## 自定义Runner代码设计说明文档
**ps:支持源服务器和目标服务器**

### 1 .需要依赖的jar包
 自定义runner任务依赖 runner-common.jar，改依赖添加了hdfs,base，mysql（jdbc的依赖）
``` 
<dependency>
            <groupId>com.tencent.teg.dc.runner</groupId>
            <artifactId>lhotse-common</artifactId>
            <version>0.0.1-SNAPSHOT</version>
 </dependency>
``` 

### 2 .继承的基类  
1. 与hive相关的实现可以继承AbstractTDWDDCTaskRunner这个基类，继承其中对数据库表的debug。    
2. HDFS与传统数据库导入的runner可以继承HDFSToDBRunner或者DBToHdfsRunner，调用DBUtil类来调用JDBC执行DAO  
3. 普通的runner直接继承jar包中的AbstractTaskRunner基类就可以了。
       
### 3 .必须实现的抽象方法  
   重载AbstractTaskRunner的execute()方法和kill（）方法。

### 4 .启动任务流程      
4.1 新建main 函数 public static void main(String[] args) {}  
> 在main 函数中实例化runner对象,并将args[0] 作为实例化对象的传入参数。  

4.2 在main函数中调用对象startwork()方法，启动自定义runner。
### 5 .获取参数的方法
__5.1 通过this.getTask()方法获取TaskRuntimeInfo对象。__  

__5.2 通过TaskRuntimeInfo对象（task），获取获取用户填写参数:__   
5.2.1  获取实例和任务信息  
>task.getId() 获取实例id  
>task.getType() 获取任务类型  
>task.getCurRunDate() 获取实例数据时间  
>task.getNextRunDate() 获取实例下一个数据时间  

5.2.2 获取用户在ui上填写的填写任务参数__  
>this.getProperties().get(key) //获取任务参数.  

5.2.3 获取源服务器和目标服务器信息  
5.2.3.1  获取源服务器和目标服务器对象  

    List<ServerRuntime> sServerList = task.getSourceServers();  // 源服务器列表  
    ServerRuntime dbServer = sServerList.get(0);// 获取第一个源服务器信息  （注意做非空判断）
  
5.2.3.2 服务器信息说明  
每一个服务器类型包括如下属性，可以使用对应的get 方法获取属性值
    
    String tag;  // 服务器别名
    String type;  // 类型
    String host;  // 地址
    int port;  // 端口
    String userName;  // 用户名 （db 类型会用到）
    String password;  // 密码（db类型会用到）
 
**tips:**  
并不是每一个服务器类型都提供5.2.3.2 节所提到的属性值，具体属性值内容请参考portal 服务器配置页面。  


### 6 .日志输出方法  
    this.writeLocalLog(Level.INFO, "****");      
    
### 7 .提交任务实例执行状态  
      this.commitTask(state, runtimeId, desc);  
Ps:   
       常用运行状态有：RUNNING（正在运行），KILLED（已经停止），SUCCESSFUL（成功），FAILED（失败）

### 8 .停止任务实例运行     
   1. 停止实例运行前，清理runner资源，并提交停止状态。
   2. 我们实现了runner调度资源的清理方法， CommonUtils.killProcess(this.taskRuntime, this);
    __建议开发者，使用同一的停止方法，如下：__  
``` 
	public void kill() throws IOException {
		this.writeLocalLog(Level.INFO, " hello word had been kill ");
		boolean killResult = false;
		try {
			killResult = CommonUtils.killProcess(this.taskRuntime, this);
			if (killResult) {
				this.writeLocalLog(Level.SEVERE, "kill job succeed!");
				this.commitTask(LState.KILLED, "", "kill job succeed!");
			} else {
				this.writeLocalLog(Level.SEVERE, "kill job failed!");
				this.commitTask(LState.HANGED, "", "kill job failed!");
			}
		} catch (Exception e) {
			this.writeLocalLog(Level.SEVERE,
					"kill job failed:" + CommonUtils.stackTraceToString(e));
			this.commitTask(LState.HANGED, "", "kill job failed!");
		}
	}
``` 
### 9. HellowordRunner示例  
    
    import com.tencent.teg.dc.lhotse.proto.LhotseObject.LState;  
    import com.tencent.teg.dc.lhotse.runner.AbstractCustomTaskTypeRunner;  
    import com.tencent.teg.dc.lhotse.runner.util.*;  
    import org.apache.commons.lang.StringUtils;  
    import java.io.IOException;
    import java.util.logging.Level;
    
    public class Helloword extends AbstractCustomTaskTypeRunner {
    
    	public Helloword(String configFileName) {
    		super(configFileName);
    	}
    
    	private boolean success = false;
    
    	public static void main(String[] args) {
    
    		String configure = "";
    		if(args == null ||args.length&lt;1){
    
    			System.out.println("需要配置文件作为输入参数");
    			System.exit(2);
    		}else{
    			configure = args[0];
    		}
    
    		Helloword runner = new Helloword(configure);
    		runner.startWork();
    	}
    
    	@Override
    	public void execute() throws IOException {
    
    		try {
    			taskRuntime = this.getTask();
    			String p1 = taskRuntime.getProperties().get("parameter1");
    
    			this.writeLocalLog(Level.INFO, "get parameter 1 =" + p1);
    
    			if(!"kill".equals(p1)){
    				commitTaskAndLog(LState.FAILED, "", "parameter is kill");
    			}
			List<ServerRuntime> sServerList = task.getSourceServers();  // 源服务器列表
			if(CollectionUtils.isEmpty(sServerList) || sServerList.size() < 1){//少于1台
				sl.steppingLog(Level.INFO , 0, "源服务器数量异常");
				 keyValues.put("exit_code", RunnerUtils.SERVER_CONFIG_ERROR_CODE);
				 keyValues.put("run_date", HOUR_FORMAT.format(runDate));
				 keyValues.put("task_desc", "Task server configuration is incorrect, expect at least 1 source server configured！");
				 commitJsonResult(keyValues, false, "");
				 committed = true;
				 return;
			}
			ServerRuntime dbServer = sServerList.get(0);
			String serverHost = dbServer.getHost();
			int serverPort = dbServer.getPort();
            writeLocalLog(Level.INFO,"源服务器 地址:" + serverHost + ",端口：" + serverPort); 

    			success = true;
    		} catch (Exception e) {
    			this.writeLocalLog(Level.SEVERE ,"执行HelloWord runner 出现异常");
    
    			String st = CommonUtils.stackTraceToString(e);
    			this.writeLocalLog(Level.SEVERE, "Exception stackTrace: " + st);
    
    			commitTaskAndLog(LState.RUNNING, "", "Exception: " + e.getMessage());
    			throw new IOException(e);
    		}
    		finally {
    			if (!success) {
    				commitTaskAndLog(LState.FAILED, "", "failed");
    			} else {
    				commitTaskAndLog(LState.SUCCESSFUL, "", "success execute");
    
    			}
    		}
    	}
    
    
    	@Override
    	public void kill() throws IOException {
    		this.writeLocalLog(Level.INFO, " hello word had been kill ");
    		boolean killResult = false;
    		try {
    			killResult = CommonUtils.killProcess(this.taskRuntime, this);
    			if (killResult) {
    				this.writeLocalLog(Level.SEVERE, "kill job succeed!");
    				this.commitTask(LState.KILLED, "", "kill job succeed!");
    			} else {
    				this.writeLocalLog(Level.SEVERE, "kill job failed!");
    				this.commitTask(LState.HANGED, "", "kill job failed!");
    			}
    		} catch (Exception e) {
    			this.writeLocalLog(Level.SEVERE,
    					"kill job failed:" + CommonUtils.stackTraceToString(e));
    			this.commitTask(LState.HANGED, "", "kill job failed!");
    		}
    	}
    
    	private void commitTaskAndLog(LState state, String runtimeId, String desc) {
    		try {
    			if (desc != null && desc.length() &gt; 4000) {
    				desc = StringUtils.substring(desc, 0, 4000);
    			}
    			this.commitTask(state, runtimeId, desc);
    		}
    		catch (Exception e) {
    			String st = CommonUtils.stackTraceToString(e);
    			this.writeLocalLog(Level.INFO, "Log_desc :" + desc);
    			this.writeLocalLog(Level.SEVERE, "Commit task failed, StackTrace: " + st);
    		}
    	}
    
    }
    
       