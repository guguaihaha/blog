lerna帮助项目多包管理
---

git的仓库地址如下：[https://github.com/lerna/lerna](https://github.com/lerna/lerna)


将大型代码库拆分为独立的独立版本包对于代码共享非常有用。 然而，在许多存储库中进行更改是麻烦和难以跟踪的事情。为了解决这些（和许多其他）问题，一些项目将它们的代码库组织成多包存储库。 像 Babel、React、Angular、Ember、Meteor、Jest 等等。

Lerna 是一个优化使用 git 和 npm 管理多包存储库的工作流工具，用于管理具有多个包的 JavaScript 项目。

**Lerna 仓库是什么样子？**

您有一个如下所示的文件系统：

```text
my-lerna-repo/
  package.json
  packages/
    package-1/
      package.json
    package-2/
      package.json
```

**Lerna 能做什么？**

Lerna 中的两个主要命令是 lerna bootstrap 和 lerna publish。 bootstrap 将把 repo 中的依赖关系链接在一起。 publish 将有助于发布软件包更新。

> 工作的两种模式

**固定模式**

固定模式，通过lerna.json的版本进行版本管理。当你执行lerna publish命令时， 如果距离上次发布只修改了一个模块，将会更新对应模块的版本到新的版本号，然后你可以只发布修改的库。

这种模式也是Babel使用的方式。如果你希望所有的版本一起变更， 可以更新minor版本号，这样会导致所有的模块都更新版本。

**独立模式**

独立模式，init的时候需要设置选项 --independent. 独立模式允许管理者对每个库单独改变版本号，每次发布的时候，你需要为每个改动的库指定版本号。这种情况下， lerna.json的版本号不会变化了， 默认为independent。


> 快速开始

##### 安装和初始化

```text
    $ npm install lerna -g
    $ mkdir lerna-gp && cd $_
    $ npm lerna init # 用的默认的固定模式，vue babel等都是这个
    
    # Add packages
    $ cd packages
    $ mkdir daybyday gpnode gpwebpack
    ...
    #分别进入三个目录初始化成包
    $ cd daybyday
    $ npm init -y 
    $ cd ../gpnode
    $ npm init -y
    $ cd ../gpwebpack
    $ npm init -y
```

> 命令集

```text
lerna publish
lerna version
lerna bootstrap
lerna list
lerna changed
lerna diff
lerna exec
lerna run
lerna init
lerna add
lerna clean
lerna import
lerna link
lerna create
lerna info
```

##### 必要设置

确保以下状态都是登录态

```text
✗ git remote add origin git@gitlab.yourSite.com:gaopo/lerna-gp.git

#查看是否登录
✗ npm whoami
gp0320

#没有则登录 
npm login 
# 输入username password 
Logged in as gp0320 on https://registry.npmjs.org/. # succeed
```
##### 必要设置

> 默认是npm, 而且每个子package都有自己的node_modules，通过这样设置后，只有顶层有一个node_modules

修改顶层 package.json and lerna.json

package.json 文件加入

```json
{
  "private": true,
  "workspaces": [
    "packages/*"
  ]
}
```
lerna.json 文件加入

```text
"useWorkspaces": true,
"npmClient": "yarn",
```

##### lerna的语法

> lerna create <name> [loc]

创建一个包，name包名，loc 位置可选

**Examples**

```text
# 根目录的package.json 
 "workspaces": [
    "packages/*",
    "packages/@gp0320/*"
  ],
  
# 创建一个包gpnote默认放在 workspaces[0]所指位置
lerna create gpnote 

# 创建一个包gpnote指定放在 packages/@gp0320文件夹下，注意必须在workspaces先写入packages/@gp0320，看上面
lerna create gpnote packages/@gp0320
```

> lerna add <package>[@version] [--dev] [--exact]

增加本地或者远程package做为当前项目packages里面的依赖

+ --dev devDependencies 替代 dependencies

+ --exact 安装准确版本，就是安装的包版本前面不带^, Eg: "^2.20.0" ➜ "2.20.0"

**Examples**

```text
# Adds the module-1 package to the packages in the 'prefix-' prefixed folders
lerna add module-1 packages/prefix-*

# Install module-1 to module-2
lerna add module-1 --scope=module-2

# Install module-1 to module-2 in devDependencies
lerna add module-1 --scope=module-2 --dev

# Install module-1 in all modules except module-1
lerna add module-1

# Install babel-core in all modules
lerna add babel-core
```

> lerna bootstrap

默认是npm i,因为我们指定过yarn，so,run yarn install,会把所有包的依赖安装到根node_modules.

> lerna list

列出所有的包，如果与你文夹里面的不符，进入那个包运行yarn init -y解决

```text
➜  lerna-gp git:(master) ✗ lerna list
lerna notice cli v3.14.1
daybyday
gpnode
gpnote
gpwebpack
lerna success found 4 packages
```

> lerna import <path-to-external-repository>

导入本地已经存在的包


> lerna run

```text
lerna run < script > -- [..args] # 运行所有包里面的有这个script的命令
$ lerna run --scope my-component test
```

> lerna exec

运行任意命令在每个包

```text
$ lerna exec -- < command > [..args] # runs the command in all packages
$ lerna exec -- rm -rf ./node_modules
$ lerna exec -- protractor conf.js
lerna exec --scope my-component -- ls -la
```

> lerna link

删除所有包的node_modules目录

> lerna changed

列出下次发版lerna publish 要更新的包。

原理：
需要先git add,git commit 提交。
然后内部会运行git diff --name-only v版本号 ，搜集改动的包，就是下次要发布的。并不是网上人说的所有包都是同一个版全发布。

```text
➜  lerna-repo git:(master) ✗ lerna changed                                     
info cli using local version of lerna
lerna notice cli v3.14.1
lerna info Looking for changed packages since v0.1.4
daybyday #只改过这一个 那下次publish将只上传这一个
lerna success found 1 package ready to publish
```

> lerna publish

会打tag，上传git,上传npm。
如果你的包名是带scope的例如："name": "@gp0320/gpwebpack",
那需要在packages.json添加

```json
{
 "publishConfig": {
    "access": "public"
  }
}
```

```text
lerna publish 
lerna info current version 0.1.4
#这句意思是查找从v0.1.4到现在改动过的包
lerna info Looking for changed packages since v0.1.4 

? Select a new version (currently 0.1.4) Patch (0.1.5)

Changes:
 - daybyday: 0.1.3 => 0.1.5 #只改动过一个

...

Successfully published:
 - daybyday@0.1.5
lerna success published 1 package
```
