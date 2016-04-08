##### android杀进程的机制
1. 三种杀进程的可能性：
  - 系统force close
  - 安全软件kill
  - 用户手动kill

2. 6类进程被杀死的阀值：（单位为page，1page＝4kb）(需要root)，当达到某一阀值时就开始清理对应进程
  - 记录在系统 /sys/module/lowmemorykiller/parameters
    ```
    shell@zerofltechn:/ $ ls /sys/module/lowmemorykiller/parameters
    adj
    cost
    debug_level
    lmkcount
    minfree
    oomcount
    ```
    ![三星note3](http://img.blog.csdn.net/20150827100733605?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
    6个进程从左到右分别为：
      1. ForegroundProgress
      2. VisibleProgress
      3. SecondaryService
      4. HiddenProgress
      5. ContentProvider
      6. EmptyProgress

    以上6个进程类型定义与kernel层，在framework走了更细致的划分，
      `com.android.server.am.ActivityManager.RunningAppProcessInfo`下查看
      ```
      /**
       * Constant for {@link #importance}: This process is running the
       * foreground UI; that is, it is the thing currently at the top of the screen
       * that the user is interacting with.
       */
      public static final int IMPORTANCE_FOREGROUND = 100;

      /**
       * Constant for {@link #importance}: This process is running a foreground
       * service, for example to perform music playback even while the user is
       * not immediately in the app.  This generally indicates that the process
       * is doing something the user actively cares about.
       */
      public static final int IMPORTANCE_FOREGROUND_SERVICE = 125;

      /**
       * Constant for {@link #importance}: This process is running the foreground
       * UI, but the device is asleep so it is not visible to the user.  This means
       * the user is not really aware of the process, because they can not see or
       * interact with it, but it is quite important because it what they expect to
       * return to once unlocking the device.
       */
      public static final int IMPORTANCE_TOP_SLEEPING = 150;

      /**
       * Constant for {@link #importance}: This process is running something
       * that is actively visible to the user, though not in the immediate
       * foreground.  This may be running a window that is behind the current
       * foreground (so paused and with its state saved, not interacting with
       * the user, but visible to them to some degree); it may also be running
       * other services under the system's control that it inconsiders important.
       */
      public static final int IMPORTANCE_VISIBLE = 200;

      /**
       * Constant for {@link #importance}: This process is not something the user
       * is directly aware of, but is otherwise perceptable to them to some degree.
       */
      public static final int IMPORTANCE_PERCEPTIBLE = 130;

      /**
       * Constant for {@link #importance}: This process is running an
       * application that can not save its state, and thus can't be killed
       * while in the background.
       * @hide
       */
      public static final int IMPORTANCE_CANT_SAVE_STATE = 170;

      /**
       * Constant for {@link #importance}: This process is contains services
       * that should remain running.  These are background services apps have
       * started, not something the user is aware of, so they may be killed by
       * the system relatively freely (though it is generally desired that they
       * stay running as long as they want to).
       */
      public static final int IMPORTANCE_SERVICE = 300;

      /**
       * Constant for {@link #importance}: This process process contains
       * background code that is expendable.
       */
      public static final int IMPORTANCE_BACKGROUND = 400;

      /**
       * Constant for {@link #importance}: This process is empty of any
       * actively running code.
       */
      public static final int IMPORTANCE_EMPTY = 500;

      /**
       * Constant for {@link #importance}: This process does not exist.
       */
      public static final int IMPORTANCE_GONE = 1000;
      ```

    当kill开始，某一进程级别有好多，根据6类进程对应adj，值越大越容易被杀
      `/sys/module/lowmemorykiller/parameters/adj`查看

    android增加Linux中没有的进程adj，CORE_SERVER_ADJ=-12、SYSTEM_ADJ = -16，定义在ActivityManagerService
    ```
    USER      PID   PPID  VSIZE  RSS     WCHAN    PC        NAME
    u0_a1071  8218  3035  1877600 25348 ffffffff 00000000 S com.ss.android.essay.joke:pushservice
    u0_a1071  11560 3035  2064120 155112 ffffffff 00000000 S com.ss.android.essay.joke
    u0_a1071  17108 3035  1879852 16564 ffffffff 00000000 S com.ss.android.essay.joke:push
    ```
    最后是根据oom_source确定杀掉谁（oom_source由oom_adj、占用内存大小、启动时间等加权算出）
      - 真正的lowmemorykiller在keinel层，位于android源码`/kernel/drivers/staging/android/lowmemorykiller`
      ![android源码杀死进程的代码片段](http://img.blog.csdn.net/20150827101425094?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
      ```
      send_sig(SIGKILL, selected, 0);
      ```
      - 负责调用的是在framework层ActivityManagerService
        1. 第三方如何杀死进程呢：ActivityManager.killBackgroundProcess(packageName)，这种方法只要配置一个start_stick就杀不死
        2. framework如何杀死除前台进程外的进程:Process.killProcessQuiet(pid),隐藏方法，反射才可获得
        3. ActivityManager.forceStopPackageAsUser(package, userId)（会杀掉所有和该packageName共享uid的进程，会取消掉该进程所有闹钟）
        ```
        通过fc杀掉的进程在packagemanager中还会设置一个flag，以至于之后所有的广播都会将你过滤掉，防止你被系统的广播呼起，直到用户手动点击icon启动该应用，该flag才会置为false。
        当然这只针对系统广播，我们自己发的广播只要加入一个flag值就ok了

        * {@link android.Manifest.permission#FORCE_STOP_PACKAGES} to be able to
        * call this method.
        forceClose第三方可不可以用呢，根据permission查看需要系统签名，第三方的方法是：利用了android的AccessibilityHelper，
          进到setting，点击对应应用的forceClose

        如果有root权限，代码可以在终端调用 kill -9 pid
        ```


###### android保活方法
  1. 将Service设置为前台进程
  ```
    只是会比activty晚点被回收掉，对于360，cm仍然轻易杀掉
    并且在API17之后，对于前台服务会出现一个无法消失的notification
  ```
  2. 在service的onstart方法里返回 STATR_STICK
  ```
    常规service里还ok，保活意义不大
    START_STICKY_COMPATIBILITY：START_STICKY的兼容版本，但不保证服务被kill后一定能重启。
    START_STICKY：系统就会重新创建这个服务并且调用onStartCommand()方法，但是它不会重新传递最后的Intent对象，这适用于不执行命令的媒体播放器（或类似的服务），它只是无限期的运行着并等待工作的到来。
    START_NOT_STICKY：直到接受到新的Intent对象，才会被重新创建。这是最安全的，用来避免在不需要的时候运行你的服务。
    START_REDELIVER_INTENT：系统就会重新创建了这个服务，并且用最后的Intent对象调。等待中的Intent对象会依次被发送。这适用于如下载文件。
  ```
  3. 添加Manifest文件属性值为android:persistent=“true”
  ```
    局限需要系统签名
  ```
  4. 覆写Service的onDestroy方法
  ```
    在onDestory时候在把自己激活
    对于360然并卵
  ```
  5. 添加广播监听android.intent.action.USER_PRESENT事件以及其他一些可以允许的事件
  ```
    在接收到这些静态广播时调起自己的service
    缺点就是用户会看到你申请了很多权限
  ```
  6. 服务互相绑定
  7. 设置闹钟，定时唤醒
  ```
    高频唤醒，手机厂商对wakeLock的控制和耗电问题
    AlarmManager.RTC，硬件闹钟，不唤醒手机（也可能是其它设备）休眠；当手机休眠时不发射闹钟。
    AlarmManager.RTC_WAKEUP，硬件闹钟，当闹钟发躰时唤醒手机休眠；
    AlarmManager.ELAPSED_REALTIME，真实时间流逝闹钟，不唤醒手机休眠；当手机休眠时不发射闹钟。
    AlarmManager.ELAPSED_REALTIME_WAKEUP，真实时间流逝闹钟，当闹钟发躰时唤醒手机休眠；
  ```
  8. 账户同步，定时唤醒
  ```
    在android设置一个自己的账户
    android会调起唤醒账户更新服务，可以自己设置同步的时间间隔
  ```
  9. native层保活



##### android调用native层保活
  1. 开启一个c进程，将保活的service名字传递进去，c进程定时给service发intent
  ```
  局限c进程不能被杀死
  只能a保b，b无法保a，且必须等到了轮训时间才会去拉活
  ```

  2. (解决父监听子)
  主进程的c层增加调用fock，利用waitpid做检查
  `fork会创建一个子进程，waitpid会是阻塞方法，在子进程死亡的时候会继续执行，监听到死亡在启动子进程`
  `局限：fork出来的子父进程用户名相同，fork出来的内存都算在自己的应用里`

  3. (解决子监听父)
  通过linux的领养机制
  `父进程挂掉，子进程被linux的init领养，父进程id变为1），通过判断父id是否为1，确定父进程是否被杀，是否要发intent`
  `局限：轮询要在10ms以内，才会保证父、子进程不会同时死掉，功耗太大`

  4. (解决轮询功耗问题)
  子父进程建立ipc管道通信机制
  `当另一端被杀掉，管道破坏，另一端的读取方法就会执行返回，由此确定对方已挂掉`
  `局限：解决了子对父的监听的问题，但是fork仍然存在同父进程的名字、内存浪费的问题(5.0以上无效)`

  5. (解决fork同父进程名字和内存问题)
  创建一个可执行文件，它是一个相对独立的进程，有可以创建父子间的管道
  `直接启动一个二进制文件，他会占用原进程，用fork启动二进制进程的工具`
  ```
  父子进程的管道是单向的，这里需要建立两根管道。
  ab两个进程，12两根管道：a进程关掉管道1的写，堵塞管道2的读；b进程关掉管道2的鞋，堵塞管道1的读
  ```

  二进制文件需要做的事情：
  ```
  1、将对应的packagename，servicename以及二进制可执行文件的路径传进来
  2、清理僵尸进程，就像最开始讲的，低端手机会忽略c进程，如果我们恰巧运行在低端手机上，那么c进程得不到释放会越来越多，我们称他为僵尸进程，需要清理一下
  3、建立两条管道
  4、执行二进制文件，将上面的参数传递进去
  5、然后关掉自己管道1的写端和管道2的读端，然后阻塞读取管道1，如果读取到，则代表子进程挂掉
  ```

  需要注意的是：需要创建第三个进程（负责同步ab进程）
  ```
  1、重新拉起来要重新建立双管道，子进程挂掉，父进程把他重启起来建立双管道还好说，如果父进程挂掉，子进程把父进程启动起来，他们之间就无法建立连接，而且如果中间出了差错，同步起来很费劲，于是我选择，无论谁监听到谁死了，都重启对方，然后自杀，重新初始化！

  2、如果执行force close， 系统先杀父进程，子进程监听到之后拉起父进程然后自杀，但是系统杀你两个进程的间隔时间非常非常短，父进程刚起来还没来得及初始化，系统赶过来杀父进程。有的手机强杀之后很短一段时间无法拉起父进程。于是我选择用第三个进程。

  第三个进程和之前的父子进程都没有任何关系，他的作用只是用做拉起常驻进程。父子进程无论谁监听到谁死，都拉起第三个进程，第三个进程负责拉起常驻进程，然后自杀。（用户实际上是看不到他的存在的，因为他可能只存活不到一秒就自杀了）
  ```
  `局限：5.0后的force close对父子进程一同终结，就是说子进程的pipe就监听不到任何事情`

##### android5.0以上解决拉活问题
  （解决：5.0之后force close子进程的pipe无法监听(也可以说成解决fork的局限)）
  1. 5.0的源码中系统强杀的时候会连同同group中的所有进程也一起干掉。在父进程被杀死的时候，子进像是被冻结了一样（pipo不在通信），做不了任何事情，拿不到监听事件
  2. 放弃fork的父子进程，采用两个普通进程，受到fifo管道的启发（fifo是通过一个文件做通信的，对方挂掉也不会提醒）
  3. 整体思路：
    ```
    1、需要让a进程先把文件1锁了，然后不去读文件2，等b进程做同样的操作之后，两边再同时读取对方的锁。
    2、需要考虑第程序重新进入的初始化状态与单个进程等待的状态冲突问题
    3、不能耗时！！时间很重要，后面会说到
    ```
  4. 实现过程：
    ```
    1、 4个文件，a进程文件a1（a锁住，代表a是否存活），a2（代表a是否ready），b进程b1，b2
    2、 a进程加锁文件a1，b进程加锁文件吧b1
    3、 a进程轮询到b2文件存在了，代表b进程已经创建并可能在对b1文件加锁，此时删除文件b2，代表a进程已经加锁完毕
    4、 a进程监听文件a2，如果a2被删除，代表b进程进行到了步骤4已经对b1加锁完成，可以开始读取b1文件的锁
    ```
  5. 局限
    ```
      监听死亡是ok的，但是无法启动对方
      （系统杀应用对应进程的时候，速度非常快，在a进程监听到b进程被杀的时候，系统的死亡镰刀已经伸向了a进程，大概只有几十毫秒的时间！而发送一个intent我们知道他是在自己进程中使用系统ActivityManagerService的一个代理类，通过一个binder将intent传给系统，虽然大部分操作是在ams中，但是我们这边执行时间居然也在百毫秒级的，难怪intent发不出去）
    ```

#### android5.1解决拉活问题
  1. 由于时间都消耗在了pacel的创建上
  2. 利用反射在初始化的时候拿到ActivityManagerNative实例然后拿binder，也就是他的一个成员变量mRemote。
  3. 然后在检测到对方进程死掉的时候，直接调用transcate方法。
  4. 局限：`6.0的binder transcate的方法无法启动一个service`

#### android6.0解决拉活问题
  1. 不在通过service调起，而是使用receiver调起
  2. forceClose时已经可以成功拉起对方，但360/cm会重复杀，被拉起来的新进程还没初始化完成，就又被杀掉了。
  3. 解决：在进程刚创建的时候使用多线程，将文件的同步加锁监听与启动另一个进程同时进行，从来能节约100毫秒左右的时间
