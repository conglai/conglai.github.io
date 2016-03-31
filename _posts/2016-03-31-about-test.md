---
layout:     	post
title:      	"前端测试"
date:       	2016-03-31 16:00:00
author:     	"宇山"
header-img: 	"img/post/bg-alibaba-bus-stop.jpg"
header-mask: 	0.3
catalog: 		true
tags:
    - mocha
    - js
---

# 单元测试

> 就目前自己的使用来讲，总希望能更快地验证自己的代码。

## 就自己本身的工作流程
1. 写主体代码（大致的结构）
2. 构造一个环境
3. 运行这段代码，来验证它
4. 如果出错（正常情况，很少有一次就运行成功的情况），重新执行第三个步骤；如果不出错，就完成了

## 集成测试
在以前的流程中，构造的环境就是运行环境。

### 好处：
* 不需要写额外环境

### 坏处：
* 构建缓慢（但是现代的热构建工具可以部分缓解这个压力）
* 测试流程更长

## 单元测试
区别于主体的环境，单元测试还要实现一套测试的环境

### 好处：
* 构建更快
* 测试流程更短

隐含的好处：

* 代码可维护性会提高，看测试更容易看懂代码


### 坏处：
* 需求写额外的测试代码

## 转换的过程
> 从无到有开始写单元测试是困难的！然而开始写了就会发现它的魅力

### 不适用场景
理论上来说，任何场景都可以使用单元测试，倒是可以说说可以不用单元测试的场景：

1. 本身就是构建脚本，是个串行的逻辑
2. 一次性的且十分短的代码

## 实战
### 接口测试
```
'use strict';
const co = require('co');
const helper = require('./helper');

describe('编辑节点的接口测试', function() {
  let res;
  let resData;
  before(function(done) {
    co(function*(){
      let options = {
        url: helper.getUrl() + '/i/enode'
      };
      options = yield helper.login(options);

      let memoryRes = yield helper.post(options);
      // console.log(memoryRes.body);
      res = memoryRes;
      resData = JSON.parse(memoryRes.body);
      done();
    });
  });
  describe('编辑节点成功', function() {
    it('response.statusCode should be equal 200', function() {
      res.statusCode.should.be.equal(200);
    });
    it('response.data.status should be equal 200', function() {
      resData.status.should.be.equal(200);
    });

  });

});
```
> 十分爽，写完这个，就可以在命令行中，watch代码的改变，不停地执行了，对比一下，单元测试和集成测试的流程：<br/>
> 单元测试：<br/>
> 1. 写代码<br/>
> 2. 看日志，来定位错误<br/>
> 集成测试：<br/>
> 1. 写代码<br/>
> 2. 编译<br/>
> 3. 打开浏览器访问一下<br/>
> 4. 看日志，来定位错误

### 功能测试
```
'use strict';
const md5 = require('spark-md5');
let s;
const config = require('../config/dev');
before(function(done) {
  let InterfaceModel = require('../router/model/interface.js');
  s = new InterfaceModel({}, config, {
    log: function() {
      console.log.apply(console, arguments);
    }
  });
  done();
});
describe('请求服务测试', function() {

  it('排列请求参数', function() {
    let paramsStr = s.sortParams({
      dd: 'ss',
      dc: 'ss',
      ad: 'ss',
      cd: 'ss'
    });
    paramsStr.should.be.equal('ad=ss&cd=ss&dc=ss&dd=ss');
  });
  it('测试post', function(done) {
    s.post('USER-WEB-LOGIN', {
      mobile: '15988199131',
      password: md5.hash('123456')
    }).then(res => {
      let data = JSON.parse(res.body);
      data.code.should.be.equal('USER-WEB-LOGIN');
      done();
    });
  });
  it('测试', function(done) {
    s.post('WEB-TIMELINE-INDEX', { uid: 1332103320130560,
      user_token: '0ed7252982704c073e5ab90d45c87617',
      action: 'WEB-TIMELINE-INDEX',
      time: 1455773354408
    }, 'http://121.41.32.161:7080/web/timeline/web-timeline-index.htm').then(res => {

      // console.log(res.body);
      console.log('---------')
      try {
        let data = new Function('return ' + res.body)();
        console.log(data);

      } catch(e) {
        console.log(e);
      }
      console.log('-----sss---')
      // data.code.should.be.equal('USER-WEB-LOGIN');
      done();
    });
  });
});
```
> 用来验证某一个函数是否正确


### 更多使用
> 我也会用它来跑数据，哈哈



```
'use strict';
let log = require('../log.js');
describe('正则测试', function() {
  it('返回正确的日志Map', function() {
    let res = log.getLogPath();
    console.log(res);
    res.should.be.a.Object();
  });

  it('跑日志', function() {
    let fs = require('fs');
    // let pre = 'withme-activity-out-7__';
    let pre = 'withme-web-out-4__';
    let date = '2016-01-27';
    let logStr = fs.readFileSync('./logs/' + pre  + date + '-00-00.log', 'utf8');
    console.log(`-----${pre}${date}-----`);
    // console.log(logStr);
    let result = log.processLog(logStr);
    console.log('ua:' + result.uas.length);
    console.log('i:' + result.iRequests.length);
    console.log('pv:' + result.pages.length);
    let uv = 0;
    let nip = 0;
    let pages = result.pages;
    let pvMap = {};
    pages.forEach(page => {
      let ip = page.header['x-real-ip'] || page.header['x-forwarded-for'];
      // console.log(ip);
      let url = page.url;
      if(!ip) {
        nip ++;
        return;
      }
      if(!pvMap[ip]) {
        pvMap[ip] = true;
        uv ++;
      }
    });
    console.log('has ip uv:' + uv);
    console.log('no ip pv:' + nip);
    // res.should.be.a.Object();
  });
});