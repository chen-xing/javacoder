# HTML播放rtmp（ckplayer）

ckplayer直播或者回播，播放类型为mp4，hls，rtmp，rtsp，目前这些测试过，代码如下：

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>实现rtmp流的直播播放-https://www.94rg.com/</title>
    <script type="text/javascript" src="http://file.94rg.com/html/ckplayer/ckplayer/ckplayer.js"></script>
</head>
<body>
<div id="video" class="video" style="width: 1000pxi; height: 800px;"></div>

<script type="text/javascript">
var videoObject = {
				container: '#video', //容器的ID或className
				variable: 'player',//播放函数名称
				autoplay:true,
				live:true,
				video: 'rtmp://rtmp01open.ys7.com/openlive/f01018a141094b7fa138b9d0b856507b.hd'
			};
			var player = new ckplayer(videoObject);
</script>
</body>
</html>


```

视频播放需要flash或者HTML自带播放器，ckplayer默认自动选择，也可在ckplayer.js中config函数中设置。视频播放根据HTML中div来加载播放位置，根据div属性width和height设置video窗口大小。视频画面大小下边说明。

函数参数说明：

```
container：#video表示div id，
.video表示className variable：播放函数名称
autoplay：true表示自动播放，false为视频播放器加载完显示暂停状态
live：true，直播或回播方式
video：视频地址
```

视频画面设置参数在ckplayerConfig函数中

1. 去掉或者修改右上角logo，style下logo方法中设置file参数为图片路径

2. style下video函数为视频比例，默认16:9，根据div大小设置，此处修改不影响点击全屏画面，只修改当前div中画面大小比例，如需修改左下角时间显示，需要修改配置文件language.xml中第30行

   

   ```
   <live>
   			 [$liveTimeY]-[$liveTimem]-[$liveTimed]
   </live>
   ```

   

关于控制进度条点击事件，修改js中播放方式为回放，config中48行

```
buttonMode: {
				player: false,//鼠标在播放器上是否显示可点击形态
				controlBar: false,//鼠标在控制栏上是否显示可点击形态
				timeSchedule: true,//鼠标在时间进度条上是否显示可点击形态
				volumeSchedule: true //鼠标在音量调节栏上是否显示可点击形态
			}
```

如果需要显示视频列表，播放上一集和下一集，需要绑定front（上一集）和next（下一集）事件。
直播可以设置延迟加载视频时间，单位：毫秒，默认30，我自己有时候播放亏导致播放器崩溃，可能播放时识别用HTML播放的，重新加载之后播放一段时间显示false溃崩了，更新到最新版本之后，发现问题还是会重现，于是现在改了时间为200ms，目前没有出现过，不知道是否为这个原因。

在线效果预览:http://file.94rg.com/html/ckplayer/94rg.html

