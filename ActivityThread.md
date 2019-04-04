`ActivityThread`  [在线源码](http://androidxref.com/8.1.0_r33/xref/frameworks/base/core/java/android/app/ActivityThread.java)
首先看看这个类是个什么东西，从名字来看，是Activity 的Thread。
```java
public final class ActivityThread extends ClientTransactionHandler {
```
`ActivityThread` 继承了 `ClientTransactionHandler` 这个类， 这个类又是干嘛的呢？
```java
/**
 * Defines operations that a {@link android.app.servertransaction.ClientTransaction} or its items
 * can perform on client.
 * @hide
 */
public abstract class ClientTransactionHandler {
 /** Prepare and schedule transaction for execution. */
    void scheduleTransaction(ClientTransaction transaction) {
        transaction.preExecute(this);
        sendMessage(ActivityThread.H.EXECUTE_TRANSACTION, transaction);
    }
     @VisibleForTesting
    public void executeTransaction(ClientTransaction transaction) {
        transaction.preExecute(this);
        getTransactionExecutor().execute(transaction);
        transaction.recycle();
    }
}
```
看文档注释大意是搞生命周期的一个东西，先不去管他
大致浏览一下`ActivityThread`的类组织结构，

```java
 final ApplicationThread mAppThread = new ApplicationThread();// 应用进程
    final Looper mLooper = Looper.myLooper();// 进入即实例化了一个Looper
    final H mH = new H();// H extends Hanlder
    final Executor mExecutor = new HandlerExecutor(mH);// 
    final ArrayMap<IBinder, ActivityClientRecord> mActivities = new ArrayMap<>();
```

发现还有一个 `main`方法，来瞧一瞧是什么玩意
```java
public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Set the reporter for event logging in libcore
        EventLogger.setReporter(new EventLoggingReporter());

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        Process.setArgV0("<pre-initialized>");

        Looper.prepareMainLooper();

        // Find the value for {@link #PROC_START_SEQ_IDENT} if provided on the command line.
        // It will be in the format "seq=114"
        long startSeq = 0;
        if (args != null) {
            for (int i = args.length - 1; i >= 0; --i) {
                if (args[i] != null && args[i].startsWith(PROC_START_SEQ_IDENT)) {
                    startSeq = Long.parseLong(
                            args[i].substring(PROC_START_SEQ_IDENT.length()));
                }
            }
        }
        ActivityThread thread = new ActivityThread();
        thread.attach(false, startSeq);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```
