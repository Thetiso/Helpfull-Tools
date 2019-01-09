
## 七牛图片/文件上传
- 参考ebooking下OrderUploadInfo服务     
- js和jar包依赖在svn://svn.efreight.cn/dev/efreight2/Training/CommonModule

### 1. WEB-Front
#### 1.1 微信
1. 引入js文件qiniuUploader.js
2. 如下调用方法：
```
uploadToQN:function(token){
    let that = this
    let options = {
      region: 'NCN', // 华北区
      uptoken: token,
        // uptoken: 'xxx',
      domain: 'http://XXX.YYY.com',
      shouldUseQiniuFileName: true
    };
    qiniuUploader.init(options);
    qiniuUploader.upload(localPath, (res) => {
      that.setData({
        finalQiNiuKey: res.key
      })
      console.log('上传成功，准备提交到后台，之后返回前面的页面')
      that.addAudioCmt()
    }, (error) => {
      console.error('error: ' + JSON.stringify(error));
    },
    // , {
    //     region: 'NCN', // 华北区
    //     uptokenURL: 'https://[yourserver.com]/api/uptoken',
    //     domain: 'http://[yourBucketId].bkt.clouddn.com',
    //     shouldUseQiniuFileName: false
    //     key: 'testKeyNameLSAKDKASJDHKAS'
    //     uptokenURL: 'myServer.com/api/uptoken'
    // }
    null,// 可以使用上述参数，或者使用 null 作为参数占位符
    (progress) => {
        console.log('上传进度', progress.progress)
        console.log('已经上传的数据长度', progress.totalBytesSent)
        console.log('预期需要上传的数据总长度', progress.totalBytesExpectedToSend)	
    }, cancelTask => that.setData({cancelTask})
    );
},
```
3. 其中token由后台来生产，通过api接口来获取

#### 1.2 纯js
https://github.com/qiniu/js-sdk
```
let config = {
  	useCdnDomain: true,
  	region: null
};
	let observable = qiniu.upload(file, null, token, null, config)
let observer = {
  	next(res){
    	progressCallBack(res.total.percent)
  	},
  	error(err){
    	console.log(err)
    	failCallBack(err)
  	},
  	complete(res){
    	successCallback(res.key)
  	}
}
let subscription = observable.subscribe(observer)
```

### 2. JAVA-Back
1. 引入jar依赖,或者直接添加包
```
<dependency>
  	<groupId>com.qiniu</groupId>
  	<artifactId>qiniu-java-sdk</artifactId>
  	<version>[7.2.0, 7.2.99]</version>
</dependency>
```
2. 以controller为例，生产token  `jdk 1.7`
```
public interface QiNiu {
	static String ACCESS_KEY = "sGOH10tmR67yQfbSxQjhh5sKSgXYxn1ZSCS0UFW1";
	static String SECRET_KEY = "L7_JS3XP7B3Ucv59o_FlT3IYbVyNNacjbb_Prx-Y";
	static String HH_BUCKET = "hothit";
}

...


import com.qiniu.util.Auth;

@RestController
@RequestMapping("/qn")
public class QNController {

	static String accessKey = QiNiu.ACCESS_KEY;
	static String secretKey = QiNiu.SECRET_KEY;
	static String bucket = QiNiu.HH_BUCKET;
	static Auth auth = Auth.create(accessKey, secretKey);
//	String upToken = auth.uploadToken(bucket);
	
	@ResponseBody
	@PostMapping("/upload/token/new")
	public ApiResult createNewUploadToken(@RequestParam("userId") int userId) {
		if (userId < 1) {
			return ApiResult.error(ExceptionCode.PARAMETER_WRONG);
		}
		String upToken = auth.uploadToken(bucket);
		if (StringUtils.isEmpty(upToken)) {
			return ApiResult.error(ExceptionCode.UNKNOWN_EXCEPTION);
		}
		return ApiResult.ok(upToken);
	}
	
}
```

如果使用的是jdk1.6得使用6.1.9的七牛SDK
```
import com.qiniu.api.auth.digest.Mac;
import com.qiniu.api.config.Config;
import com.qiniu.api.rs.PutPolicy;
public class Uptoken {
	public static void main(String[] args) throws Exception {
		Config.ACCESS_KEY = "<YOUR APP ACCESS_KEY>";
		Config.SECRET_KEY = "<YOUR APP SECRET_KEY>";
		Mac mac = new Mac(Config.ACCESS_KEY, Config.SECRET_KEY);
		// 请确保该bucket已经存在
		String bucketName = "Your bucket name";
		PutPolicy putPolicy = new PutPolicy(bucketName);
		String uptoken = putPolicy.token(mac);
	}
}
```



