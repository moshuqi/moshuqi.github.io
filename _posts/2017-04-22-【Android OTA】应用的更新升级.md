---
layout: post
title:  【Android OTA】应用的更新升级
category: "Android"
---

## App更新升级

本文通过实现一个简单的Demo，来介绍App的更新升级方式。提供更新信息的服务器用到了[上一篇文章实现的服务器Demo](https://moshuqi.github.io/2017/04/21/Android-OTA-用nodejs搭建服务器/)。

App的更新方式主要有两种：

+	完全更新（Full updates）
+	增量更新（Incremental updates，也叫差分包升级）


应用更新前

![image1](/images/posts/OtaApp/1.jpg)

更新完成后

![image2](/images/posts/OtaApp/2.jpg)

应用比较简单，关键是更新的流程，看背景色既可知更新是否成功。

### App实现

文末附Demo完整源码，这里只大概介绍主要的步骤。

主要的界面就是一列表，列表有2个选项

**activity_main.xml**

	<?xml version="1.0" encoding="utf-8"?>
	<RelativeLayout
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:id="@+id/activity_main"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:paddingBottom="@dimen/activity_vertical_margin"
	    android:paddingLeft="@dimen/activity_horizontal_margin"
	    android:paddingRight="@dimen/activity_horizontal_margin"
	    android:paddingTop="@dimen/activity_vertical_margin"
	    tools:context="com.example.hd.ota.Activity.MainActivity">
	
	    <ListView
	        android:id="@+id/list"
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content">
	
	    </ListView>
	</RelativeLayout>
	
列表项初始化，点击列表选项做对应的跳转

**MainActivity.java**

	protected void onCreate(Bundle savedInstanceState) {
	
        ...

        mListView = (ListView)findViewById(R.id.list);

        mList = new ArrayList<String>();
        mList.add("版本信息");
        mList.add("版本更新");

        ArrayAdapter adapter = new ArrayAdapter(this, android.R.layout.simple_list_item_1, mList);
        mListView.setAdapter(adapter);

        mListView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, final View view, int position, long id) {
                Intent intent = null;
                switch (position) {
                    case 0:
                        intent = InfoActivity.getInfoIntent(getApplicationContext());
                        break;

                    case 1:
                        intent = UpdateActivity.getUpdateIntent(getApplicationContext());
                        break;

                    default:
                }

                if (intent != null) {
                    startActivity(intent);
                }
            }
        });
    }
    
**UpdateActivity** 为更新升级界面，进入该界面时会自动向服务器进行更新信息的请求，并将请求的结果显示。

![image3](/images/posts/OtaApp/3.jpg)

新建 **AsyncTask<Void, Void, String>** 子类 **UpdateCheckTask**，用来做后台网络请求。

	protected String doInBackground(Void... voids) {
        HttpURLConnection uRLConnection = null;
        InputStream is = null;
        BufferedReader buffer = null;
        String result = null;
        String urlStr = Constants.UPDATE_REQUEST_URL + "?" + Constants.APK_VERSION_NAME + "=" + InfoUtils.getVersionName(this.mContext);

        try {
            URL url = new URL(urlStr);
            uRLConnection = (HttpURLConnection) url.openConnection();
            uRLConnection.setRequestMethod("GET");

            is = uRLConnection.getInputStream();
            buffer = new BufferedReader(new InputStreamReader(is));
            StringBuilder strBuilder = new StringBuilder();
            String line;
            while ((line = buffer.readLine()) != null) {
                strBuilder.append(line);
            }
            result = strBuilder.toString();
        } catch (Exception e) {
            Log.e(TAG, "http post error");
        } finally {
            ...
        }

        return result;
    }

将获取到的服务器返回数据做json解析，并返回给**UpdateActivity**

	protected void onPostExecute(String result) {
        Log.i(TAG, "onPostExecute()");
        UpdateInfo info = parseJson(result);
        if (this.mListener != null) {
            this.mListener.onSuccess(info);
        }
    }
    
***parseJson*** 函数为**UpdateCheckTask**中实现的解析方法，具体见源码。

**UpdateCheckTask**在*onCreate*中调用更新请求

	new UpdateCheckTask(UpdateActivity.this, this).execute();

检测到更新后，界面出现下载按钮，点击按钮，下载指定url中的更新文件，显示下载进度条

![image4](/images/posts/OtaApp/4.jpg)

新建 **DownloadService** 用来处理下载流程，**DownloadService** 继承自 **IntentService** 类

**DownloadService** 通过指定地址将文件下载到本地

	@Override
    protected void onHandleIntent(Intent intent) {
    	String urlStr = Constants.OTA_SERVER_IP + intent.getStringExtra(Constants.APK_DOWNLOAD_URL);
        String md5 = intent.getStringExtra(Constants.APK_MD5);
        boolean isDiff = intent.getBooleanExtra(Constants.APK_DIFF_UPDATE, false);
        InputStream in = null;
        FileOutputStream out = null;
        try {
            URL url = new URL(urlStr);
            HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
            
            ...
    }
    
下载过程中将进度以消息的方式通知**UpdateActivity**以更新进度条

	...
	
	int oldProgress = 0;
    Intent sendIntent = new Intent(UpdateActivity.SERVICE_RECEIVER);
    while ((byteread = in.read(buffer)) != -1) {
        bytesum += byteread;
        out.write(buffer, 0, byteread);

        int progress = (int) (bytesum * 100L / bytetotal);
        // 如果进度与之前进度相等，则不更新，如果更新太频繁，否则会造成界面卡顿
        if (progress != oldProgress) {
            sendIntent.putExtra(Constants.UPDATE_DOWNLOAD_PROGRESS, progress);
            getApplicationContext().sendBroadcast(sendIntent);
        }
        oldProgress = progress;
    }
    
    ...
    
**UpdateActivity** 监听进度消息

	mReceiver = new ProgressReceiver();
    IntentFilter intentFilter = new IntentFilter();
    intentFilter.addAction(SERVICE_RECEIVER);
    registerReceiver(mReceiver, intentFilter);
    
**UpdateActivity** 获取到消息时更新进度条

	public class ProgressReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            int progress = intent.getIntExtra(Constants.UPDATE_DOWNLOAD_PROGRESS, 0);
            Log.i(TAG, "progress......" + progress);
            mProgressBar.setProgress(progress);

            if (mProgressBar.getVisibility() != View.VISIBLE) {
                mProgressBar.setVisibility(View.VISIBLE);
                mDownloadBtn.setVisibility(View.GONE);
            }
        }
    }
    
下载完成后安装apk

	File apkFile = downloadFile;
	installAPk(apkFile);
	
*downloadFile* 为下载完成保存到本地的apk文件，*installAPk* 的具体实现

	private void installAPk(File apkFile) {
        Intent intent = new Intent(Intent.ACTION_VIEW);
        //如果没有设置SDCard写权限，或者没有sdcard,apk文件保存在内存中，需要授予权限才能安装
        try {
            String[] command = {"chmod", "777", apkFile.toString()};
            ProcessBuilder builder = new ProcessBuilder(command);
            builder.start();
        } catch (IOException ignored) {
        }
        intent.setDataAndType(Uri.fromFile(apkFile), "application/vnd.android.package-archive");

        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        startActivity(intent);
    }
    
Android本身自带接口，当打开**.apk**格式的文件时跳转到安装界面，主要实现为这行代码

	intent.setDataAndType(Uri.fromFile(apkFile), "application/vnd.android.package-archive");
	
安装界面

![image5](/images/posts/OtaApp/5.jpg)

若安装文件有问题，系统会提示解析错误

![image6](/images/posts/OtaApp/6.jpg)


### 完全更新

为了区分完全更新和增量更新，服务器返回的json数据增加一个**diffUpdate**字段，用来区分文件是.apk文件还是差分包文件

修改服务器的info数据

	var info = {
        'url': '/ota_file/update.zip',
        'updateMessage': 'Fix bugs.',
        'versionName': 'v2',
        'md5': '',
        'diffUpdate': true                        
	};
	
完全更新时，**diffUpdate** 值设为 **true**
**url** 设置为 **/ota_file/update.zip**

应用做些修改，修改主界面的背景色，增加一行代码

	android:background="@color/colorAccent"
	

![image7](/images/posts/OtaApp/7.jpg)

对应的项目版本信息修改

![image8](/images/posts/OtaApp/8.jpg)

然后运行Android Studio **Build Apk**，生成的Apk文件在项目目录下

![image9](/images/posts/OtaApp/9.jpg)

将apk文件拷贝到 ota服务器项目的 **ota_file** 文件夹下，并修改文件名为 **update.zip**

然后将修改的代码回退，运行旧的版本，点击版本更新，下载更新文件完成后进行安装

![image10](/images/posts/OtaApp/10.jpg)

安装完成后，重新打开应用，便可以看到主界面的背景颜色改变了。


### 增量更新

增量更新的原理，就是将手机上已安装apk与服务器端最新apk进行二进制对比，得到差分包，用户更新程序时，只需要下载差分包，并在本地使用差分包与已安装apk，合成新版apk。

apk文件的差分、合成可以通过[开源的二进制比较工具 bsdiff](http://www.daemonology.net/bsdiff/)实现。

#### 处理差分包的代码实现

项目中引入第三方动态库 **libApkPatchLibrary.so**

引入对应的包 com.cundong.utils.PatchUtils

(差分包更新的实现参考了[这篇文章](https://github.com/cundong/SmartAppUpdates)，其中也用到了里面的文件)

**DownloadService** 下载完成时，根据服务器提供的 **diffUpdate** 字段来判断是否为差分包文件，若是，则将差分包文件与当前的apk合成新的apk，并安装新的apk

安装前合成代码的实现

	if (isDiff) {
		 // 增量式升级，先将patch合成新apk
		 String oldApkPath = InfoUtils.getBaseApkPath(getApplicationContext());
		 String newApkName = "update.apk";
		 String newApkPath = dir.getPath() + "/" + newApkName;
		 String patchPath = downloadFile.getPath();
		
		 Log.i(TAG, "MD5:");
		 Log.i(TAG, "old apk md5: " + SignUtils.getMd5ByFile(new File(oldApkPath)));
		 Log.i(TAG, "new apk md5: " + SignUtils.getMd5ByFile(new File(newApkPath)));
		 Log.i(TAG, "patch md5: " + SignUtils.getMd5ByFile(new File(patchPath)));
		
		 Log.i(TAG, "Patch diff...");
		 int patchResult = PatchUtils.patch(oldApkPath, newApkPath, patchPath);
		 if (patchResult == 0) {
		     apkFile = new File(newApkPath);
		 }
	}
	
	installAPk(apkFile);
	
应用当前的apk会保存在系统特定的位置，通过 ***getBaseApkPath*** 方法获取路径，方法的具体实现

	public static String getBaseApkPath(Context context) {
        String pkName = context.getPackageName();
        try {
            ApplicationInfo appInfo = context.getPackageManager().getApplicationInfo(pkName, 0);
            String path = appInfo.sourceDir;
            return path;
        } catch (PackageManager.NameNotFoundException e) {

        }

        return null;
    }

**PatchUtils.patch** 方法将旧apk文件与差分包文件合成新apk，并保存到newApkPath路径下。

然后和完全更新的方式一样，通过 **installAPk(apkFile);** 安装更新。

#### 生成差分包文件

首先安装差分包生成工具，下载地址为[http://www.daemonology.net/bsdiff/](http://www.daemonology.net/bsdiff/)

安装完成后，试着在命令行敲 **bsdiff** 命令

	bsdiff
	bsdiff: usage: bsdiff oldfile newfile patchfile
	
可看到该命令接收3个参数，依次为*旧的文件*、*新的文件*、*生成的差分包文件*

和之前Build Apk方式一样，先将未修改前的应用编一个apk文件，命名为old.apk

然后修改应用的背景色和版本信息，重新编一个apk文件，命名为new.apk

将新旧两个apk文件拷贝到同一目录下，命令行cd到当前目录下，运行命令 

	old.apk new.apk patch.zip
	
完成后可看到在当前目录下生成了新文件 **patch.zip**，将**patch.zip**拷贝到服务器的**ota_file**文件夹下

服务器返回的信息做对应修改

	var info = {
        'url': '/ota_file/patch.zip',
        'updateMessage': 'Fix bugs.',
        'versionName': 'v2',
        'md5': '',
        'diffUpdate': true                        
	};
	
**diffUpdate** 改为 **true**，**url** 改为差分包文件

#### 运行

之前的运行都是通过Android Studio直接连手机真机跑起来的，但是调试差分包升级时需要注意，差分包必须保证本地应用的apk文件与生成差分包时的old.apk文件完全一致，否则合成新的apk会失败。

而每次通过Android Studio连接手机编起来的应用所生成的apk文件都是不一样的，虽然代码一模一样。

为了测试差分包，手机必须通过旧的apk来安装并运行应用

用 **adb** 命令安装

	adb install -r old.apk 
	
结果打印

	[100%] /data/local/tmp/old.apk
	pkg: /data/local/tmp/old.apk
	Success
	
安装完成后运行，和之前一样操作的流程。

end。

### 其他

实际生产中的升级过程还会涉及到很多逻辑处理细节，比如下载完成之后的MD5校验，下载前判断本地是否存在更新文件等，但基本的更新方式和流程如上所示。

Demo附完整源码，源码里还附了一些工具类处理一些细节。

运行时只需按照文章之前所示，修改服务器信息即可进行相应的升级方式。

[Demo源码地址](https://github.com/moshuqi/DemoCodes/tree/master/OTA)
	


