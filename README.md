## wcs-android-sdk

ANDROID SDK基于网宿云存储API规范构建，适用于ANDROID。使用此SDK构建您的移动APP，能让您非常便捷地将数据安全地存储到网宿云平台上。

- [下载链接](https://wcs.chinanetcenter.com/document/SDK/wcs-android-sdk#下载链接)
- [移动端场景演示](https://wcs.chinanetcenter.com/document/SDK/wcs-android-sdk#移动端场景演示)
- [使用指南](https://wcs.chinanetcenter.com/document/SDK/wcs-android-sdk#使用指南) 
  [准备开发环境](https://wcs.chinanetcenter.com/document/SDK/wcs-android-sdk#准备开发环境)
[配置信息](https://wcs.chinanetcenter.com/document/SDK/wcs-android-sdk#配置信息)
[文件上传](https://wcs.chinanetcenter.com/document/SDK/wcs-android-sdk#文件上传)

## 工程介绍

[wcs-android-sdk](https://github.com/Wangsu-Cloud-Storage/wcs-android-sdk/tree/master/wcs-android-sdk)

工程源码

[app](https://github.com/Wangsu-Cloud-Storage/wcs-android-sdk)

demo 例子

## 



### 下载链接

[wcs-android-sdk下载链接](https://wcsd.chinanetcenter.com/sdk/cnc-android-sdk-wcs.zip)

### 移动端场景演示

1) 移动端向企业自建WEB服务端请求上传凭证 
2) 企业自建WEB服务端将构建好的上传凭证返回移动端 
3) 移动端调用网宿云存储平台提供的接口上传文件 
4) 网宿云存储在检验凭证合法性后，执行移动端请求的接口逻辑，最终返回给移动端处理结果 
![移动端场景演示](https://wcs.chinanetcenter.com/indexNew/image/wcs/wcs-android-sdk1.png)

### 使用指南

#### 准备开发环境

一、移动端开发环境准备 
1)把wcs-android-sdk-x.x.x.jar复制到项目里libs目录下； 

![复制到项目里libs目录](https://wcs.chinanetcenter.com/indexNew/image/wcs/wcs-android-sdk2.png)

2)如果使用的ADT插件是16及以上，则会自动把jar放到Android Dependencies中，并自动完成后续的jar包的导入；如果不是，请继续浏览第3步；

3)右键选择工程，选择Properties； 

4)点击Java Build Path->Libraries； 

5)点击Add Jars，选择工程libs目录下的wcs-android-x.x.x.jar； 

6)点击OK。 

![移动开发环境准备2](https://wcs.chinanetcenter.com/indexNew/image/wcs/wcs-android-sdk3.png)


7)配置网络权限：AndroidManifest.xml添加入<uses-permission android:name="android.permission.INTERNET"/>。

二、服务端开发环境准备 

在工程中引入SDK的wcs-java-sdk-x.x.x.jar包，和wcs-java-sdk-x.x.x-dependencies.zip文件中解压出来的第三方jar包（以eclipse为例） 
![服务端开发环境准备](https://wcs.chinanetcenter.com/indexNew/image/wcs/wcs-android-sdk4.png)


#### 配置信息

用户接入网宿云存储时，需要使用一对有效的AK和SK进行签名认证，并填写“上传域名”进行文件上传。配置信息只需要在整个应用程序中初始化一次即可，具体操作如下：

&emsp;&emsp;开通网宿云存储平台账户

&emsp;&emsp;登录网宿云存储平台，在“安全管理”下的“密钥管理”查看AK和SK，“域名查询”查看上传、管理域名。

在获取到AK和SK等信息之后，您可以按照如下方式进行信息初始化：

    import com.chinanetcenter.api.util.Config;
    //1.初始化AK&SK信息
    String ak = "your access key";
    String sk = "your secrete key";
    String PUT_URL = "your uploadDomain";
    Config.init(ak,sk,PUT_URL,"");

#### 文件上传

<1>表单上传时可开启returnurl进行页面跳转，其他情况下建议不设置returnurl。

<2>若文件大小超过2M，建议使用分片上传 

<3>云存储提供的上传域名为普通域名，若对上传速度较为敏感，有要求的客户建议采用网宿上传加速服务。


1.普通上传（POST方式） 
用户在上传文件后，上传返回结果由云存储平台统一控制，规范统一化。

&emsp;&emsp;如果用户指定上传策略数据的returnUrl，网宿云存储将反馈一个指向returnUrl的HTTP 303，驱动客户端执行跳转；

&emsp;&emsp;如果用户没指定上传策略数据的returnUrl，网宿云存储根据returnbody的设定向客户端发送反馈信息。

**范例：**

移动端代码：

    /**
     * 上传接口范例
     */
    private void uploadFile(File srcFile) {
    /**
             * UPLOADER_TOKEN-上传凭证
             * srcFile-本地待上传的文件
             */
            FileUploader.upload(UPLOADER_TOKEN, srcFile, new FileUploaderListener() {
    
                @Override
                public void onSuccess(int status, JSONObject responseJson) {
                    Log.d(TAG, "responseJson : " + responseJson.toString());
                }
    
                @Override
                public void onFailure(OperationMessage operationMessage) {
                    Log.e(TAG, "errorMessage : " + operationMessage.toString());
                }
    
                @Override
                public void onProgress(int bytesWritten, int totalSize) {
                    Log.d(TAG, String.format("Progress %d from %d (%s)", bytesWritten, totalSize, (totalSize > 0) ? ((float) bytesWritten / totalSize) * 100 : -1));
                }
            });
        }

服务端代码（生成上传凭证）：

    import com.chinanetcenter.api.domain.PutPolicy;
    import com.chinanetcenter.api.util.DateUtil;
    import com.chinanetcenter.api.util.TokenUtil;
    import com.chinanetcenter.api.util.Config;
    
        ......
    
        public String uploadToken() {
    //初始化AK&SK信息
            String ak = "your access key";
            String sk = "your secrete key";
            Config.init(ak, sk);
            PutPolicy putPolicy = new PutPolicy();
            //设置scope（空间:文件名）
            putPolicy.setScope("viptest:sz1001");
            //设置请求过期时间
            long time = DateUtil.parseDate("2099-01-01 00:00:00", DateUtil.COMMON_PATTERN).getTime();
            putPolicy.setDeadline(String.valueOf(time));
            String uploadToken = TokenUtil.getUploadToken(putPolicy);
            return uploadToken;
        }

2.回调上传（POST方式） 

用户上传文件后，对返回给客户端的信息进行自定义格式。 
使用该上传模式需要启用上传策略数据的callbackUrl参数,而callbackBody参数可选（建议使用该参数）。 
*注意：returnUrl和callbackUrl不能同时指定。*

&emsp;&emsp;如果指定了callbackBody参数，云存储将接收此参数，并向callbackUrl指定的地址发起一个HTTP请求回调业务服务器，同时向业务服务器发送数据。发送的数据内容由callbackBody指定。业务服务器完成回调的处理后，可以在HTTP Response中放入数据，网宿云存储会响应客户端，并将业务服务器反馈的数据发送给客户端。
如果不指定callbackBody参数，云存储将返回空串给客户端。

**范例：**

移动端代码：

     /**
         * 上传接口范例
         */
    private void uploadFile(File srcFile) {
    /**
             * UPLOADER_TOKEN-上传凭证
             * srcFile-本地待上传的文件
             */
            FileUploader.upload(UPLOADER_TOKEN, srcFile, new FileUploaderListener() {
    
                @Override
                public void onSuccess(int status, JSONObject responseJson) {
                    Log.d(TAG, "responseJson : " + responseJson.toString());
                }
    
                @Override
                public void onFailure(OperationMessage operationMessage) {
                    Log.e(TAG, "errorMessage : " + operationMessage.toString());
                }
    
                @Override
                public void onProgress(int bytesWritten, int totalSize) {
                    Log.d(TAG, String.format("Progress %d from %d (%s)", bytesWritten, totalSize, (totalSize > 0) ? ((float) bytesWritten / totalSize) * 100 : -1));
                }
            });
    
        }
        
服务端代码（生成上传凭证）：

    import com.chinanetcenter.api.domain.PutPolicy;
    import com.chinanetcenter.api.util.DateUtil;
    import com.chinanetcenter.api.util.TokenUtil;
    import com.chinanetcenter.api.util.Config;
    
        ......
    
        public String uploadToken() {
    //初始化AK&SK信息
            String ak = "your access key";
            String sk = "your secrete key";
            Config.init(ak, sk);
            PutPolicy putPolicy = new PutPolicy();
            //设置scope（空间:文件名）
            putPolicy.setScope("viptest:sz1001");
            //设置请求过期时间
            long time = DateUtil.parseDate("2099-01-01 00:00:00", DateUtil.COMMON_PATTERN).getTime();
            putPolicy.setDeadline(String.valueOf(time));
            //设置回调URL
            putPolicy.setCallbackUrl(callbackUrl);
            //设置回调信息格式（如果不想自定义格式，该处设置为NULL即可）
            putPolicy.setCallbackBody(callbackBody);
            String uploadToken = TokenUtil.getUploadToken(putPolicy);
            return uploadToken;
        }

3.通知上传 (POST方式) 

用户在上传文件的同时，提交文件处理指令，请求网宿云存储平台对上传的文件进行处理。由于处理操作较耗时，为了不影响客户端的体验，网宿云存储平台采用异步处理策略，处理完成后将结果自动通知客户服务端。 
使用该上传模式需要启用上传策略数据的persistentOps参数和persistentNotifyUrl参数。

**范例：**

移动端代码：

    /**
         * 上传接口范例
         */
    private void uploadFile(File srcFile) {
    /**
             * UPLOADER_TOKEN-上传凭证
             * srcFile-本地待上传的文件
             */
            FileUploader.upload(UPLOADER_TOKEN, srcFile, new FileUploaderListener() {
    
                @Override
                public void onSuccess(int status, JSONObject responseJson) {
                    Log.d(TAG, "responseJson : " + responseJson.toString());
                }
    
                @Override
                public void onFailure(OperationMessage operationMessage) {
                    Log.e(TAG, "errorMessage : " + operationMessage.toString());
                }
    
                @Override
                public void onProgress(int bytesWritten, int totalSize) {
                    Log.d(TAG, String.format("Progress %d from %d (%s)", bytesWritten, totalSize, (totalSize > 0) ? ((float) bytesWritten / totalSize) * 100 : -1));
                }
            });
    
        }

服务端代码（生成上传凭证）：

    import com.chinanetcenter.api.domain.PutPolicy;
    import com.chinanetcenter.api.util.DateUtil;
    import com.chinanetcenter.api.util.TokenUtil;
    import com.chinanetcenter.api.util.Config;
    
        ......
    
        public String uploadToken() {
    //初始化AK&SK信息
            String ak = "your access key";
            String sk = "your secrete key";
            Config.init(ak, sk);
            PutPolicy putPolicy = new PutPolicy();
            //设置scope（空间:文件名）
            putPolicy.setScope("viptest:sz1001");
            //设置请求过期时间
            long time = DateUtil.parseDate("2099-01-01 00:00:00", DateUtil.COMMON_PATTERN).getTime();
            putPolicy.setDeadline(String.valueOf(time));
            //设置异步数据处理指令（参考附录一的处理指令介绍-目前支持视频类的异步处理，如果不需要异步处理，不建议使用该方式做上传）
            putPolicy.setPersistentOps(cmd); 
            //设置异步处理后回调的URL
            putPolicy.setPersistentNotifyUrl(notifyUrl); 
    String uploadToken = TokenUtil.getUploadToken(putPolicy);
            return uploadToken;
        }

4.分片上传（POST方式） 

移动端上传大文件需要耗费较长时间，一旦在传输过程中出现异常，文件内容需全部重传，影响用户体验，为避免这个问题，引进分片上传机制。 

分片上传机制是将一个大文件切分成多个自定义大小的块，然后将这些块并行上传，如果某个块上传失败，客户端只需要重新上传这个块即可。 

*注意：每个块的最大大小不能超过4M，超过4M的设置，将采用默认最大4M切分；最小不能小于1M，小于1M，将会采用1M去切分。*

**范例**

移动端代码：

    private static final long DEFAULT_BLOCK_SIZE = 1 * 1024 * 1024;
    /**
    * context-应用当前的上下文
    * uploadToken-上传Token
    * ipaFile-需要上传的文件
    * DEFAULT_BLOCK_SIZE-块大小
    */
    FileUploader.sliceUpload(context, uploadToken, ipaFile, DEFAULT_BLOCK_SIZE, new SliceUploaderListener() {
              @Override
              public void onSliceUploadSucceed(JSONObject jsonObject) {
    Log.d("CNCLog", "slice upload succeeded.");
    }
    
    @Override
    public void onSliceUploadFailured(OperationMessage operationMessage) {
    Log.d("CNCLog", "slice upload failured.");
    }
    
    @Override
    public void onProgress(long uploaded, long total) {
    Log.d("CNCLog", String.format(Locale.CHINA, "uploaded : %s, total : %s", uploaded, total));
    }
    });

服务端代码：

    import com.chinanetcenter.api.domain.PutPolicy;
    import com.chinanetcenter.api.util.DateUtil;
    import com.chinanetcenter.api.util.TokenUtil;
    import com.chinanetcenter.api.util.Config;
    
        ......
    
        public String uploadToken() {
    //初始化AK&SK信息
            String ak = "your access key";
            String sk = "your secrete key";
            Config.init(ak, sk);
            PutPolicy putPolicy = new PutPolicy();
            //设置scope（空间:文件名）
            putPolicy.setScope("viptest:sz1001");
            //设置请求过期时间
            long time = DateUtil.parseDate("2099-01-01 00:00:00", DateUtil.COMMON_PATTERN).getTime();
            putPolicy.setDeadline(String.valueOf(time));
            //设置异步数据处理指令（参考附录一的处理指令介绍-目前支持视频类的异步处理，如果不需要异步处理，不建议使用该方式做上传）
            putPolicy.setPersistentOps(cmd); 
            //设置异步处理后回调的URL
            putPolicy.setPersistentNotifyUrl(notifyUrl); 
    String uploadToken = TokenUtil.getUploadToken(putPolicy);
            return uploadToken;
        }
