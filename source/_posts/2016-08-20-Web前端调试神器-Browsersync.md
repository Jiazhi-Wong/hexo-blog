---
title: 前端调试神器-Browsersync
date: 2016-08-20 21:31:43
tags:
  - 页面调试
  - Browsersync
  - Gulp
author: Jiazhi
header-img: debug.jpg
---

## 前言

前端调试是前端工程师必不可少的技能，掌握好的调试技巧，利用好的调试工具，可以有效地提高我们的开发效率。

本文要介绍的是调试工具是——Browsersync。引用[Browsersync中文网](http://www.browsersync.cn/)的介绍：

> Browsersync能让浏览器实时、快速响应您的文件更改（html、js、css、sass、less等）并自动刷新页面。更重要的是 **Browsersync可以同时在PC、平板、手机等设备下进项调试**。您可以想象一下：“假设您的桌子上有pc、ipad、iphone、android等设备，同时打开了您需要调试的页面，当您使用browsersync后，您的任何一次代码保存，以上的设备都会同时显示您的改动”。无论您是前端还是后端工程师，使用它将提高您30%的工作效率。

我强烈建议你配合使用Browsersync + Gulp/Grunt来搭建环境，因为Gulp/Grunt可以使Browsersync更强大。例如，Gulp对所有sass文件进行监听（`watch`），一旦文件被修改并保存，就会立即编译成CSS文件，并且Browsersync会把变化结果立即显示在所有终端上。而且配合Gulp/Grunt，会让Browsersync工作更加稳定。我就经常在只使用Browsersync的情况下，出现不稳定的情况。配合Gulp/Grunt，就不会出现什么问题。

本文将配合Browsersync + [Gulp](http://www.gulpjs.com.cn/)来搭建环境，并实现从sass到css进行调试的自动化过程，调试的整个过程你只要修改sass文件。你需要使用npm进行模块管理。

## 淘宝NPM镜像

有使用npm经验的都知道，在国内访问npm官方服务速度既慢，还不稳定。所以推荐使用[淘宝 NPM 镜像](https://npm.taobao.org/)。

你可以使用`cnpm`命令行工具代替默认的`npm`:

{% codeblock %}
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
{% endcodeblock %}

## cnpm init

我新建了一个名为"browsersync"的项目文件夹，在该项目目录下：

{% codeblock %}
$ cnpm init
{% endcodeblock %}

初始化完成后会在该项目目录下，生成一个`package.json`的文件。

在全局安装Gulp包：

{% codeblock %}
$ cnpm install -g gulp
{% endcodeblock %}

在项目里面安装gulp、gulp-sass、browser-sync：

{% codeblock %}
$ cnpm install --save-dev gulp gulp-sass browser-sync
{% endcodeblock %}

## gulpfile

接着我们创建`src`目录，并将我们的代码整理好放在该目录下。

顺利安装完以上这些模块后，就能开始写Gulp任务了。在该项目目录下新建一个名为`gulpfile.js`的文件，下面是我最终生成的目录结构：

{% asset_img catalog.jpg %}

`gulpfile.js`是定义Gulp任务的文件，它可以通过`gulp`命令来运行，接着把下面的代码放到`gulpfile.js`文件里面。

{% codeblock lang:javascript %}
// 引入了需要用到的模块
var gulp = require('gulp'),
  sass = require('gulp-sass'),
  browserSync = require('browser-sync');

// 任务sass，找到sass文件，通过编译后将css文件保存在'src/css'下
gulp.task('sass', function () {
  return gulp.src('src/sass/*.scss')
    .pipe(sass({outputStyle: 'expanded'}).on('error', sass.logError))
    .pipe(gulp.dest('src/css'));
});

// 任务sass:watch，监听sass文件
gulp.task('sass:watch', function () {
  var watcher = gulp.watch('src/sass/*.scss', ['sass']); // 一旦sass文件发生修改，会执行任务sass
});

// 任务browser-sync，监听html/css/js文件
gulp.task('browser-sync', function () {
  var files = [
    'src/**/*.html',
    'src/**/*.css',
    'src/**/*.js'
  ];

  // 本地静态文件
  browserSync.init(files, {
    server: {
      baseDir: 'src'  // 设置browsersync服务器根目录
    }
  });
});

// 任务default，gulp命令的默认入口
gulp.task('default', ['browser-sync', 'sass:watch']);
{% endcodeblock %}

## gulp

打开终端，在项目目录下输入命令`gulp`，出现下面的提示就说明成功了：

{% asset_img gulp.jpg %}

我们看到了预期的结果，包括服务器根目录`Serving files from: src`，本地服务器`Local: http://localhost:3000`，外置服务器`External: http://192.168.0.110:3000`。

接下来我们就可以在本地访问`http://localhost:3000`来访问`index.html`页面。

手机需要先跟我的电脑连在同一个wifi（同一局域网）下，通过访问`http://192.168.0.110:3000`，也可以得到想要的页面。

接着我们尝试修改`index.sass`，保存后，我们能够看到任务sass被执行了。并且任务完成后，Browsersync监听到了css文件的改变。同时也可以监听到html/js的变化。

{% asset_img gulp-change.jpg %}

## BroserSync

BrowserSync也可以在不同浏览器之间同步点击翻页、表单操作、滚动位置。你可以在电脑和手机上打开不同的浏览器然后进行操作。

{% asset_img BroserSync.gif %}

所有设备上的链接将会随之变化，当你在表单中输入文本时，每个窗口都会有输入。

{% asset_img BroserSync-input.gif %}

当你向下滚动页面时，所有设备上页面都会向下滚动（通常还很流畅！）。当你不想要这种行为时，也可以在`http://localhost:3001/sync-options`把这个功能关闭。

{% asset_img BroserSync-scroll.gif %}

BroserSync给我们前端工程师的调试带来了很大的方便，有了它，你不用在多个浏览器、多个设备间来回切换，频繁的刷新页面。

BroserSync的功能也不仅仅如此。BroserSync不仅可以创建本地静态环境，还可以在PHP，ASP，Rails和更多网站运行使用。使用Remote Debug (weinre)，当在weinre上审查元素的时候，手机上都会有相应的变化。想要了解更多，去[Browsersync中文网](http://www.browsersync.cn/)看看吧。