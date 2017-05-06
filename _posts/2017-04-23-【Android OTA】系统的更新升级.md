---
layout: post
title:  【Android OTA】系统的更新升级
category: "Android"
---

上一篇说了Android应用的升级，这篇来介绍下关于整个系统的升级。公司的车载系统使用了MTK的板子，深度定制的Android系统，平时开发过程中的修改可以直接重新烧录固件，但设备量产投入市场之后的修改只能通过OTA的方式进行更新。

系统升级一般通过差分包的方式，因为整包更新需要的安装包有几百M，一般的小更新打个差分包的话也就几M十几M而已，能省不少流量。

设备检测更新的流程基本和[上一篇文章](https://moshuqi.github.io/2017/04/22/Android-OTA-应用的更新升级/)一样，系统可以在设置界面上增加检测升级的入口。


#### 系统升级流程

1.	向服务器请求系统更新信息；
2.	获取到返回信息后，判断是否有更新版本，若有更新，进入步骤3；
3.	下载服务器对应的更新文件；
4.	下载完成后验证更新文件；
5.	系统重启进入recovery模式进行升级，升级完成后设备重启。

更新信息的请求和返回信息的处理过程与[应用的更新升级](https://moshuqi.github.io/2017/04/22/Android-OTA-应用的更新升级/)一致。

更新文件的下载这次使用了系统自带的类 **DownloadManager**。

**DownloadManager** 能自动管理系统的下载任务，在联网发生变化、系统开关机、断点续传等情况自行进行处理。

创建下载任务，设置指定下载目录

	mDownloadManager = (DownloadManager)getSystemService(DOWNLOAD_SERVICE);
	
	DownloadManager.Request request = new DownloadManager.Request(Uri.parse(url));
	request.setDestinationInExternalPublicDir(OTAUtil.otaFolderName(), OTAUtil.otaFileName());
	request.setNotificationVisibility(DownloadManager.Request.VISIBILITY_HIDDEN);
	request.setVisibleInDownloadsUi(false);
	mDownloadId = mDownloadManager.enqueue(request);
	
**DownloadManager** 下载进度可以显示在通知栏中，也可以选择只在特定联网情况下（如wifi、移动信号）进行任务。

监听下载状态

	class CompleteReceiver extends BroadcastReceiver {

		@Override
		public void onReceive(Context context, Intent intent) {
			long completeDownloadId = intent.getLongExtra(DownloadManager.EXTRA_DOWNLOAD_ID, -1);
			if (completeDownloadId == mDownloadId) {
				// download successful
				String msg;
				switch (OTAUtil.getOTADownloadStatus(completeDownloadId, mDownloadManager)) {
					case DownloadManager.STATUS_FAILED:
						msg = "Download failed!";
						break;

					case DownloadManager.STATUS_PAUSED:
						msg = "Download paused!";
						break;

					case DownloadManager.STATUS_PENDING:
						msg = "Download pending!";
						break;

					case DownloadManager.STATUS_RUNNING:
						msg = "Download in progress!";
						break;

					case DownloadManager.STATUS_SUCCESSFUL:
						break;

					default:
						msg = "Download is nowhere in sight";
						break;
				}

				Log.i(TAG, "CompleteReceiver onReceive...." + msg);
			}
		}
	}
	
用来更新界面的进度条
	
	class DownloadChangeObserver extends ContentObserver {

		public DownloadChangeObserver() {
			super(mHandler);
		}

		@Override
		public void onChange(boolean selfChange) {

			int progress = OTAUtil.getOTADownloadPro(mDownloadId, mDownloadManager);

			Message msg = mHandler.obtainMessage();
			msg.what = UPDATE_DOWNLOAD_PROGRESS;
			msg.arg1 = progress;
			mHandler.sendMessage(msg);
		}
	}
	
其中***getOTADownloadPro*** 方法通过**id**获取指定下载任务的进度值

	public static int getOTADownloadPro(long id, DownloadManager downloadManager) {
        double progress = 0.0;

        DownloadManager.Query q = new DownloadManager.Query();
        q.setFilterById(id);

        Cursor c = downloadManager.query(q);
        if (c.moveToFirst()) {
            int sizeIndex = c.getColumnIndex(DownloadManager.COLUMN_TOTAL_SIZE_BYTES);
            int downloadedIndex = c.getColumnIndex(DownloadManager.COLUMN_BYTES_DOWNLOADED_SO_FAR);
            long size = c.getInt(sizeIndex);
            long downloaded = c.getInt(downloadedIndex);

            if (size != -1) {
                progress = downloaded * 100.0 / size;
            }
        }
        c.close();

        return (int)progress;
    }
    
这里有一个注意问题，正常情况下使用**DownloadManager**下载的文件只能保存在系统的外部存储中（系统级的DownloadManager有个私有方法能将文件保存在内部存储），而调用系统的接口更新升级时，recovery模式下系统文件路径会重定向，此时无法找到原来保存在外部存储的文件。所以为了保证能正常安装，下载好的更新文件要保存到内部存储中。

将更新文件从外部存储拷贝到**/data/recovery/ota.zip**路径下，这里使用了第三方库 **RootTools**，能够以命令行的方式，将文件从源目录拷贝到指定目录。

因为要对内部存储进行读写，所以记得加上对应的权限

	<uses-permission android:name="android.permission.WRITE_INTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.READ_INTERNAL_STORAGE"/>
    
安装更新的函数具体实现

	public static boolean installOtaPackageAuto(final Context context, File file) {

        String internalPath = "/data/recovery/ota.zip";
        String cmd = "cat " + file.getAbsolutePath() + " > " + internalPath;

        String res = Tools.shell(cmd, false);
        Log.i(TAG, "shell: " + res);

        File otaPackageFile = new File(internalPath);

        try {
            RecoverySystem.verifyPackage(otaPackageFile , null , null);
        }
        catch ( IOException e ) {
            ((HDUpdateActivity)context).runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    Toast.makeText(context, "RecoverySystem verifyPackage failed, file doesn't exist", Toast.LENGTH_LONG).show();
                }
            });
            e.printStackTrace( );
            return false;
        }
        catch ( GeneralSecurityException e ) {
            ((HDUpdateActivity)context).runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    Toast.makeText(context, "RecoverySystem verifyPackage failed, invalid package", Toast.LENGTH_LONG).show();
                }
            });
            e.printStackTrace( );
            return false;
        }

        try {
            RecoverySystem.installPackage( context , otaPackageFile );
        }
        catch ( IOException e ) {
            ((HDUpdateActivity)context).runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    Toast.makeText(context, "RecoverySystem installPackage error, failed to install", Toast.LENGTH_LONG).show();
                }
            });
            e.printStackTrace( );
            return false;
        }
        return true;
    }

传入的参数 **file** 为**DownloadManager**已经下载到外部存储的OTA更新文件，通过执行cat命令，将更新文件保存到内部存储**/data/recovery/ota.zip**

	String internalPath = "/data/recovery/ota.zip";
    String cmd = "cat " + file.getAbsolutePath() + " > " + internalPath;
    
	String res = Tools.shell(cmd, false);
	
调用了系统方法***verifyPackage***验证更新包是否合法，若验证失败则安装过程终止

	RecoverySystem.verifyPackage(otaPackageFile , null , null);
	
接下来直接调用系统安装更新的方法

	RecoverySystem.installPackage( context , otaPackageFile );
	
若成功则设备重启，并进入recovery模式更新升级，升级完成后设备重启。

#### 制作差分包

MTK系统的目录

![image1](/images/posts/OtaSystem/1.jpg)

可以看到每次通过**make**命令编译后生成的**image**文件既是 **SP Flash Tool** 烧录需要用到的固件。

ota打包

	make otapackage
	
打包完成后命令行打印的结果

![image2](/images/posts/OtaSystem/2.jpg)

这里有两个文件需要注意的，一个在**out/target/product/CM01B**目录下，一个在**out/target/product/CM01B/obj/PACKAGING/target_files_intermediates**目录下（CM01B为产品编号，每个项目不一样），虽然文件名一样，但前者有400多M，后者800多M，和image固件大小差不多。

**out/target/product/CM01B**目录下的文件为完全更新的包，可以用来给系统进行完全升级。

**out/target/product/CM01B/obj/PACKAGING/target_files_intermediates**目录下的文件主要用来生成差分包（具体能不能直接用来升级没尝试过），通过两个这种包可以生成差分包文件。

系统生成差分包的工具在这个位置，官方标准工具，里面包含了制作差分包的脚本

![image3](/images/posts/OtaSystem/3.jpg)

这里需要注意，执行差分包命令时必须在根目录下执行，因为脚本里面写定了相对路径的引用文件。

制作差分包需要准备两个不同版本的文件（**out/target/product/CM01B/obj/PACKAGING/target_files_intermediates**目录下的），假设分别为**a.zip**和**b.zip**

将两个文件拷贝到根目录下，然后运行命令

	./build/tools/releasetools/ota_from_target_files -i a.zip b.zip ota.zip
	
最终生成的差分包文件为**ota.zip**，即**b**相对于**a**修改了的内容，只能用在**a**上将**a**升级成**b**。ota.zip差分包的升级使用要求当前系统必须是a，否则无法进行升级。

#### 系统升级

系统通过差分包升级，安装时进入刷机界面，完成后自动重启，升级完成

![image4](/images/posts/OtaSystem/4.jpg)

如果差分包有不正确或者没保存到内部存储中会报错

![image5](/images/posts/OtaSystem/5.jpg)



**以上**

源码和设备相关联的，这次就不给了。完。：）