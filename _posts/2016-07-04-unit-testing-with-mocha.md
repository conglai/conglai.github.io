---
layout:     post
title:      "利用Gulp和Mocha构建一个简单的测试环境"
subtitle:   "build a unit testing environment with gulp and mocha"
date:       2016-07-04 22:00:00
author:     "雷猫"
header-img: "img/post/bg-iphone6s-apple.jpg"
header-mask: 0.6
catalog: 	true
tags:
 	- f2e
    - bdd   
---

# 利用Gulp和Mocha构建一个简单的测试环境

M（Monica）是一个前端新手，最近听说了一个新名词“单元测试”。上网搜了好多的介绍文章还是云里雾里。于是，这时候，旁边打着2K的男朋友L（Leslie）就遭殃了，于是，他开始了屈辱的教女朋友写代码的苦逼周末。以下是手把手的纪要。
>今天我们来学一学单元测试。这里不介绍一些概念性的东西，有兴趣的话可以上 [官网](http://mochajs.org/) 一探究竟。

## Mocha与Should的强强联手
---
L：好了，女王大人，学习时间到啦，我要开始了哦。
首先，老样子，安装一下mocha，建议安装在全局哈。

M：我知道，是不是Café Mocha里的那个Mocha，好想喝哇。

L：结束之后带你去喝好伐。

	npm install mocha -g
	
然后，新建一个mocha-test文件夹。在里面新建一个tests的文件夹来存放测试文件。好啦，开始写第一个测试文件吧。

	var assert = require('assert');
	describe('数组中是否存在某数', function () {
	  it('如果值不存在请返回－1谢谢', function () {
      	assert.equal(-1, [1,2,3].indexOf(2));
      });
    });

OK，运行一下

	mocha tests/demo1.test.js  //测试文件所在路径
	
发现界面如下：

	┌─[leslie@LeslieMiaus-MacBook-Pro] - [~/Documents/leslie/mocha-test] - [Sun Jun 	19, 07:36]
	└─[$] <> mocha tests


  	  数组中是否存在某数
    	1) 如果值不存在请返回－1谢谢


  	  0 passing (12ms)
  	  1 failing

  	  1) 数组中是否存在某数 如果值不存在请返回－1谢谢:

      	AssertionError: -1 == 1
      	+ expected - actual

     	--1
      	+1

      	at Context.<anonymous> (tests/demo1.test.js:4:15)

M：啊啊啊，怎么出错啦，你做了啥？

L：当然会出错啦😒，2在数组里面咯，稍微修改一下，把2改成2333，再运行试试：


	┌─[leslie@LeslieMiaus-MacBook-Pro] - [~/Documents/leslie/mocha-test] - [Sun Jun 	19, 07:38]
	└─[$] <> mocha tests/demo1.test.js
	
      数组中是否存在某数
        ✓ 如果值不存在请返回－1谢谢


  	  1 passing (8ms)
  	  
M：好棒，成功了诶。这是不是证明我们的第一个文件编写通过了。可是有一点很困惑诶，上面的那啥

	assert.equal(-1, [1,2,3].indexOf(2333));

这句话怎么看起来这么别扭啊，你能不能说人话，宝宝不开心，不稀饭这样子的写法。

L：好好好，下面隆重介绍should.js。一个符合我们英语表达的断言库。

OK，我们照常理通过`npm install should -g`安装`should.js`模块。然后在demo1文件中引入，然后，我们就可以改写成这样。

	[1,2,3].should.not.containEql(2333);
是不是跟写英语句子一样。什么？英语渣渣，滚回去考六级😄。

M：你小子想造反啊。（此处暴打5分钟）

L：哎，一言不合就家暴，宝宝心里苦啊。其实呢，官网给了判断的写法，可以参考，答应我不要死记好吗，忘了的话 [一键直达](https://github.com/tj/should.js)，不用感谢我，我叫雷锋。

好了，一个文件达成，那么，要是测试多个文件会变成什么样子呢？我们继续在`tests`目录下新建`demo2.test.js` 以及通过`mkdir subtests`来新建一个子目录，并在其中建立`demo3.test.js`。

M：什么吗？你写的辣么快，文件层级还辣么多，我哪里能记得住？

L：那么，我们来看看文件结构长啥样吧，请不要吐槽我的画风好吗？我尽力了😊

	mocha-test-
		|	
		- tests
			|
			- demo1.test.js
			- demo2.test.js
			- subtests
				|
				-demo3.test.js	
M：哈哈哈哈哈哈哈哈哈哈哈哈！😝

L：好的，我们看看`demo2.test.js`和`demo3.test.js`分别写了什么东西。

	//demo2
	var assert = require('assert');
	var should = require('should');
	it('给你4秒够不够', function(done) {
  	  var x = true;
  	  var f = function() {
    	x = false;
    	(x).should.not.be.ok;
    	done(); // 通知Mocha测试结束
  	  };
  	  setTimeout(f, 3000);
	});
	
	//demo3
	var assert = require('assert');
	var should = require('should');
	describe('哈哈哈我是demo3', function() {
		it('么么么', function() {});
	})
因为Mocha默认2000毫秒超时，所以呢，我们这里在运行时要加上`-t`来显式的告诉Mocha要等4秒钟来使其运行。像这样 

	mocha -t 4000 tests
M：咦啊啊啊，你怎么乱加东西咯，快解释清楚，那个done是啥意思？不解释清楚，呐，搓衣板在此，不谢。

L：其实done这个回调函数是用来通知Mocha测试结束，可以收工回家啦。如果不done的话Mocha就会乖乖的等到超时哦。不信你试试😄。当然`done`这个名称是约定俗成的，你可以叫任意的名字，比如`Wangcai`😄。

M：为什么demo3没执行？555，我写的香吻怎么没粗来啦。

L：其实呢，只要加上这么一句。别说子文件夹，子子孙孙文件夹我都能让你运行起来。

	mocha tests --recursive

香吻就出来了，不信你试试😄。来，奖励给我好不好😁。

M：切，想的美。这样子每次都跑一次那不是要到天荒地老。我要实时监测文件改动，自动执行Mocha。

L：😁，很简单，加上它就好啦。
 
 	mocha --watch tests --recursive
 
M：但是，还是有点想法，我如果说要在每次运行测试的时候先要做准备工作或者某一测试结束之后要清理现场怎么办好呢？
 
L：Mocha当然想到了这个，它提供了`before`、`after`、`beforeEach`、`afterEach`等四个钩子来帮你实现。

比如说我希望在项目运行结束前后给他画一个分隔符，这样我就知道这次测试了几个文件。那么，来写在demo2里看一下吧。当然，要引入`chalk`这个模块只是改一下颜色，不用在意，你也可以直接`console.log()`完事。

	before(function() {
	  console.log(chalk.bold.italic.cyan('<<<<<<<<<<<<<<<< test start >>>>>>>>>>>>>>>>>>>'));
	})
	after(function() {
	  console.log(chalk.bold.italic.cyan('<<<<<<<<<<<<<<<< test end >>>>>>>>>>>>>>>>>>>'));
	})
	
运行结果如下：

	┌─[leslie@LeslieMiaus-MacBook-Pro] - [~/Documents/leslie/mocha-test] - [Sun 	Jun 19, 08:47]
	└─[$] <> mocha tests --recursive


	<<<<<<<<<1<<<<<<< test start >>>>>>>>>>>>>>>>>>>
  	  ✓ 给你3秒够不够 (1006ms)
  	  数组中是否存在某数
      ✓ 如果值不存在请返回－1谢谢

  	  哈哈哈我是demo3
      ✓ 么么么

	<<<<<<<<<s<<<<<<< test end >>>>>>>>>>>>>>>>>>>

  	  3 passing (1s)
其实你只要在一个文件里写就可以把所有子孙目录下的文件包裹于里，所以可以在后期把这个单独写在一个公用的文件夹下面。

M：哇哦，好棒😁。
## 当Mocha遇上Gulp
---
L：上面我们介绍了mocha跟should结合的一点基本用法，一不小心说的太多啦。那么，我们现在来介绍加上Gulp之后的Mocha长啥样。

先引入`gulp`以及`gulp-mocha`：

	npm install gulp gulp-mocha
	
在mocha-test目录下新建一个`gulpfile`文件，写入如下代码：

	gulp.task('default', function() {
	  return gulp.src('tests/**/*.test.js', {read: false})
		.pipe(mocha({reporter: 'spec'}))
	})
在命令行中输入`gulp`看一下执行效果：

	┌─[leslie@LeslieMiaus-MacBook-Pro] - [~/Documents/leslie/mocha-test] - [Sun 	Jun 19, 08:59]
	└─[$] <> gulp
	[21:00:07] Using gulpfile ~/Documents/leslie/mocha-test/gulpfile.js
	[21:00:07] Starting 'default'...


	<<<<<<<<<<<<<<<< test start >>>>>>>>>>>>>>>>>>>
  	  ✓ 给你3秒够不够 (1005ms)
  	  数组中是否存在某数
      ✓ 如果值不存在请返回－1谢谢

  	  哈哈哈我是demo3
      ✓ 么么么

	<<<<<<<<<<<<<<<< test end >>>>>>>>>>>>>>>>>>>
	
M：那如果测试的时候，就更改了某一个文件，那怎么知道这个文件测试通过了，总不能全部再重新跑一遍吧？😢用Gulp来监测？

L：对，就是Gulp，来，直接看代码：

	var gulp = require('gulp');
	var mocha = require('gulp-mocha');
	gulp.task('default', ['watch'], function() {
	  return gulp.src('tests/**/*.test.js', {read: false})
		.pipe(mocha({reporter: 'spec'}))
	})
	gulp.task('watch', function() {
	  gulp.watch('tests/*.test.js', function(event) {
		return gulp.src(event.path)
		  .pipe(mocha({
			reporter: 'spec'
		  }))
	  })
	})

其实这里是用了`gulp.watch`中的`event`参数，这本来是用来告诉用户什么文件发生了什么类型的修改，刚好用来监测测试文件的变化，单一文件更改保存后，会单独执行该文件，也就不用等待Mocha漫长的测试时间（我不想黑Mocha，不过它确实比较慢😄）。

当然你也可以用单独的`gulp-watch`或者`gulp-changed`来实现单一文件的监测与修改。原理其实差不多，不妨自己试试看。
	
M：这里的代码是啥意思：

	{read: false}
	
L：当然是防止gulp读取档案内容，使其更快一些，笨蛋。

M：你说啥？骂我笨蛋，哀家天资聪颖，怎么会不知道，考考你，哼。

L：单元测试的工具除了mocha其实还有像`Jasmine`、`Karma`、`Tape`，`ava`，也值得去了解与学习。
其实单元测试应该加上`istanbul`(代码覆盖率)和`supertest`(模拟http请求)还有`makefile`(自动化编译)，这样凑齐的五大神兽或许才是单元测试的高阶形态吧。

M：你说啥？

L：这个不重要啊喂。讲的好久了哇，其它三大神兽我们以后在与他们会面吧。来，我们先去楼下喝Mocha吧😘。

>此文重点不在虐狗，在于学习写单元测试吗😁

## 参考资料
- [https://github.com/tj/should.js](https://github.com/tj/should.js)
- [http://mochajs.org/](http://mochajs.org/)
- [测试框架 Mocha 实例教程](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)

