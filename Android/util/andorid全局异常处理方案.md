```java
package cn.rjgc.project.util;

import android.content.Context;
import android.content.pm.PackageInfo;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Environment;
import android.os.Looper;
import android.text.TextUtils;
import android.widget.Toast;

import java.io.File;
import java.io.FileOutputStream;
import java.io.PrintWriter;
import java.io.StringWriter;
import java.io.Writer;
import java.lang.ref.WeakReference;
import java.lang.reflect.Field;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

public class CrashHandler implements Thread.UncaughtExceptionHandler {

    private WeakReference<Context> mContext;
    //异常处理器
    private Thread.UncaughtExceptionHandler mHandler;

    //用来存储设备信息和异常信息
    private Map<String, String> mInfo = new HashMap<>();
    private SimpleDateFormat simpledateformat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

    // 到SD卡目录
    private final static String DISK_CACHE_PATH = "/don/";

    private CrashHandler() {

    }

    private static class CrashHandlerHolder {
        static final CrashHandler INSTANCE = new CrashHandler();
    }

    public static CrashHandler getInstance() {
        return CrashHandlerHolder.INSTANCE;
    }


    @Override
    public void uncaughtException(Thread thread, Throwable throwable) {

        if (!handleException(throwable)) {
            //没有人为处理，调用系统默认的处理器处理
            if (mHandler != null) {
                mHandler.uncaughtException(thread, throwable);
            }
        } else {
            //已经人为处理
            Toast.makeText(mContext.get(), "嗷嗷，出错了", Toast.LENGTH_SHORT).show();
        }
    }

    /**
     * 初始化
     * @param context
     */
    public void init(Context context) {
        mContext = mContext = new WeakReference<>(context);
        mHandler = Thread.getDefaultUncaughtExceptionHandler();
        Thread.setDefaultUncaughtExceptionHandler(this);
    }

    /**
     * 人为处理异常
     * @param e
     * @return true:已经处理  false:未处理
     */
    private boolean handleException(Throwable e) {
        if (e == null) {
            return false;
        }
        //toast提示
        new Thread(){
            @Override
            public void run() {
                Looper.prepare();
                Toast.makeText(mContext.get(), "发生了未知异常", Toast.LENGTH_SHORT).show();
                Looper.loop();
            }
        }.start();
        //收集错误信息
        collectErrorInfo();
        //保存错误信息
        saveErrorInfo(e);
        return true;
    }

    private void collectErrorInfo() {
        PackageManager pm = mContext.get().getPackageManager();
        try {
            PackageInfo pi = pm.getPackageInfo(mContext.get().getPackageName(), PackageManager.GET_ACTIVITIES);
            if (pi != null) {
                String versionName = TextUtils.isEmpty(pi.versionName) ? "未设置版本名称" : pi.versionName;
                String versionCode = pi.versionCode + "";

                mInfo.put("versionName", versionName);
                mInfo.put("versionCode", versionCode);
            }

            //利用Build类，通过反射可以获得构建类里的所有属性(包含设备信息等)
            Field[] fields = Build.class.getFields();
            if (fields != null && fields.length > 0) {
                for (Field field : fields) {
                    field.setAccessible(true);
                    mInfo.put(field.getName(), field.get(null).toString());
                }
            }
        } catch (PackageManager.NameNotFoundException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }


    private void saveErrorInfo(Throwable e) {
        StringBuilder stringBuilder = new StringBuilder();
        for (Map.Entry<String, String> entry : mInfo.entrySet()) {
            String keyName = entry.getKey();
            String value = entry.getValue();
            stringBuilder.append(keyName + "=" + value + "\n");
        }

        Writer writer = new StringWriter();
        PrintWriter printWriter = new PrintWriter(writer);
        e.printStackTrace(printWriter);//栈异常信息
        Throwable cause = e.getCause();//错误信息
        while (cause != null) {
            cause.printStackTrace(printWriter);
            cause = e.getCause();
        }
        printWriter.close();
        String result = writer.toString();
        stringBuilder.append(result);

        String time = simpledateformat.format(new Date());
        String filename = "/" + "_crash_" + time + ".log";
        File file = new File(getFilePath() + filename);

        FileOutputStream fos = null;
        try {
            if (file.exists()) {
                // 清空之前的记录
                file.delete();
            } else {
                file.getParentFile().mkdirs();
            }
            file.createNewFile();
            fos = new FileOutputStream(file);
            fos.write(stringBuilder.toString().getBytes());
        } catch (Exception ex) {
            ex.printStackTrace();
        } finally {
            try {
                if (fos != null) {
                    fos.close();
                }
            } catch (Exception ex) {
                ex.printStackTrace();
            }

            //上传文件到服务器 todo
        }
    }

    /**
     * 创建总文件夹
     */
    public String getFilePath() {
        String cachePath;
        // /mnt/sdcard判断有无SD卡
        if (Environment.getExternalStorageState().equals(
                Environment.MEDIA_MOUNTED)) {
            cachePath = Environment.getExternalStorageDirectory()
                    + DISK_CACHE_PATH;
        } else {
            // 没有就创建到手机内存
            cachePath = mContext.get().getCacheDir() + DISK_CACHE_PATH;
        }
        File file = new File(cachePath);
        if (!file.exists()) {
            // 创建文件夹
            file.mkdirs();
        }
        return cachePath;
    }

}


```

在自定义的application中使用如下代码调用
```java
CrashHandler.getInstance().init(this);
```

ps：另外推荐第三方错误日志处理工具腾讯的Bugly，https://bugly.qq.com/v2/
