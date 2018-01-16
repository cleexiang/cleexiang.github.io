---

layout: single
title: Fastlane高级用法之fastfile重用
date: 2018-01-16
category: fastlane

---

### 背景

*在项目中，我们常常有很多公用的组件通过fastlane来发布，而每个组件都需要配置一个fastfile进行部署，一般情况下这些fastfile都大同小异，例如我们团队之前几乎所有组件的fastfile大概是这样的*

```
lane :deploy do
  git_pull
	ensure_git_branch(
  		branch: 'dev'
	)
	version = version_bump_podspec(path: "XXXX.podspec")
  	git_commit(message: "Bumped to version #{version}")
  	add_git_tag(
  		tag: version
	)
  	push_to_git_remote(remote_branch: 'dev', force: false, tags: true)
  	changelog = changelog_from_git_commits
  	sh("git fetch --tags")
 	pod_push(
 		path: "XXXX.podspec",
 		repo: "XXXX-specs",
 		allow_warnings: true, 
 		verbose: true
 	)
 end

```
*这样的操作会存在两个问题：*
 
1. 如果需要对fastfile中的某个步骤进行修改或者添加一个步骤，需要对所有组件的fastfile都修改一遍。
2. 团队成员如果新加一个组件的话，大家偷偷懒又从之前的fastfile拷贝一份出来改下自己用。

### 解决方案
> Fastlane官方提供了一个action([import\_from\_git](https://docs.fastlane.tools/actions/import_from_git/))，可以通过git的repo来访问远程的个一个fastfile。

*具体操作方法：*

1. 将通用的fastlane抽取为公用的方法，变量通过参数的形式传入。
2. 在git仓库中创建一个repo来专门存放这个通用的fastfile.
3. 然后在各自组建中通过`import_from_git` 来访问公用的fastlane。

以下为公用的fastlane组件

```
lane :deploy_component do |options|
  branch = options[:branch]
  project = options[:project]
  repo = options:[:repo]

  ensure_git_branch(branch: branch)
  version = version_bump_podspec(path: "#{project}.podspec")
  git_commit(message: "Bumped to version #{version}")
  add_git_tag(tag: version)
  push_to_git_remote(remote_branch: branch, force: false, tags: true)
  pod_lib_lint(allow_warnings: true)
  pod_push(
    path: "#{project}.podspec",
    repo: repo,
    allow_warnings: true, 
    verbose: true
  )
end
```

以下为组件中的fastfile中的使用:

```
default_platform :ios
import_from_git(
  url: "http://path/to/Fastfiles.git", # The URL of the repository to import the Fastfile from.
  branch: "master", # The branch to checkout on the repository. Defaults to `HEAD`.
  path: "fastlane/ios_fastfile.rb" # The path of the Fastfile in the repository. Defaults to `fastlane/Fastfile`.
)

lane :deploy do
  deploy_component(
    branch:"master",
    project:"XXXX",
    repo:"XXXX",
  )
end

```

### 总结
*这样通过以上方法能够很好的重利用fastfile，并且能够让团队的所有成员更方便的集成fastlane。*

