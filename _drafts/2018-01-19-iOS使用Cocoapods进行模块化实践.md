---

layout: single
title: iOS使用Cocoapods进行模块化实践
date: 2018-01-19
category: iOS

---

#### 遇到的问题
1. 头文件找不到	
	* 需要在工程中将头文件设置为public
2. framework 报错`Include of non-modular header inside framework module 'XXX'`;
	*   