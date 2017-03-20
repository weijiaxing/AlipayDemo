# AlipayDemo
###接入流程
一：公司开通支付宝商户号
二：注册登录蚂蚁金服开放平台创建应用
三：项目集成支付宝SDK

####一：公司开通支付宝商户号
1 注册商户号：https://mobiless.alipay.com/  可以把链接发给人事，让她弄一下，需要提交的资料也只能由人事来弄。
![这里写图片描述](http://img.blog.csdn.net/20161118090804924)
![这里写图片描述](http://img.blog.csdn.net/20161118090830972)
![这里写图片描述](http://img.blog.csdn.net/20161118090841707)

####二：注册登录蚂蚁金服开放平台创建应用
蚂蚁金服开放平台：https://open.alipay.com/platform/home.htm  
创建成功并上线是如下面：
![这里写图片描述](http://img.blog.csdn.net/20161118091623562)

1 创建应用
![这里写图片描述](http://img.blog.csdn.net/20161118092115950)

2 应用环境设置
![这里写图片描述](http://img.blog.csdn.net/20161118092646686)

应用公钥配置  点击支付宝的秘钥生成器
![这里写图片描述](http://img.blog.csdn.net/20161118092908656)
![这里写图片描述](http://img.blog.csdn.net/20161118093740312)
![这里写图片描述](http://img.blog.csdn.net/20161118093715862)
![这里写图片描述](http://img.blog.csdn.net/20161118093930626)

AES秘钥点击生成即可
![这里写图片描述](http://img.blog.csdn.net/20161118094112691)

上线和添加支付功能
![这里写图片描述](http://img.blog.csdn.net/20161118094426521)

选择APP支付，确定后会提示需要签约，进行签约就是了
![这里写图片描述](http://img.blog.csdn.net/20161118094614198)

点击上线，签约涉及到公司信息的就交给人事去弄吧


####三：项目中集成支付宝SDK
先上文档：
https://doc.open.alipay.com/docs/doc.htm?spm=a219a.7629140.0.0.DjzzBP&treeId=193&articleId=105296&docType=1
![这里写图片描述](http://img.blog.csdn.net/20161118095535320)
按着文档一步一步的做就行了

修改Manifest
在商户应用工程的AndroidManifest.xml文件里面添加声明：

```
<activity
            android:name="com.alipay.sdk.app.H5PayActivity"
            android:configChanges="orientation|keyboardHidden|navigation"
            android:exported="false"
            android:screenOrientation="behind" >
</activity>
<activity
            android:name="com.alipay.sdk.auth.AuthActivity"
            android:configChanges="orientation|keyboardHidden|navigation"
            android:exported="false"
            android:screenOrientation="behind" >
 </activity>
```
和权限声明：

```
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```
添加混淆规则
在商户应用工程的proguard-project.txt里添加以下相关规则：

```
-libraryjars libs/alipaySDK-20150602.jar
 
-keep class com.alipay.android.app.IAlixPay{*;}
-keep class com.alipay.android.app.IAlixPay$Stub{*;}
-keep class com.alipay.android.app.IRemoteServiceCallback{*;}
-keep class com.alipay.android.app.IRemoteServiceCallback$Stub{*;}
-keep class com.alipay.sdk.app.PayTask{ public *;}
-keep class com.alipay.sdk.app.AuthTask{ public *;}
```
至此，开发包开发资源导入完成。

支付接口调用
需要在新线程中调用支付接口。（可参考alipay_demo实现）
获取PayTask支付对象调用支付（支付行为需要在独立的非ui线程中执行），代码示例：

```
final String orderInfo = info;   // 订单信息
 
        Runnable payRunnable = new Runnable() {
 
            @Override
            public void run() {
                PayTask alipay = new PayTask(DemoActivity.this);
                String result = alipay.payV2(orderInfo,true);
 
                Message msg = new Message();
                msg.what = SDK_PAY_FLAG;
                msg.obj = result;
                mHandler.sendMessage(msg);
            }
        };
         // 必须异步调用
        Thread payThread = new Thread(payRunnable);
        payThread.start();
```

支付结果获取和处理
调用pay方法支付后，将通过2种途径获得支付结果：

同步返回
商户应用客户端通过当前调用支付的Activity的Handler对象，通过它的回调函数获取支付结果。（可参考alipay_demo实现）
代码示例：

```
private Handler mHandler = new Handler() {
        public void handleMessage(Message msg) {
            Result result = new Result((String) msg.obj);
            Toast.makeText(DemoActivity.this, result.getResult(),
                        Toast.LENGTH_LONG).show();
        };
    };
```
异步通知
商户需要提供一个http协议的接口，包含在请求支付的入参中，其key对应notify_url。支付宝服务器在支付完成后，会以POST方式调用notify_url传输数据。

获取当前开发包版本号
调用PayTask对象的getVersion()方法查询。
代码示例：

```
PayTask payTask = new PayTask(activity);
String version = payTask.getVersion();
```

在支付类添加
APPID
PID
商户私钥，pkcs8格式(在支付宝商户里面配置一下)

![这里写图片描述](http://img.blog.csdn.net/20161118101242373)

![这里写图片描述](http://img.blog.csdn.net/20161118143730849)
![这里写图片描述](http://img.blog.csdn.net/20161118143741303)
![这里写图片描述](http://img.blog.csdn.net/20161118143756319)
![这里写图片描述](http://img.blog.csdn.net/20161118143805210)

这样运行Demo就可以成功调用支付宝了，是不是感觉挺简单的呢，这是博主第二次做个移动支付了，第一次做的是用第三方SDK Ping++接入的，一次性集成了支付宝和微信支付，跳槽换了一家公司，虽然第三方SDK接入支付简单，Ping++号称只需一行代码，毕竟这世上没有免费的午餐，所以这次决定单独接入，有时间会带来Android微信支付的接入详解。

Demo github链接：https://github.com/weijiagithub/AlipayDemo
