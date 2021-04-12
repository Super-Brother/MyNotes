# **主要对象功能介绍**

- **ActivityManagerServices**，简称AMS，服务端对象，负责系统中所有Activity的生命周期
- **ActivityThread**，App的真正入口。当开启App之后，会调用main()开始运行，开启消息循环队列，这就是传说中的UI线程或者叫主线程。与ActivityManagerServices配合，一起完成Activity的管理工作
- **ApplicationThread**，用来实现ActivityManagerService与ActivityThread之间的交互。在ActivityManagerService需要管理相关Application中的Activity的生命周期时，通过ApplicationThread的代理对象与ActivityThread通讯。
- **ApplicationThreadProxy**，是ApplicationThread在服务器端的代理，负责和客户端的ApplicationThread通讯。AMS就是通过该代理与ActivityThread进行通信的。
- **Instrumentation**，每一个应用程序只有一个Instrumentation对象，每个Activity内都有一个对该对象的引用。Instrumentation可以理解为应用进程的管家，ActivityThread要创建或暂停某个Activity时，都需要通过Instrumentation来进行具体的操作。
- **ActivityStack**，Activity在AMS的栈管理，用来记录已经启动的Activity的先后关系，状态信息等。通过ActivityStack决定是否需要启动新的进程。
- **ActivityRecord**，ActivityStack的管理对象，每个Activity在AMS对应一个ActivityRecord，来记录Activity的状态以及其他的管理信息。其实就是服务器端的Activity对象的映像。
- **TaskRecord**，AMS抽象出来的一个“任务”的概念，是记录ActivityRecord的栈，一个“Task”包含若干个ActivityRecord。AMS用TaskRecord确保Activity启动和退出的顺序。如果你清楚Activity的4种launchMode，那么对这个概念应该不陌生。



# **主要流程介绍**

## **zygote是什么？有什么作用？**

**zygote意为“受精卵“**。Android是基于Linux系统的，而在Linux中，所有的进程都是由init进程直接或者是间接fork出来的，zygote进程也不例外。

在Android系统里面，zygote是一个进程的名字。Android是基于Linux System的，当你的手机开机的时候，Linux的内核加载完成之后就会启动一个叫“init“的进程。在Linux System里面，所有的进程都是由init进程fork出来的，我们的zygote进程也不例外。

**我们都知道，每一个App其实都是**

- **一个单独的dalvik虚拟机**
- **一个单独的进程**

所以当系统里面的第一个zygote进程运行之后，在这之后再开启App，就相当于开启一个新的进程。而为了实现资源共用和更快的启动速度，Android系统开启新进程的方式，是通过fork第一个zygote进程实现的。所以说，除了第一个zygote进程，其他应用所在的进程都是zygote的子进程，这下你明白为什么这个进程叫“受精卵”了吧？因为就像是一个受精卵一样，它能快速的分裂，并且产生遗传物质一样的细胞！

## **SystemServer是什么？有什么作用？它与zygote的关系是什么？**

SystemServer也是一个进程，而且是由zygote进程fork出来的。

这个进程是Android Framework里面两大非常重要的进程之一——另外一个进程就是上面的zygote进程。

为什么说SystemServer非常重要呢？因为系统里面重要的服务都是在这个进程里面开启的，比如

ActivityManagerService、PackageManagerService、WindowManagerService等等，看着是不是都挺眼熟的？

那么这些系统服务是怎么开启起来的呢？

在zygote开启的时候，会调用ZygoteInit.main()进行初始化

```
public static void main(String argv[]) {

     ...ignore some code...

    //在加载首个zygote的时候，会传入初始化参数，使得startSystemServer = true
     boolean startSystemServer = false;
     for (int i = 1; i < argv.length; i++) {
                if ("start-system-server".equals(argv[i])) {
                    startSystemServer = true;
                } else if (argv[i].startsWith(ABI_LIST_ARG)) {
                    abiList = argv[i].substring(ABI_LIST_ARG.length());
                } else if (argv[i].startsWith(SOCKET_NAME_ARG)) {
                    socketName = argv[i].substring(SOCKET_NAME_ARG.length());
                } else {
                    throw new RuntimeException("Unknown command line argument: " + argv[i]);
                }
            }

            ...ignore some code...

         //开始fork我们的SystemServer进程
     if (startSystemServer) {
                startSystemServer(abiList, socketName);
         }

     ...ignore some code...

}
```

我们看下startSystemServer()做了些什么

```
  /**留着这个注释，就是为了说明SystemServer确实是被fork出来的
     * Prepare the arguments and fork for the system server process.
     */
    private static boolean startSystemServer(String abiList, String socketName)
            throws MethodAndArgsCaller, RuntimeException {

         ...ignore some code...

        //留着这段注释，就是为了说明上面ZygoteInit.main(String argv[])里面的argv就是通过这种方式传递进来的
        /* Hardcoded command line to start the system server */
        String args[] = {
            "--setuid=1000",
            "--setgid=1000",
            "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1018,1032,3001,3002,3003,3006,3007",
            "--capabilities=" + capabilities + "," + capabilities,
            "--runtime-init",
            "--nice-name=system_server",
            "com.android.server.SystemServer",
        };

        int pid;
        try {
            parsedArgs = new ZygoteConnection.Arguments(args);
            ZygoteConnection.applyDebuggerSystemProperty(parsedArgs);
            ZygoteConnection.applyInvokeWithSystemProperty(parsedArgs);

        //确实是fuck出来的吧，我没骗你吧~不对，是fork出来的 -_-|||
            /* Request to fork the system server process */
            pid = Zygote.forkSystemServer(
                    parsedArgs.uid, parsedArgs.gid,
                    parsedArgs.gids,
                    parsedArgs.debugFlags,
                    null,
                    parsedArgs.permittedCapabilities,
                    parsedArgs.effectiveCapabilities);
        } catch (IllegalArgumentException ex) {
            throw new RuntimeException(ex);
        }

        /* For child process */
        if (pid == 0) {
            if (hasSecondZygote(abiList)) {
                waitForSecondaryZygote(socketName);
            }

            handleSystemServerProcess(parsedArgs);
        }

        return true;
    }
```



## **ActivityManagerService是什么？什么时候初始化的？有什么作用？**

ActivityManagerService，简称AMS，服务端对象，**负责系统中所有Activity的生命周期。**

ActivityManagerService进行初始化的时机很明确，就是在SystemServer进程开启的时候，就会初始化ActivityManagerService。从下面的代码中可以看到

```
public final class SystemServer {

    //zygote的主入口
    public static void main(String[] args) {
        new SystemServer().run();
    }

    public SystemServer() {
        // Check for factory test mode.
        mFactoryTestMode = FactoryTest.getMode();
    }

    private void run() {

        ...ignore some code...

        //加载本地系统服务库，并进行初始化 
        System.loadLibrary("android_servers");
        nativeInit();

        // 创建系统上下文
        createSystemContext();

        //初始化SystemServiceManager对象，下面的系统服务开启都需要调用SystemServiceManager.startService(Class<T>)，这个方法通过反射来启动对应的服务
        mSystemServiceManager = new SystemServiceManager(mSystemContext);

        //开启服务
        try {
            startBootstrapServices();
            startCoreServices();
            startOtherServices();
        } catch (Throwable ex) {
            Slog.e("System", "******************************************");
            Slog.e("System", "************ Failure starting system services", ex);
            throw ex;
        }

        ...ignore some code...

    }

    //初始化系统上下文对象mSystemContext，并设置默认的主题,mSystemContext实际上是一个ContextImpl对象。调用ActivityThread.systemMain()的时候，会调用ActivityThread.attach(true)，而在attach()里面，则创建了Application对象，并调用了Application.onCreate()。
    private void createSystemContext() {
        ActivityThread activityThread = ActivityThread.systemMain();
        mSystemContext = activityThread.getSystemContext();
        mSystemContext.setTheme(android.R.style.Theme_DeviceDefault_Light_DarkActionBar);
    }

    //在这里开启了几个核心的服务，因为这些服务之间相互依赖，所以都放在了这个方法里面。
    private void startBootstrapServices() {

        ...ignore some code...

        //初始化ActivityManagerService
        mActivityManagerService = mSystemServiceManager.startService(
                ActivityManagerService.Lifecycle.class).getService();
        mActivityManagerService.setSystemServiceManager(mSystemServiceManager);

        //初始化PowerManagerService，因为其他服务需要依赖这个Service，因此需要尽快的初始化
        mPowerManagerService = mSystemServiceManager.startService(PowerManagerService.class);

        // 现在电源管理已经开启，ActivityManagerService负责电源管理功能
        mActivityManagerService.initPowerManagement();

        // 初始化DisplayManagerService
        mDisplayManagerService = mSystemServiceManager.startService(DisplayManagerService.class);

    //初始化PackageManagerService
    mPackageManagerService = PackageManagerService.main(mSystemContext, mInstaller,
       mFactoryTestMode != FactoryTest.FACTORY_TEST_OFF, mOnlyCore);

    ...ignore some code...

    }

}
```



## **Launcher是什么？什么时候启动的？**

Launcher本质上也是一个应用程序，和我们的App一样，也是继承自Activity

packages/apps/Launcher2/src/com/android/launcher2/Launcher.java

我们找到BubbleTextView对象的点击事件，就可以找到Launcher如何开启一个App了。

BubbleTextView的点击事件在哪里呢？我来告诉你：在Launcher.onClick(View v)里面。

```
/**
     * Launches the intent referred by the clicked shortcut
     */
    public void onClick(View v) {

          ...ignore some code...

         Object tag = v.getTag();
        if (tag instanceof ShortcutInfo) {
            // Open shortcut
            final Intent intent = ((ShortcutInfo) tag).intent;
            int[] pos = new int[2];
            v.getLocationOnScreen(pos);
            intent.setSourceBounds(new Rect(pos[0], pos[1],
                    pos[0] + v.getWidth(), pos[1] + v.getHeight()));
        //开始开启Activity咯~
            boolean success = startActivitySafely(v, intent, tag);

            if (success && v instanceof BubbleTextView) {
                mWaitingForResume = (BubbleTextView) v;
                mWaitingForResume.setStayPressed(true);
            }
        } else if (tag instanceof FolderInfo) {
            //如果点击的是图标文件夹，就打开文件夹
            if (v instanceof FolderIcon) {
                FolderIcon fi = (FolderIcon) v;
                handleFolderClick(fi);
            }
        } else if (v == mAllAppsButton) {
        ...ignore some code...
        }
    }
```

从上面的代码我们可以看到，在桌面上点击快捷图标的时候，会调用

```
startActivitySafely(v, intent, tag);
```

下面我们就可以一步步的来看一下Launcher.startActivitySafely()到底做了什么事情。

```
boolean startActivitySafely(View v, Intent intent, Object tag) {
        boolean success = false;
        try {
            success = startActivity(v, intent, tag);
        } catch (ActivityNotFoundException e) {
            Toast.makeText(this, R.string.activity_not_found, Toast.LENGTH_SHORT).show();
            Log.e(TAG, "Unable to launch. tag=" + tag + " intent=" + intent, e);
        }
        return success;
    }
```

调用了startActivity(v, intent, tag)

```
boolean startActivity(View v, Intent intent, Object tag) {

        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        try {
            boolean useLaunchAnimation = (v != null) &&
                    !intent.hasExtra(INTENT_EXTRA_IGNORE_LAUNCH_ANIMATION);

            if (useLaunchAnimation) {
                if (user == null || user.equals(android.os.Process.myUserHandle())) {
                    startActivity(intent, opts.toBundle());
                } else {
                    launcherApps.startMainActivity(intent.getComponent(), user,
                            intent.getSourceBounds(),
                            opts.toBundle());
                }
            } else {
                if (user == null || user.equals(android.os.Process.myUserHandle())) {
                    startActivity(intent);
                } else {
                    launcherApps.startMainActivity(intent.getComponent(), user,
                            intent.getSourceBounds(), null);
                }
            }
            return true;
        } catch (SecurityException e) {
        ...
        }
        return false;
    }
```



## **Instrumentation是什么？和ActivityThread是什么关系？**

还记得前面说过的Instrumentation对象吗？每个Activity都持有Instrumentation对象的一个引用，但是**整个进程只会存在一个Instrumentation对象**。当startActivityForResult()调用之后，实际上还是调用了**mInstrumentation.execStartActivity()**

所以当我们在程序中调用startActivity()的 时候，实际上调用的是Instrumentation的相关的方法。

Instrumentation意为“仪器”，我们先看一下这个类里面包含哪些方法吧

，Instrumentation这个类很重要，对Activity生命周期方法的调用根本就离不开他，他可以说是一个大管家，但是，这个大管家比较害羞，是一个女的，**管内不管外**，是老板娘~

那么你可能要问了，老板是谁呀？

老板当然是大名鼎鼎的ActivityThread了！

ActivityThread你都没听说过？那你肯定听说过传说中的UI线程吧？是的，这就是**UI线程**。我们前面说过，App和AMS是通过Binder传递信息的，那么**ActivityThread就是专门与AMS的外交工作的**。

所以说，AMS是董事会，负责指挥和调度的，ActivityThread是老板，虽然说家里的事自己说了算，但是需要听从AMS的指挥，而Instrumentation则是老板娘，负责家里的大事小事，但是一般不抛头露面，听一家之主ActivityThread的安排。



## **不要使用 startActivityForResult(intent,RESULT_OK)**

```
startActivityForResult(intent,RESULT_OK) = startActivity()
```



## **一个App的程序入口到底是什么？**

是ActivityThread.main()。



## **整个App的主线程的消息循环是在哪里创建的？**

是在ActivityThread初始化的时候，就已经创建消息循环了



## **Application是在什么时候创建的？onCreate()什么时候调用的？**

也是在ActivityThread.main()的时候，再具体点呢，就是在thread.attach(false)的时候。