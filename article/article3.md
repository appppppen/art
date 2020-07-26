##新建Unity工程



##unity泛型单例脚本
SingletonUnity.cs
```cs
using UnityEngine;
public class SingletonUnity<T> : MonoBehaviour where T : MonoBehaviour
{	
	public static T instance = null;
    public virtual void Awake()
    {
        if (instance == null)
        {
            DontDestroyOnLoad(gameObject);
            instance = (T)FindObjectOfType(typeof(T));
        }
        else if (instance != null)
        {
            Destroy(gameObject);
        }
    }
}
```
##调用Java脚本
MobPlugin .cs
```cs
using UnityEngine;

public class MobPlugin : SingletonUnity<MobPlugin>
{
#elif UNITY_ANDROID && !UNITY_EDITOR
    AndroidJavaClass jc;
    AndroidJavaObject activity;
#endif

    public void Start()
    {
#if UNITY_ANDROID && !UNITY_EDITOR
        jc = new AndroidJavaClass("com.unity3d.player.UnityPlayer");
        activity = jc.GetStatic<AndroidJavaObject>("currentActivity");
        activity.Call("SetSplashGone");
#endif
    }

    public void UnityToast(string msg)
    {
#if UNITY_ANDROID && !UNITY_EDITOR
        activity.Call("unityToast", msg);
#endif
        Debug.Log("UnityToast:" + msg);
    }

    public void AndroidToast(string msg)
    {
        TestAndroid testAndroid = GetComponent<TestAndroid>();
        if (testAndroid!=null)
        {
            testAndroid.Toast(msg);
        }
    }
}
```
####测试脚本
TestAndroid.cs
```cs
using UnityEngine;
using UnityEngine.UI;

public class TestAndroid : MonoBehaviour {
    public Text text;
    public void UnityToast() {
        MobPlugin.instance.UnityToast("Unity中点击了按钮");
    }

    public void Toast(string msg)
    {
        if (text)
        {
            text.text = msg;
        }
    }
}
```

##简易unity界面搭建
![QQ截图20181119155934.png](https://upload-images.jianshu.io/upload_images/5336648-4c9536e968c40ab9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##unity导出设置
包名可以随意命名之后用不到该包名
![QQ截图20181119160236.png](https://upload-images.jianshu.io/upload_images/5336648-c84c313ec17ed5e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![QQ截图20181119160225.png](https://upload-images.jianshu.io/upload_images/5336648-ba696dda4b80a8db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####导出后的Android工程中间件
![QQ截图20181119160358.png](https://upload-images.jianshu.io/upload_images/5336648-563d3349052988c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###新建Android工程
![QQ截图20181119152618.png](https://upload-images.jianshu.io/upload_images/5336648-bb8ff55393977b3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
####这里使用Android 8.0
因为Google play的要求
![QQ截图20181119152639.png](https://upload-images.jianshu.io/upload_images/5336648-9c8741e36bd8262a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
新建空Activity
![QQ截图20181119152651.png](https://upload-images.jianshu.io/upload_images/5336648-4bd5ff99002da777.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![QQ截图20181119152716.png](https://upload-images.jianshu.io/upload_images/5336648-ea4a30d3bcf78bec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

新建一个模块作为unity模块供主模块调用，让代码更简洁
![QQ截图20181119154107.png](https://upload-images.jianshu.io/upload_images/5336648-8ef1e1b2d474aa6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![QQ截图20181119154121.png](https://upload-images.jianshu.io/upload_images/5336648-7816ff043f1d26ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![QQ截图20181119154146.png](https://upload-images.jianshu.io/upload_images/5336648-e550779ad645be3a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####settings.gradle中要包含unity模块
![QQ截图20181119154222.png](https://upload-images.jianshu.io/upload_images/5336648-fdb7d2ef8ef69953.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####在app模块添加依赖
```java
  implementation project(':unitylib')
```
![QQ截图20181119154340.png](https://upload-images.jianshu.io/upload_images/5336648-52280b6f66819ea9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
添加assets文件夹
![QQ截图20181119154439.png](https://upload-images.jianshu.io/upload_images/5336648-610477c91d9377a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
添加jniLibs文件夹

![QQ截图20181119155208.png](https://upload-images.jianshu.io/upload_images/5336648-00e3698e9cbee3be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
同步代码混淆规则配置文件
![QQ截图20181119160439.png](https://upload-images.jianshu.io/upload_images/5336648-e6bca16142585ed9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![QQ截图20181119160501.png](https://upload-images.jianshu.io/upload_images/5336648-58e84ca50a6407da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
同步unitylib模块的 gradlew
![QQ截图20181119160552.png](https://upload-images.jianshu.io/upload_images/5336648-83f36d4a9fd0c969.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![QQ截图20181119160634.png](https://upload-images.jianshu.io/upload_images/5336648-6f8cfe42b3191c7f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
放置jar包
![QQ截图20181119160732.png](https://upload-images.jianshu.io/upload_images/5336648-2a6b8ff9afd3492a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
放置assets下的文件
![QQ截图20181119160805.png](https://upload-images.jianshu.io/upload_images/5336648-113e1fd38da2bbab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
放置.so文件
![QQ截图20181119160821.png](https://upload-images.jianshu.io/upload_images/5336648-eab9cdcad334ae37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![QQ截图20181119160847.png](https://upload-images.jianshu.io/upload_images/5336648-ffb070918f1846ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####修改UnityPlayerActivity.java
```java
package com.apen.unitylib;//package com.apen.uni;将之前的包名修改为库模块的包名

import com.unity3d.player.*;
import android.app.Activity;
import android.content.Intent;
import android.content.res.Configuration;
import android.view.KeyEvent;
import android.view.MotionEvent;
import android.view.View;
import android.widget.RelativeLayout;

public class UnityPlayerActivity extends Activity
{
    protected UnityPlayer mUnityPlayer;

     /**
     *  设置父视图
     */
    public void setView(RelativeLayout mParent){
        this.mUnityPlayer = new UnityPlayer(this);
        View mView = this.mUnityPlayer.getView();
        mParent.addView(mView);
        this.mUnityPlayer.requestFocus();
    }

....//打包后生成的代码保留，仅删除protected void onCreate(Bundle savedInstanceState) {}方法，为了之后继承该activity不出错
}
```
在app模块中继承UnityPlayerActivity并设置视图
```java
package com.apen.unity;

import android.os.Bundle;
import android.view.View;
import android.view.ViewGroup;
import android.widget.FrameLayout;
import android.widget.ImageView;
import android.widget.RelativeLayout;
import android.widget.Toast;
import com.unity3d.player.UnityPlayer;
import com.apen.unitylib.UnityPlayerActivity;


public class UnityActivity extends WebActivity {
    RelativeLayout mParent;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_unity);

        mParent=findViewById(R.id.UnityView);
        setView(mParent);
        findViewById(R.id.androidclick).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                UnityPlayer.UnitySendMessage("TestAndroid", "Toast", "Android中点击了按钮");
            }
        });
    }
    public void unityToast(final String msg) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        Toast.makeText(UnityActivity.this, msg, Toast.LENGTH_LONG).show();
                    }
                });
            }
        }).start();
    }
}
```

视图xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".UnityActivity">
    <RelativeLayout
        android:id="@+id/UnityView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
    <TextView
        android:id="@+id/androidclick"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="TestAndroid"/>
</RelativeLayout>
```
配置Android 清单文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.apen.unity"
    android:installLocation="preferExternal">
    <supports-screens
        android:smallScreens="true"
        android:normalScreens="true"
        android:largeScreens="true"
        android:xlargeScreens="true"
        android:anyDensity="true" />

    <uses-permission android:name="android.permission.INTERNET" />

    <uses-feature android:glEsVersion="0x00020000" />
    <uses-feature android:name="android.software.leanback" android:required="false" /><!--因为该工程不适配Android TV，所以要设置这项-->
    <uses-feature android:name="android.hardware.touchscreen" android:required="false" />
    <uses-feature android:name="android.hardware.touchscreen.multitouch" android:required="false" />
    <uses-feature android:name="android.hardware.touchscreen.multitouch.distinct" android:required="false" />
    <application
        android:allowBackup="true"
        android:name=".App"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"
        android:isGame="true"
        android:banner="@drawable/app_banner">

        <activity android:label="@string/app_name"
            android:screenOrientation="fullSensor"
            android:launchMode="singleTask"
            android:configChanges="mcc|mnc|locale|touchscreen|keyboard|keyboardHidden|navigation|orientation|screenLayout|uiMode|screenSize|smallestScreenSize|fontScale|layoutDirection|density"
            android:name=".UnityActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
                <category android:name="android.intent.category.LEANBACK_LAUNCHER" />
            </intent-filter>
            <meta-data android:name="unityplayer.UnityActivity" android:value="true" />
        </activity>
        <meta-data android:name="unity.build-id" android:value="64375dc2-7bb1-4490-ad40-f03e6d678f1c" />
        <meta-data android:name="unity.splash-mode" android:value="0" />
        <meta-data android:name="unity.splash-enable" android:value="True" />
        <meta-data android:name="android.max_aspect" android:value="2.1" />
    </application>

</manifest>
```
git：https://github.com/appppppen/Unity4Android-Project.git，欢迎star