###### DaemonClient
  1. DaemonConfigurations mConfigurations
  2. String DAEMON_PERMITTING_SP_FILENAME 	= "d_permit";
  3. String DAEMON_PERMITTING_SP_KEY 		= "permitted";

  ```
  String processName = getProcessName();  从/proc/24839/cmdline中读取进程名
  String packageName = base.geMtPackageName();
  if(processName.startsWith(mConfigurations.PERSISTENT_CONFIG.PROCESS_NAME)){
    IDaemonStrategy.Fetcher.fetchStrategy().onPersistentCreate(base, mConfigurations);      // 判断是否是稳固进程，是执行create
  }else if(processName.startsWith(mConfigurations.DAEMON_ASSISTANT_CONFIG.PROCESS_NAME)){
    IDaemonStrategy.Fetcher.fetchStrategy().onDaemonAssistantCreate(base, mConfigurations); // 判断是否是守护进程，是执行create
  }else if(processName.startsWith(packageName)){
    IDaemonStrategy.Fetcher.fetchStrategy().onInitialization(base);
  }

  ```

###### IDaemonStrategy（真正执行守护的策略，分API21/API22/API23/under21/xiaomi）
  1. private final static String INDICATOR_DIR_NAME 					= "indicators";               // 作为指示的文件夹
	2. private final static String INDICATOR_PERSISTENT_FILENAME 		= "indicator_p";          // 指示稳定进程-文件1，用来确定是否创建成功的文件
	3. private final static String INDICATOR_DAEMON_ASSISTANT_FILENAME = "indicator_d";
	4. private final static String OBSERVER_PERSISTENT_FILENAME		= "observer_p";             // 指示稳定进程-文件2，用来确定是否线程或者的观察文件
	5. private final static String OBSERVER_DAEMON_ASSISTANT_FILENAME	= "observer_d";
  private AlarmManager			mAlarmManager;
  private PendingIntent 			mPendingIntent;
  private DaemonConfigurations 	mConfigs;

  - onInitialization()
    调用initIndicators（）                                             // 到indicator文件夹下，创建两个进程的indicator文件
  - onPersistentCreate（）
    从mConfigs中拿到守护进程的serviceName，并启动它
    initAlarm(稳定进程的service)                                       // 拿到alartManager，拿到稳定进程service，并取消alartManager与intent的绑定
    开启线程，线程优先级最高，执行NativeDaemonAPI21.doMaemon（）native     // 主要调起进程，进程间守护，传入4个文件路径File
    对稳定进程的listener执行onPersistentStart（）
  - onDaemonAssistantCreate
    拿到稳定进程的service，并启动它
    initAirlarm(稳定进程的service)
    开启线程，NativeDaemonAPI21.doMaemon（）
    对稳定进程的listener执行onPersistentStart（）
  - onDaemonDead（）
    用alarmManager启动稳定进程的service，100ms一次
    对listener执行onWatchDaemonDaed（）
    并杀死当前进程 android.os.Process.killProcess(android.os.Process.myPid());

######




###### DaemonConfigurations(守护进程配置)
  1. DaemonConfiguration PERSISTENT_CONFIG（稳定的进程）
  2. DaemonConfiguration DAEMON_ASSISTANT_CONFIG（守护代理进程）


  - DaemonConfiguration
    - PROCESS_NAME
    - SERVICE_NAME
    - RECEIVER_NAME


  - DaemonListener
    - onPersistentStart（）
    - onDaemonAssistantStart（）
    - onWatchDaemonDaed（）
