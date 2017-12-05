---

layout: single
title: Jenkins+Fastlane搭建iOS集成测试环境
date: 2017-06-14

---

### 推荐插件
* Environment Injector Plugin

### 常见问题及解决办法
* *如果使用了fastlane工具，遇到找不到fastlane command not found*
    * 加上$PATH环境变量，如PATH=$PATH:/path/to/fastlane
* *如果执行shell脚本的时候遇到错误: Invalid byte sequence in US-ASCII*
    * 在系统管理->全局属性中设置环境变量LC_ALL=en_US.UTF-8
* *如果编译时遇到错误: Couldn't find specified scheme ‘XXX’*
	 * 在工程设置中将scheme设置为share
* *WARNING: clock of the subversion server appears to be out of sync. This can result in inconsistent check out behavior.*
    * 在repo URL后面加上@HEAD
* *No certificate matching ‘XXXX' 没有找到签名文件*
    * 首先打开keychain keys 找到apple 的开发者证书。然后复制。 再选择左边的系统（system）把刚复制的证书放进去。
* *Xcode 8.3 以后 xcodebuild 命令 没有 -exportProvisioningProfile 和 -exportFormat 两个参数了*
    * 用-exportOptionsPlist 制定plist配置文件替代
* *如果忘记Admin初始密码*
	 * 查看文件 ~/.jenkins/secrets/initialAdminPassword
* *xcodebuild 打包失败*
	 * 执行打包前执行 security create-keychain -p "password" "~/Library/Keychains/login.keychain-db"
