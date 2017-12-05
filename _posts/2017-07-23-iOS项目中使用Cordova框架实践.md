---

layout: single
title: iOS项目中使用Cordova框架实践
date: 2017-07-23

---

## 插件的使用
> Cordova比较有优势的地方就是提供了一套标准接口，这套标准接口可以用于JS和Native之间相互调用，传递数据。

#### 常用命令
* 添加插件
	* cordova plugin add cordova-plugin-camera (仅适用与cordova官方提供的插件)
	* cordova plugin add https://github.com/apache/cordova-plugin-camera.git
	* cordova plugin add /path/to/plugin
* 移除插件
	* cordova plugin remove cordova-plugin-camera

#### 如何自定义插件?
* 安装plugman
	* plugman create --name xxxx --plugin_id com.xxx.xxx --plugin_version 1.0.0 用于创建一个插件所需的标准模板。
* 创建一个插件类，继承与CDVPlugin。
* 修改Plugin.xml配置
	*
* 修改xxx.js，该js用于将原生提供的接口封装成js将要调用的接口。
* 创建package.json文件

## 遇到的坑
* 使用CDVPluginResult回调多个参数</br>

> 正确姿势

```
+ (CDVPluginResult*)resultWithStatus:(CDVCommandStatus)statusOrdinal messageAsMultipart:(NSArray*)theMessages
```

> 错误

```
+ (CDVPluginResult*)resultWithStatus:(CDVCommandStatus)statusOrdinal messageAsArray:(NSArray*)theMessage.
```
 
* 找不到文件:Internal navigation rejected: \<allow-navigation not set\> </br>
**解决办法:** `如果wwwFolderName设置为沙盒目录，路径前面需要加上”file://“前缀`

* 前端页面显示乱码
**解决办法:** `在Header标签中加上<meta http-equiv="Content-Type" content="text/html;charset=utf-8" />`

* 如果想通过CDVViewController加载`http://www.baidu.com`这种外部网页, 需要在config.xml里面配置`<allow-navigation href="http://*/*" />` 白名单。

* 设置`self.automaticallyAdjustsScrollViewInsets = YES`后，导致某些html页面显示有问题，比如穿过statusbar之类，为了兼容性，可以设置`self.automaticallyAdjustsScrollViewInsets = NO`,让scollview穿过statusbar，同事强行设置webview的frame.height为20。可以让webview从距离顶端20px的位置开始显示。