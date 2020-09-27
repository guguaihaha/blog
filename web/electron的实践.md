electron的实践
---

近段时间以来，开始尝试使用electron去做客户端，因为本人电脑是mac，所以官方建议使用`Xcode`，但是如果使用window建议安装`Visual Studio`，总之目前大部分应用跑在
windows上，至于基于mac、linux等其他的较少。今天分享的是如何快速的启动一款Electron应用。

> 环境配置

当然是要装机Nodejs了，至于如何装机，请移步[Nodejs官网](https://nodejs.org/en/)

> 创建工程

随便创建一个文件夹，然后

```text
npm init
```

npm 会帮助你创建一个基本的 package.json 文件。 其中的 main 字段所表示的脚本为应用的启动脚本，它将会在主进程中执行。 如下片段是一个 package.json 的示例

```text
{
  "name": "your-app",
  "version": "0.1.0",
  "main": "main.js",
  "scripts": {
      "start": "electron ."
    }
}
```

> 安装Electron

以上准备工作做完了，就需要安装万能核心包，基本安装了这个你就到处放肆了（除了已做了一些安全约束考虑之外）。

```text
npm install --save-dev electron
```

以上运行时间比较长，耐心等待即可，然后开始上手使用

> 跑起来程序

在跑程序之前，首先要明白以下几个重要知识点

+ `Electron` 模块所提供的功能都是通过命名空间暴露出来的

+ `Electron` 运行 `package.json` 的 `main` 脚本的进程被称为主进程。 在主进程中运行的脚本通过创建web页面来展示用户界面。 一个 Electron 应用总是有且只有一个主进程。

 由于 Electron 使用了 Chromium 来展示 web 页面，所以 Chromium 的多进程架构也被使用到。 每个 Electron 中的 web 页面运行在它的叫渲染进程的进程中。
 
 在普通的浏览器中，web页面通常在沙盒环境中运行，并且无法访问操作系统的原生资源。 然而 Electron 的用户在 Node.js 的 API 支持下可以在页面中和操作系统进行一些底层交互。
 
 那么就有人会问了，主进程和渲染进程区别在哪里？
 
 主进程使用 `BrowserWindow` 实例创建页面。 每个 `BrowserWindow` 实例都在自己的渲染进程里运行页面。 当一个 `BrowserWindow` 实例被销毁后，相应的渲染进程也会被终止。
 
 主进程管理所有的web页面和它们对应的渲染进程。 每个渲染进程都是独立的，它只关心它所运行的 web 页面。
 
 在页面中调用与 GUI 相关的原生 API 是不被允许的，因为在 web 页面里操作原生的 GUI 资源是非常危险的，而且容易造成资源泄露。 如果你想在 web 页面里使用 GUI 操作，其对应的渲染进程必须与主进程进行通讯，请求主进程进行相关的 GUI 操作。
 
+ 主进程和渲染进程可以互相通信，比如`ipcMain`方式，当然通信有很多注意事项，在后续文章介绍

+ 各种安全性检查，当然内容并不全面，不过一切都是基于内容是否信任，一般非系统内部的都不信任，所以导致很多渲染进程调用主进程的方法无法使用。具体参考[检查安全列表](https://www.electronjs.org/docs/tutorial/security#checklist-security-recommendations)

