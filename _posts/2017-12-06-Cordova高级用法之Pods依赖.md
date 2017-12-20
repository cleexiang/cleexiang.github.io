---

layout: single
title: Cordova高级用法之Pods依赖
date: 2017-12-06

---

#### *背景*
> 如果当前开发的Cordova插件需要依赖外部第三方库或者想公用私有组件。

#### *解决方案*

使用插件[cordova-plugin-cocoapods-support](https://github.com/blakgeek/cordova-plugin-cocoapods-support)

使用方法：

* 先安装插件 `cordova plugin add cordova-plugin-cocoapod-support --save
`
* 然后再你自己的插件配置plugins.xml中添加以下配置,
更多配置方法可以参考官方文档。

```
   <platform name="ios">
        <pods-config ios-min-version="9.0" use-frameworks="true">
            <source url="git@github.com:foo/foo-specs.git"/>
        </pods-config>
        <pod name="JSONKit" version="1.0.0" />
    </platform>
    
```

通过以上两个步骤就能实现在安装你开发的插件时，自动将pods依赖的添加到工程中。
说完怎么使用，再来看下插件`cordova-plugin-cocoapods-support`本身是怎么实现的。从plugins.xml中不难看出，关键在于两个hook的js。

* `<hook type="before_plugin_install" src="scripts/npmInstall.js"/>`
* `<hook type="after_prepare" src="scripts/podify.js"/>`
