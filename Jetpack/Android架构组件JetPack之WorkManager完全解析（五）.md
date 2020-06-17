**阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)**
## 什么是Workmanager
WorkManager是google提供的异步执行任务的管理框架，会根据手机的API版本和应用程序的状态来选择适当的方式执行任务。当应用在运行的时候会在应用的进程中开一条线程来执行任务，当退出应用时，WorkManager会选择根据设备的API版本使用适合的算法调用JobScheduler或者Firebase JobDispatcher,或者AlarmManager来执行任务。如下图：
![](https://upload-images.jianshu.io/upload_images/19956127-6df67d80fad826da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由上图可以看出WorkManager管理任务执行时底层还是调用JobScheduler,JobDispatcher,AlarmManager，不过WorkManager会根据Android系统的API和应用的运行状态来选择合适的方式执行，并不用我们自己去考虑应用复杂的运行状态来进行选择使用JobScheduler还是JobDispatcher或者AlarmManager调用规则如下图：
![](https://upload-images.jianshu.io/upload_images/19956127-fe6582a166fd5326.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## WorkManager在项目中配置
使用WorkManager需要gradle依赖，进行一下简单配置就可以使用了。找到项目中的app/build.gradle目录下在上面加上下面的依赖。
```
dependencies {
    // 其他依赖配置
    def work_version = "1.0.0-beta02"
    implementation "android.arch.work:work-runtime:$work_version"
}
```
可以在https://developer.android.com/topic/libraries/architecture/adding-components#workmanager获取当前的work-runtime版本并且设置正确的版本号。

## WorkManager主要类及使用
如下图给出了WorkManager中主要的类以及关系图，黄色区域是最主要的三个类，构成了WorkManager的基本框架，红色部分和绿色部分是关联的黄色部分的具体实现或者类里面包含一些规则或数据。
![](https://upload-images.jianshu.io/upload_images/19956127-1102a41cccbfee92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**1、Worker处理要执行的任务的具体逻辑。**

**2、WorkerRequest代表一个独立的可以执行的任务，以及任务执行时的条件和规则，比如说任务执行一次还是多次以及任务的触发条件是什么任务有什么约束等。**

**3、WorkManager提供队列将要执行的WorkerRequest放到队列中管理和执行。**
如下图，三个主要类的关系：
![](https://upload-images.jianshu.io/upload_images/19956127-9250c80847d105fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面分别介绍三个类的作用和使用方法。

**Worker**
Worker是一个抽象类，当有一个要执行的任务的时候可以继承Worker类，重写doWork()方法在doWork()方法中实现具体任务的逻辑。
```
public class MyWorker extends Worker {
    public MyWorker(
            @NonNull Context appContext,
            @NonNull WorkerParameters workerParams) {
        super(appContext, workerParams);
    }
    @NonNull
    @Override
    public Worker.Result doWork() {

        Context applicationContext = getApplicationContext();

        try {

            Bitmap picture = BitmapFactory.decodeResource(
                    applicationContext.getResources(),
                    R.drawable.test);
                    
            return Worker.Result.SUCCESS;
            
        } catch (Throwable throwable) {
        
            return Worker.Result.FAILURE;
            
        }
    }
}
```
在上面的MyWorker实例中，继承了Worker 并且重写了doWork()方法，需要注意的是doWork()方法是有返回值Worker.Result的，可以在任务执行成功是返回Worker.Result.SUCCESS，在任务执行出现异常时返回Worker.Result.FAILURE
doWork()方法的返回值主要有三种
1、Worker.Result.SUCCESS 表示任务执行成功

2、Worker.Result.FAILURE 表示任务执行失败

3、Worker.Result.RETRY 通知WorkManager之后再尝试执行该任务

**WorkRequest**
WorkRequest要指定执行任务的Worker，也可以给WorkRequest加一些规则，比如说什么时候执行任务，任务执行一次还是多次，每一个WorkRequest都有一个自动产生的唯一ID，可以根据唯一ID获取对应任务的状态以及是否取消对应的任务。如下图WorkRequest有两个实现类如下图：
![](https://upload-images.jianshu.io/upload_images/19956127-bad51b7beed165d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1、OneTimeWorkRequest 任务只执行一次
```
OneTimeWorkRequest myWorkRequest =
        new OneTimeWorkRequest.Builder(MyWorker.class)
    .build();
    //将上面定义的MyWorker加入到OneTimeRequest.Builder方法中
WorkManager.getInstance().enqueue(myWorkRequest);//获取WorkManager实例并将WorkRequest进队
```
2、PeriodicWorkRequest
PeriodicWorkRequest重复执行任务，直到被取消才停止。首次执行是任务提交后立即执行或者满足所给的 Constraints条件。以后执行都会根据所给的时间间隔来执行。注意任务的执行可能会有延时，因为WorkManager会根据OS的电量进行优化。
假如设置的Periodic Work是24小时执行一次，有可能根据电池优化策略执行的过程如下：
```
     1     | Jan 01, 06:00 AM
     2     | Jan 02, 06:24 AM
     3     | Jan 03, 07:15 AM
     4     | Jan 04, 08:00 AM
     5     | Jan 05, 08:00 AM
     6     | Jan 06, 08:02 AM
```
由上面的执行时间可以看出，PeriodicWorkRequest并不是准确的按照24小时来执行，会有一定的时间延迟。因此如果需要准确的间隔时间来执行任务的话不能使用PeriodicWorkRequest。
```
Constraints constraints = new Constraints.Builder().setRequiredNetworkType(NetworkType.CONNECTED).build();
PeriodicWorkRequest build = new PeriodicWorkRequest.Builder(MyWorker.class, 25, TimeUnit.MILLISECONDS)
           .addTag(TAG)
           .setConstraints(constraints)
           .build();

WorkManager instance = WorkManager.getInstance();
if (instance != null) {
          instance.enqueueUniquePeriodicWork(TAG, ExistingPeriodicWorkPolicy.REPLACE, build);
}
```
**Constraints**
可以给任务加一些运行的Constraints条件，比如说当设备空闲时或者正在充电或者连接WiFi时执行任务。
```
Constraints myConstraints = new Constraints.Builder()
    .setRequiresDeviceIdle(true)
    .setRequiresCharging(true)
    // Many other constraints are available, see the
    // Constraints.Builder reference
     .build();
OneTimeWorkRequest myWork =
                new OneTimeWorkRequest.Builder(CompressWorker.class)
     .setConstraints(myConstraints)
     .build();
```
**WorkManager**
WorkManager管理WorkRequest队列。并根据设备和其他条件选择执行的具体方式。在大部分情况在如果没有给队列加Contraints，WorkManager会立即执行任务。
```
WorkManager.getInstance().enqueue(myWork);
```
如果要检查任务的执行状态可以通过获取WorkInfo,WorkInfo在WorkManager里面的LiveData<WorkInfo>中。下面是判断任务是否结束的方式。
```
WorkManager.getInstance().getWorkInfoByIdLiveData(myWork.getId())
    .observe(lifecycleOwner, workInfo -> {
        // Do something with the status
        if (workInfo != null && workInfo.getState().isFinished()) {
            // ...
        }
    });
```
**取消任务执行**
通过任务的ID可以获取任务从而取消任务。任务ID可以从WorkRequest中获取。
```
UUID compressionWorkId = compressionWork.getId();
WorkManager.getInstance().cancelWorkById(compressionWorkId);
```
注意并不是所有的任务都可以取消，当任务正在执行时是不能取消的，当然任务执行完成了，取消也是意义的，也就是说当任务加入到ManagerWork的队列中但是还没有执行时才可以取消。

**WorkManager多任务调度**
有时候可能有很多任务需要执行，并且这些任务之前可能有先后顺序或者某些依赖关系，WorkManager提供了很好的方式。
1、先后顺序执行单个任务
比如说有三个任务workA,workB,workC,并且执行顺序只能时workA---->workB---->workC可以用如下的方式处理。
```
WorkManager.getInstance()
    .beginWith(workA)
    .then(workB)  instance
    .then(workC)
    .enqueue();
```
上面的workA,workB,workC,都是WorkRequest的子类实现对象。WorkManager会根据上面的先后顺序来执行workA,workB,workC,，但是如果执行过程中三个任务中有一个失败，整个执行都会结束。并且返回Result.failure()。
2、先后顺序执行多个任务列
有时候可能要先执行一组任务，然后再执行下一组任务，可以使用下面的方式来完成。
```
WorkManager.getInstance()
    // First, run all the A tasks (in parallel):
    .beginWith(Arrays.asList(workA1, workA2, workA3))
    // ...when all A tasks are finished, run the single B task:
    .then(workB)
    // ...then run the C tasks (in any order):
    .then(Arrays.asList(workC1, workC2))
    .enqueue();
```
3、多路径先后执行
上面两种方式都是单路径执行，可以实现更加复杂的多路径执行方式，使用WorkContinuation.combine(List<OneTimeWorkRequest>)如下图要实现的执行方式：
![](https://upload-images.jianshu.io/upload_images/19956127-15257a017922cc38.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
WorkContinuation chain1 = WorkManager.getInstance()
    .beginWith(workA)
    .then(workB);
WorkContinuation chain2 = WorkManager.getInstance()
    .beginWith(workC)
    .then(workD);
WorkContinuation chain3 = WorkContinuation
    .combine(Arrays.asList(chain1, chain2))
    .then(workE);
chain3.enqueue();
```
**使用WorkManager遇到的问题**
1、使用PeriodicWorkRequest只执行一次，并不重复执行。
```
     WorkManager instance= new PeriodicWorkRequest.Builder(PollingWorker.class, 10, TimeUnit.MINUTES)
                .build();
```
原因：PeriodicWorkRequest默认的时间间隔是15分钟如果设置的时间小于15分钟，就会出现问题。

解决方法：设置的默认时间必须大于或等于15分钟。另外要注意，就算设置的时间为15分钟也不一定间隔15分钟就执行。

2、在doWork()方法中更新UI导致崩溃。
原因：doWork()方法是在WorkManager管理的后台线程中执行的，更新UI操作只能在主线程中进行。

解决方法：当doWork()耗时方法执行完之后，将更新UI操作抛到主线中执行，可以用handle来实现，如下：
```
/**
 * Created time 15:32.
 *
 * @author huhanjun
 * @since 2019/1/23
 */
public class PollingWorker extends Worker {
    public static final String TAG = "PollingWorker";

    @NonNull
    @Override
    public Result doWork() {
        Log.d(TAG, "doWork");
        try {
            polling();
            runOnUIThread(new Runnable() {
                @Override
                public void run() {
                    //更新UI操作
                }
            });
            return Result.SUCCESS;
        } catch (Exception e) {
            Log.d(TAG, "failure");
            return Result.FAILURE;
        }
    }

    private void polling() {
        Log.d(TAG, "Polling");
    }

    private void runOnUIThread(Runnable runnable) {
        new Handler(Looper.getMainLooper()).post(runnable);
    }
}
```
在执行完Polling方法后，将更新操作抛给了runOnUIThread()方法，这样就可以在上面代码注释的地方执行更新操作。

原文链接：https://blog.csdn.net/u013309870/article/details/86553531
**阿里P7移动互联网架构师进阶视频（每日更新中）免费学习请点击：[https://space.bilibili.com/474380680](https://links.jianshu.com/go?to=https%3A%2F%2Fspace.bilibili.com%2F474380680)**
