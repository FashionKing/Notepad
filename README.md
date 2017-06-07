# Notepad
# 一点笔记

compile 'pub.devrel:easypermissions:0.3.0'
Easypermissions主要简化了对权限申请结果的处理和判断，直接以接口的方式回调处理结果。不需要再自行进行处理。

compile 'com.belerweb:pinyin4j:2.5.1'
是一个可以将汉字转换成拼音的lib，非常实用

compile 'com.makeramen:roundedimageview:2.1.1'
Android 将图片快速转换成圆角的方法

compile 'com.google.zxing:core:3.3.0'
二维码扫一扫

compile 'com.github.chrisbanes.photoview:library:1.2.4'
功能特性
支持放缩超出边界，多点触控和双击事件
滚动和滑动
和ViewPager等能完美兼容
矩阵变化等有回调，方便前台其他展示的改变
单击，长按都有回调提醒


我一行代码都不写实现Toolbar!你却还在封装BaseActivity?
http://www.jianshu.com/p/75a5c24174b2
GitHub: https://github.com/JessYanCoding


通用 IM 聊天 UI 组件，已经同时支持 Android/iOS。 
https://github.com/jpush/aurora-imui

https://github.com/qibin0506亓斌

/**
 * 获取手机上apk文件信息类，主要是判断是否安装再手机上了，安装的版本比较现有apk版本信息
 * @author  Dylan
 */
public class ApkSearchUtils {
	private static int INSTALLED = 0; // 表示已经安装，且跟现在这个apk文件是一个版本
	private static int UNINSTALLED = 1; // 表示未安装
	private static int INSTALLED_UPDATE =2; // 表示已经安装，版本比现在这个版本要低，可以点击按钮更新

	private Context context;
	private List<MyFile> myFiles = new ArrayList<MyFile>();

	public List<MyFile> getMyFiles() {
		return myFiles;
	}

	public void setMyFiles(List<MyFile> myFiles) {
		this.myFiles = myFiles;
	}

	public ApkSearchUtils(Context context) {
		super();
		this.context = context;
	}

	/**
	 * @param args
	 *            运用递归的思想，递归去找每个目录下面的apk文件
	 */
	public void FindAllAPKFile(File file) {

		// 手机上的文件,目前只判断SD卡上的APK文件
		// file = Environment.getDataDirectory();
		// SD卡上的文件目录
		if (file.isFile()) {
			String name_s = file.getName();
			MyFile myFile = new MyFile();
			String apk_path = null;
			// MimeTypeMap.getSingleton()
			if (name_s.toLowerCase().endsWith(".apk")) {
				apk_path = file.getAbsolutePath();// apk文件的绝对路劲
				// System.out.println("----" + file.getAbsolutePath() + "" +
				// name_s);
				PackageManager pm = context.getPackageManager();
				PackageInfo packageInfo = pm.getPackageArchiveInfo(apk_path, PackageManager.GET_ACTIVITIES);
				ApplicationInfo appInfo = packageInfo.applicationInfo;

				
				 /**获取apk的图标 */
				appInfo.sourceDir = apk_path;
				appInfo.publicSourceDir = apk_path;
				Drawable apk_icon = appInfo.loadIcon(pm);
				myFile.setApk_icon(apk_icon);
				/** 得到包名 */
				String packageName = packageInfo.packageName;
				myFile.setPackageName(packageName);
				/** apk的绝对路劲 */
				myFile.setFilePath(file.getAbsolutePath());
				/** apk的版本名称 String */
				String versionName = packageInfo.versionName;
				myFile.setVersionName(versionName);
				/** apk的版本号码 int */
				int versionCode = packageInfo.versionCode;
				myFile.setVersionCode(versionCode);
				/**安装处理类型*/
				int type = doType(pm, packageName, versionCode);
				myFile.setInstalled(type);
				
				Log.i("ok", "处理类型:"+String.valueOf(type)+"\n" + "------------------我是纯洁的分割线-------------------");
				myFiles.add(myFile);
			}
			// String apk_app = name_s.substring(name_s.lastIndexOf("."));
		} else {
			File[] files = file.listFiles();
			if (files != null && files.length > 0) {
				for (File file_str : files) {
					FindAllAPKFile(file_str);
				}
			}
		}
	}

	/*
	 * 判断该应用是否在手机上已经安装过，有以下集中情况出现 
	 * 1.未安装，这个时候按钮应该是“安装”点击按钮进行安装
	 * 2.已安装，按钮显示“已安装” 可以卸载该应用 
	 * 3.已安装，但是版本有更新，按钮显示“更新” 点击按钮就安装应用 
	 */
	
	/**
	 * 判断该应用在手机中的安装情况
	 * @param pm                   PackageManager  
	 * @param packageName  要判断应用的包名
	 * @param versionCode     要判断应用的版本号
	 */
	private int doType(PackageManager pm, String packageName, int versionCode) {
		List<PackageInfo> pakageinfos = pm.getInstalledPackages(PackageManager.GET_UNINSTALLED_PACKAGES);
		for (PackageInfo pi : pakageinfos) {
			String pi_packageName = pi.packageName;
			int pi_versionCode = pi.versionCode;
			//如果这个包名在系统已经安装过的应用中存在
			if(packageName.endsWith(pi_packageName)){
				//Log.i("test","此应用安装过了");
				if(versionCode==pi_versionCode){
					Log.i("test","已经安装，不用更新，可以卸载该应用");
					return INSTALLED;
				}else if(versionCode>pi_versionCode){
					Log.i("test","已经安装，有更新");	
					return INSTALLED_UPDATE;
				}
			}
		}
		Log.i("test","未安装该应用，可以安装");	
		return UNINSTALLED;
	}
	
}
