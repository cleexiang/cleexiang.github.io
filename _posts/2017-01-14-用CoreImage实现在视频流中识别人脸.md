---

layout: single
title: 用CoreImage实现在视频流中识别人脸
date: 2017-01-14

---

最近做一个功能: 需要从一段视频中尽可能识别到包含人脸的镜头，
第一反应是用iOS系统自带的CoreImage框架，相信很多开发者都已经了解或者使用过CoreImage中的CIImage, CIFilter, CIDetector等类进行图片的处理，处理视频的场景相对较少。其实，相对于处理照片来说，处理视频只是多了一个时间的维度，需要更加关注性能方面。
以一个每秒30帧1分钟的视频为例，如果按高精度逐帧识别的策略的话，大概需要60\*30\*0.18=324秒(注：每帧需耗费0.18s)，这个时间显然是没法接受的，于是我从以下两个大的方面进行了优化，希望能从识别精度和时间上达到一个平衡。

>  识别单张图片的优化
 
* 设置识别精度，精度越高识别越准确，但是耗费的时间也越多。
* 开启追踪模式，但是有可能导致识别的结果不准确
* 先缩小原图
* 二值化，但是貌似并没有提高速度(如果有同学知道原因麻烦回复一下)
* 设置最大Feature点数目
* 设置最小识别结果的size

> 识别帧选取的策略

* 在介绍策略之前，先说一下两种得到图像数据的方法：
	* 使用AVAssetReader和AVAssetReaderTrackOutput进行逐帧读取。
	* 使用AVAssetImageGenerator为某一个时间点生成一张缩略图。
	* 第一种方法适合逐帧检测的场景，第二种方法适合跳帧检测的场景。

* 怎么选取合适的策略？
	* 如果视频总时长比较短（比如10s以内的），建议采用逐帧检测的方法，因为这样能保证尽可能的识别到人脸。
	* 如果视频总时长比较长，建议采用逐帧检测和跳帧检测结合的方法，具体的做法是：设置一个间隔m，每隔m帧，进行一次人脸检测，如果检测到了人脸，则采用逐帧检测策略，直到检测不到人脸，继续采用跳帧检测。这样既能保证识别时间不会过长，又能尽可能的识别到更多的人脸。

> 补充几点

* 发现在调用CIDetector的features方法时，会有内存泄露，不知道是不是苹果的一个Bug，解决办法：用`autoreleasepool` 将整个调用包起来，就不会产生内存泄漏了。
* 将识别结果缓存到沙盒里，可以避免重复扫描，进一步优化了时间，目前我是这么做的。


