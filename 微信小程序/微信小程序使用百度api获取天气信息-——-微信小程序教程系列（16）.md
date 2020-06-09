下面介绍使用百度api来获取天气信息。



1> 第一步：先到百度开放平台http://lbsyun.baidu.com申请ak
http://lbsyun.baidu.com/index.php?title=wxjsapi/guide/key
![](https://upload-images.jianshu.io/upload_images/19956127-486dcc6ac6fd207b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

申请到ak后，在我的应用里就能查看到
![](https://upload-images.jianshu.io/upload_images/19956127-af6d18112e655b4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2> 第二步：配置你的request合法域名

配置域名请到微信公众平台的后台里设置
![](https://upload-images.jianshu.io/upload_images/19956127-176b1703e595b0ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3> 第三步：下载百度地图的api ，链接：http://download.csdn.net/detail/michael_ouyang/9754015
解压后，里面有2个js文件，一个是常规没压缩的，另一个是压缩过的
PS：由于小程序项目文件大小限制为1M，建议使用压缩版的js文件！
![](https://upload-images.jianshu.io/upload_images/19956127-fc8f3ca7e7992eee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4> 第四步：引入JS模块
在项目根目录下新建一个路径，将百度的js文件拷贝到新建的路径下，完成。
如下图所示，新建路径 "libs/bmap-wx" ，将 bmap-xw.min.js 文件拷贝至 "libs/bmap-wx" 路径下。
![](https://upload-images.jianshu.io/upload_images/19956127-862105b9301fab8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5> 第五步：在所需的js文件内导入js
// 引用百度地图，注意：require传入一个相对路径
var bmap = require('../../libs/bmap-wx/bmap-wx.js'); 


6> 第六步：编辑代码
注意：此处楼主使用的ak是随便写的，同学们需要自行申请！！！
xxx.wxml：
```
<view> 
  <text>{{weatherData}}</text> 
</view>
<view style="padding-top:30px"></view>
<block wx:for="{{futureWeather}}">
	<view style="border:1px solid #ccc; margin:5px">
		<view>{{item.date}}</view>
		<view>{{item.temperature}}</view>
		<view>{{item.weather}}</view>
		<view>{{item.wind}}</view>
	</view>
</block>
```

xxx.js：
```
// 引用百度地图微信小程序JSAPI模块 
var bmap = require('../../libs/bmap-wx/bmap-wx.min.js');
 
Page({
  data:{
  	ak:"FHG7utZtdyXN23W",
  	weatherData:'',
  	futureWeather:[]
  },
  onLoad:function(options){
    var that = this;
    // 新建bmap对象 
    var BMap = new bmap.BMapWX({ 
        ak: that.data.ak 
    }); 
    var fail = function(data) { 
        console.log(data);
    }; 
    var success = function(data) { 
    		console.log(data);
    		
    		var weatherData = data.currentWeather[0]; 
    		var futureWeather = data.originalData.results[0].weather_data;
    		console.log(futureWeather);
        weatherData = '城市：' + weatherData.currentCity + '\n' + 'PM2.5：' + weatherData.pm25 + '\n' +'日期：' + weatherData.date + '\n' + '温度：' + weatherData.temperature + '\n' +'天气：' + weatherData.weatherDesc + '\n' +'风力：' + weatherData.wind + '\n'; 
        that.setData({ 
          weatherData: weatherData,
          futureWeather: futureWeather
        }); 
    } 
		
		// 发起weather请求 
		BMap.weather({ 
		    fail: fail, 
		    success: success 
		}); 
  }
  
})
```
7> 第七步：运行
 ![](https://upload-images.jianshu.io/upload_images/19956127-2a44a5845feb57a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


更多的百度地图api，可到github查看：https://github.com/baidumapapi/wxapp-jsapi

原文作者：michael_ouyang
原文链接：https://blog.csdn.net/michael_ouyang/article/details/55099684
