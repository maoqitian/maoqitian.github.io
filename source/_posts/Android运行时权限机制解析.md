---
title: Android运行时权限机制解析
date: 2019-01-19 00:14:42
categories:  
- Android基础回顾 #分类
tags:
- Android
- Android运行时权限
- permission
---
>从Android M（6.0 API级别23）开始，用户开始在应用运行时向其授予权限，而不是在应用安装时授予。此方法可以简化应用安装过程，因为用户在安装或更新应用时不需要授予权限。
<!--more-->
### 权限介绍
- 权限机制的目的是保护用户的隐私，Android应用程序必须请求访问敏感用户数据（如联系人和短信）以及某些系统特性（如照相机和互联网）的许可。根据该特征，系统可以自动授予许可，或者提示用户批准请求。
- 如果设备运行的是 Android 6.0 或更高版本，或者应用的目标 SDK 为 23 或更高：应用必须在清单文件中中列出权限，并且它必须在运行时请求其需要的每项危险权限。用户可以授予或拒绝每项权限，且即使用户拒绝权限请求，应用仍可以继续运行有限的功能。


### 权限类别 
  
#### 正常权限
- 正常权限不会给用户的隐私带来风险，如果你在清单文件（AndroidManifest.xml）中加入了正常权限声明，则安卓系统会自动授予App应用该权限，如下列出了正常权限
  
```
   android.permission.ACCESS_LOCATION_EXTRA_COMMANDS
   android.permission.ACCESS_NETWORK_STATE
   android.permission.ACCESS_NOTIFICATION_POLICY
   android.permission.ACCESS_WIFI_STATE
   android.permission.ACCESS_WIMAX_STATE
   android.permission.BLUETOOTH
   android.permission.BLUETOOTH_ADMIN
   android.permission.BROADCAST_STICKY
   android.permission.CHANGE_NETWORK_STATE
   android.permission.CHANGE_WIFI_MULTICAST_STATE
   android.permission.CHANGE_WIFI_STATE
   android.permission.CHANGE_WIMAX_STATE
   android.permission.DISABLE_KEYGUARD
   android.permission.EXPAND_STATUS_BAR
   android.permission.FLASHLIGHT
   android.permission.GET_ACCOUNTS
   android.permission.GET_PACKAGE_SIZE
   android.permission.INTERNET
   android.permission.KILL_BACKGROUND_PROCESSES
   android.permission.MODIFY_AUDIO_SETTINGS
   android.permission.NFC
   android.permission.READ_SYNC_SETTINGS
   android.permission.READ_SYNC_STATS
   android.permission.RECEIVE_BOOT_COMPLETED
   android.permission.REORDER_TASKS
   android.permission.REQUEST_INSTALL_PACKAGES
   android.permission.SET_TIME_ZONE
   android.permission.SET_WALLPAPER
   android.permission.SET_WALLPAPER_HINTS
   android.permission.SUBSCRIBED_FEEDS_READ
   android.permission.TRANSMIT_IR
   android.permission.USE_FINGERPRINT
   android.permission.VIBRATE
   android.permission.WAKE_LOCK
   android.permission.WRITE_SYNC_SETTINGS
   com.android.alarm.permission.SET_ALARM
   com.android.launcher.permission.INSTALL_SHORTCUT
   com.android.launcher.permission.UNINSTALL_SHORTCUT
```
#### 危险权限
- 查看危险权限
    
```
    可以通过adb shell pm list permissions -d -g进行查看(Windows使用adb 命令首先需要自行配置环境变量)
```
![adb查看危险权限](https://github.com/maoqitian/MaoMdPhoto/raw/master/Android%20%E8%BF%90%E8%A1%8C%E6%97%B6%E6%9D%83%E9%99%90/adb%20%E6%9F%A5%E7%9C%8B%E5%8D%B1%E9%99%A9%E6%9D%83%E9%99%90.png)
    
    - 危险权限会授予应用访问用户隐私数据的权限。如果您的应用在清单中列出了正常权限，系统将自动授予该权限。如果您列出了危险权限，则用户在清单文件中列出的同时还必须在触发使用相应功能的时候让用户同意应用使用这些权限
    
    ![危险权限列表](https://github.com/maoqitian/MaoMdPhoto/raw/master/Android%20%E8%BF%90%E8%A1%8C%E6%97%B6%E6%9D%83%E9%99%90/%E5%8D%B1%E9%99%A9%E6%9D%83%E9%99%90.png)

- 由危险权限的表可以看出，危险权限都是一组一组出现的，并且你只要授予一组权限的其中一个，那么该组危险权限的其他权限也同样被授予了（例如，如果某应用已经请求并且被授予了READ_CONTACTS 权限，然后它又请求WRITE_CONTACTS，系统将立即授予该权限）
#### 使用权限
- 当我们新建一个Android应用的时候，默认应用是没有有申请任何权限的，我们不管需要什么权限，首先需在清单文件中使用<uses-permission>标签声明
```
  <manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.android.app.myapp" >
    <uses-permission android:name="android.permission.RECEIVE_SMS" />
    ...
   </manifest>
```
### 权限相关API
#### 检查权限
- 当我们应用需要危险的权限的时候，每次执行操作都需要检查是否授予了危险权限，检查是否具有该权限我们使用ContextCompat.checkSelfPermission() 方法
```
   int permissionCheck = ContextCompat.checkSelfPermission(thisActivity,
        Manifest.permission.WRITE_CALENDAR);
        
   //如果具有该权限，则方法返回PackageManager.PERMISSION_GRANTED，并且应用可以
   //继续操作。如果应用不具有此权限，方法将返回 PERMISSION_DENIED，且应用必须明确向用户要求权限        
```
#### 请求获取权限
- 当我们应用某个功能操作需要危险权限的申请，则我们可以调用ActivityCompat.requestPermissions的方法来获取相应的权限。该方法异步运行：它会立即返回，并且在用户响应对话框之后，系统会使用结果调用应用的回调方法，将应用传递的相同请求代码传递到ActivityCompat.requestPermissions方法。
- 以下代码可以检查应用是否具备读取用户联系人的权限，并根据需要请求该权限
```
  if (ContextCompat.checkSelfPermission(thisActivity,
                Manifest.permission.READ_CONTACTS)
        != PackageManager.PERMISSION_GRANTED) {
    //是否具有该读取联系人权限
    
    if (ActivityCompat.shouldShowRequestPermissionRationale(thisActivity,
            Manifest.permission.READ_CONTACTS)) {

        //判断是否需要向用户解释，为什么需要这些权限。有时候用户会不理解应用程序为什么需要这些权限。
        //这个方法只有在APP请求过某一权限且用户禁止APP使用该权限的时候返回true。在用户授权了权限和禁止权限时勾选了“Don't ask again”选项的情况下都会返回false
        //也就是说如果进入到这里，就说明该权限曾经被拒绝过
        
        /**
        被拒绝又想再次申请可跳转应用信息界面授予权限
        Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
        intent.setData(Uri.fromParts("package", context.getPackageName(), null));
        startActivity(intent);
        */

    } else {

        //申请权限

        ActivityCompat.requestPermissions(thisActivity,
                new String[]{Manifest.permission.READ_CONTACTS},
                MY_PERMISSIONS_REQUEST_READ_CONTACTS);

        // MY_PERMISSIONS_REQUEST_READ_CONTACTS is an
        // app-defined int constant. The callback method gets the
        // result of the request.
    }
  }
```
#### 处理权限请求的响应
- 当权限申请提示框与用户交互的时候，我们开发人员必须知道用户到底是否同意应用的权限申请。所以用户响应时，Android系统将调用应用的 onRequestPermissionsResult() 方法，向其传递用户响应 
- 延续上面读取联系人的例子
    
```
    /**
     * 
     * @param requestCode 申请权限传入的请求码
     * @param permissions 申请权限的数组
     * @param grantResults 请求的结果（用于区分上一个参数permissions中的权限有没有被授予，permissions和grantResults两个数组大小是一样的，具体值和上方提到的PackageManager中的两个常量做比较）
     */
    @Override
    public void onRequestPermissionsResult(int requestCode,
        String permissions[], int[] grantResults) {
     switch (requestCode) {
        case MY_PERMISSIONS_REQUEST_READ_CONTACTS: {
            // If request is cancelled, the result arrays are empty.
            if (grantResults.length > 0
                && grantResults[0] == PackageManager.PERMISSION_GRANTED) {

                // permission was granted, yay! Do the
                // contacts-related task you need to do.
                //用户同意权限申请，继续应用操作

            } else {

                // permission denied, boo! Disable the
                // functionality that depends on this permission.
                //用户拒绝权限申请，提示用户应用的操作需要该权限
            }
            return;
        }

        // other 'case' lines to check for other
        // permissions this app might request
       }
      }
```
#### 运行时权限小例子
- 读取联系人列表完整例子
```
      public class MainActivity extends AppCompatActivity {

        private static final int MY_PERMISSIONS_REQUEST_READ_CONTACTS = 1;
         //保存联系人
        public List<String> list = new ArrayList<>();

        @Override
        protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        readContacts();
        }
        //读取联系人
        public void readContacts(){
        if(ContextCompat.checkSelfPermission(this, Manifest.permission.READ_CONTACTS)
                != PackageManager.PERMISSION_GRANTED){
          //申请权限
            //申请权限
            if (ActivityCompat.shouldShowRequestPermissionRationale(this,
                    Manifest.permission.READ_CONTACTS)) {
                //之前申请权限的时候拒绝过 ，向用户解释为什么需要该权限
                Toast.makeText(MainActivity.this, "Permission Denied，Show an expanation to the user *asynchronously* -- don't block", Toast.LENGTH_SHORT).show();
            }else {
                ActivityCompat.requestPermissions(this,
                        new String[]{Manifest.permission.READ_CONTACTS},
                        MY_PERMISSIONS_REQUEST_READ_CONTACTS);
            }

          }else {
            //获取联系人列表
            list=readContact();
          }
        }

        @Override
        public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        if(requestCode == MY_PERMISSIONS_REQUEST_READ_CONTACTS){
            if (grantResults[0] == PackageManager.PERMISSION_GRANTED)
            {
                //获取联系人列表
                list=readContact();
            } else {
                // Permission Denied
                Toast.makeText(MainActivity.this, "Permission Denied", Toast.LENGTH_SHORT).show();
            }
            return;
        }
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
       }

       //读取联系人
       public List<String> readContact(){
        List<String> list = new ArrayList<>();
        Cursor cursor = null;
        try {
            //cursor指针 query询问 contract协议 kinds种类
            cursor = getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI, null, null, null, null);
            if (cursor != null) {
                while (cursor.moveToNext()) {
                    String displayName = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
                    String number = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
                    list.add(displayName + '\n' + number);
                }
            }
            return list;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (cursor != null) {
                cursor.close();
            }
        }
        return null;
       }
     }
     
     
    //清单文件中别忘了加上 
    <uses-permission android:name="android.permission.READ_CONTACTS"/>
```
### 热门框架了解
- AndPermission（严振杰大大的框架）
- 项目地址
    [https://github.com/yanzhenjie/AndPermission](https://github.com/yanzhenjie/AndPermission)
- 该框架也是日常开发用得比较多的一个框架，该框架一句话搞定权限申请，还是比较方便的
```
    AndPermission.with(this).runtime()
                .permission(Permission.Group.STORAGE)
                .onGranted(new Action<List<String>>() {
                    @Override
                    public void onAction(List<String> data) {

                    }
                }).onDenied(new Action<List<String>>() {
            @Override
            public void onAction(List<String> data) {

            }
        }).start();
```
- 通过阅读框架源码，可以看到这个框架的核心就是PermissionActivity,它是一个没有界面的Activity,所有的权限申请都由它来发起，并进行相应操作的回调，结合前面了解的运行时权限的知识，相信这个Activity你可以很轻易就了解。
```
       /**
        * <p>
        * Request permission.
        * </p>
        * Created by Yan Zhenjie on 2017/4/27.
        */
       public final class PermissionActivity extends Activity {

       private static final String KEY_INPUT_OPERATION = "KEY_INPUT_OPERATION";
       private static final int VALUE_INPUT_PERMISSION = 1;
       private static final int VALUE_INPUT_PERMISSION_SETTING = 2;
       private static final int VALUE_INPUT_INSTALL = 3;
       private static final int VALUE_INPUT_OVERLAY = 4;
       private static final int VALUE_INPUT_ALERT_WINDOW = 5;

        private static final String KEY_INPUT_PERMISSIONS = "KEY_INPUT_PERMISSIONS";

        private static RequestListener sRequestListener;

        /**
         * Request for permissions.
         */
        public static void requestPermission(Context context, String[] permissions, RequestListener requestListener) {
        PermissionActivity.sRequestListener = requestListener;

        Intent intent = new Intent(context, PermissionActivity.class);
        intent.putExtra(KEY_INPUT_OPERATION, VALUE_INPUT_PERMISSION);
        intent.putExtra(KEY_INPUT_PERMISSIONS, permissions);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(intent);
        }

        /**
         * Request for setting.
         */
        public static void permissionSetting(Context context, RequestListener requestListener) {
        PermissionActivity.sRequestListener = requestListener;

        Intent intent = new Intent(context, PermissionActivity.class);
        intent.putExtra(KEY_INPUT_OPERATION, VALUE_INPUT_PERMISSION_SETTING);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(intent);
        }

        /**
         * Request for package install.
         */
        public static void requestInstall(Context context, RequestListener requestListener)  {
        PermissionActivity.sRequestListener = requestListener;

        Intent intent = new Intent(context, PermissionActivity.class);
        intent.putExtra(KEY_INPUT_OPERATION, VALUE_INPUT_INSTALL);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(intent);
        }

        /**
         * Request for overlay.
        */
        public static void requestOverlay(Context context, RequestListener requestListener) {
        PermissionActivity.sRequestListener = requestListener;

        Intent intent = new Intent(context, PermissionActivity.class);
        intent.putExtra(KEY_INPUT_OPERATION, VALUE_INPUT_OVERLAY);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(intent);
        }

        /**
        * Request for alert window.
        */
        public static void requestAlertWindow(Context context, RequestListener requestListener) {
        PermissionActivity.sRequestListener = requestListener;

        Intent intent = new Intent(context, PermissionActivity.class);
        intent.putExtra(KEY_INPUT_OPERATION, VALUE_INPUT_ALERT_WINDOW);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(intent);
        }

        @Override
        protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Intent intent = getIntent();
        int operation = intent.getIntExtra(KEY_INPUT_OPERATION, 0);
        switch (operation) {
            case VALUE_INPUT_PERMISSION: {
                String[] permissions = intent.getStringArrayExtra(KEY_INPUT_PERMISSIONS);
                if (permissions != null && sRequestListener != null) {
                    requestPermissions(permissions, VALUE_INPUT_PERMISSION);
                } else {
                    finish();
                }
                break;
            }
            case VALUE_INPUT_PERMISSION_SETTING: {
                if (sRequestListener != null) {
                    RuntimeSettingPage setting = new RuntimeSettingPage(new ContextSource(this));
                    setting.start(VALUE_INPUT_PERMISSION_SETTING);
                } else {
                    finish();
                }
                break;
            }
            case VALUE_INPUT_INSTALL: {
                if (sRequestListener != null) {
                    Intent manageIntent = new Intent(Settings.ACTION_MANAGE_UNKNOWN_APP_SOURCES);
                    manageIntent.setData(Uri.fromParts("package", getPackageName(), null));
                    startActivityForResult(manageIntent, VALUE_INPUT_INSTALL);
                } else {
                    finish();
                }
                break;
            }
            case VALUE_INPUT_OVERLAY: {
                if (sRequestListener != null) {
                    OverlaySettingPage settingPage = new OverlaySettingPage(new ContextSource(this));
                    settingPage.start(VALUE_INPUT_OVERLAY);
                } else {
                    finish();
                }
                break;
            }
            case VALUE_INPUT_ALERT_WINDOW: {
                if (sRequestListener != null) {
                    AlertWindowSettingPage settingPage = new AlertWindowSettingPage(new ContextSource(this));
                    settingPage.start(VALUE_INPUT_ALERT_WINDOW);
                } else {
                    finish();
                }
                break;
            }
            default: {
                throw new AssertionError("This should not be the case.");
            }
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        if (sRequestListener != null) {
            sRequestListener.onRequestCallback();
        }
        finish();
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (sRequestListener != null) {
            sRequestListener.onRequestCallback();
        }
        finish();
    }

    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_BACK) {
            return true;
        }
        return super.onKeyDown(keyCode, event);
    }

    @Override
    public void finish() {
        sRequestListener = null;
        super.finish();
    }

    /**
     * permission callback.
     */
    public interface RequestListener {
        void onRequestCallback();
    }
   }
```
### 最后说点
- 看过不一定记得，记得不一定会写，会写不代表不会忘记，所以记下来是最好的选择。好了，又通过一篇文章让我对Android运行时权限有了更深入的了解。文章中如果有错误，请大家给我提出来，大家一起学习进步，如果觉得我的文章给予你帮助，也请给我一个喜欢和关注。
- 参考链接:
    - 《Android进阶之光》
    - [在运行时请求权限](https://developer.android.google.cn/training/permissions/requesting#handle-response)
    
    - [系统权限](https://developer.android.google.cn/guide/topics/security/permissions#permissions)
