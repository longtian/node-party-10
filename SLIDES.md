class: middle, center
background-image:  url(  );

# Node.JS 项目中的

### 性能监控的原理 和 Docker 的使用

<a href="https://github.com/wyvernnot/node-party-10">
<img style="position: absolute; top: 0; right: 0; border: 0;"
src="https://camo.githubusercontent.com/e7bbb0521b397edbd5fe43e7f760759336b5e05f/68747470733a2f2f73332e616d617a6f6e6177732e636f6d2f6769746875622f726962626f6e732f666f726b6d655f72696768745f677265656e5f3030373230302e706e67" alt="Fork me on GitHub" data-canonical-src="https://s3.amazonaws.com/github/ribbons/forkme_right_green_007200.png">
</a>

.text-muted[王龑 2016.03.26]

---

# 内容

- 关于我
- Node.JS 性能分析原理
 - 插桩
 - 采样
 - 内存
- Docker 在 Node.JS 项目中的使用
 - 兼容性测试
 - 分布式环境
 - 基于 Docker 的前端自动化工作流

---

# 关于我

--

count: false

## .label.label-danger[APM]

--

count: false

## .label.label-info[可视化]

--

count: false

## .label.label-warning[React]

--

count: false

## .label.label-primary[全栈]

---
class: middle,center,txt-orange
background-image: url(http://7xsa3z.com1.z0.glb.clouddn.com/f1.jpg)


# JavaScript 性能分析


---

## .center[插桩]

> 最早是由J.C. Huang 教授提出的，它是在保证被测程序原有逻辑完整性的基础上在程序中插入一些探针（又称为“探测仪”），
> 通过探针的执行并抛出程序运行的特征数据，通过对这些数据的分析，可以获得程序的控制流和数据流信息，进而得到逻辑覆盖等动态信息，从而实现测试目的的方法。

--

count:false

- `探针`

--

count:false

- `分析`

--

count:false

<br/>

.alert.alert-info[
　在运行时动态地修改代码
]

---

### 以 `alert` 为例

```
var _original=window.alert;
window.alert=function(){
    console.log('in');
    var result=_original.apply(this,arguments);
    console.log('out');
    return result;
}
```

--

count:false

我们可以得到:

- 进入
- 退出
- 返回值

---

### 以 `setTimeout` 为例

```
var _original_sett = window.setTimeout;
window.setTimeout = function () {
    console.log('enter');
    var args = [].slice.apply(arguments);
    var _original_callback = args.shift();
    var callback = function () {
        console.log('exit');
        _original_callback.apply(this, arguments);
    }
    args.unshift(callback);
    var result = _original_sett.apply(this, args);
    return result;
}
```

---

## 如何引入探针

```
const http = require('http');
http.createServer((req,res)=>{
  res.end('hello world');
}).listen(8080)
```

--
count: false

### 正常启动

```sh
node server.js
```

--
count: false

### 开启监控的启动

- **修改代码**

```
require('oneapm')
```

--
count:false

- **修改启动命令**

```sh
node -r oneapm server.js
```


---

### 发生了什么

```
node -h

Options:
  -v, --version         print Node.js version
  -e, --eval script     evaluate script
  -p, --print           evaluate script and print result
  -c, --check           syntax check script without executing
  -i, --interactive     always enter the REPL even if stdin
                        does not appear to be a terminal
* -r, --require         module to preload
```

```
  // Load preload modules
  function preloadModules() {
    if (process._preload_modules) {
*      NativeModule.require('module')
*          ._preloadModules(process._preload_modules);
    }
  }
```

---
class: middle, center

background-image: url(http://7xsa3z.com1.z0.glb.clouddn.com/trap.jpg)
---

### 前端埋点

<div class="well">
<img class="img-rounded" style="width:230px" src='http://7xsa3z.com1.z0.glb.clouddn.com/szj.jpg'/>
<h1 style="color:red">20.0</h1>
<button class="btn btn-primary">
<i class="fa fa-cart-plus"></i> 买买买</button>
</div>

---

### 一段 React 代码

```
class ProductItem extends React.Componet{

  render(){
    return <div>
      <img src="image-820yh402347y90.jpg"/>
      <button onClick={this.handleClick}> 买买买</button>
    </div>
  }

  handleClick(){
    // 加到购物车
  }
}
```

--
count:false

.alert.alert-info[
<q>运营: 统计点击访问和点击次数</q>
]

---
### ES7 Decorator

```
*import {pageview,event} from 'blablabla-decorators';

class ProductItem extends React.Componet{

* @pageview
  render(){
    return <div>
      <img src="image-820yh402347y90.jpg"/>
      <button onClick={this.handleClick}>点击我买买买</button>
    </div>
  }

* @event('add_to_cart')
  handleClick(){
    // 加到购物车
  }
}
```

---

class: center, middle

# 采样

---

class: bg-black,center

![](http://7xsa3z.com1.z0.glb.clouddn.com/banana.gif)

---

### 开始一段采样

- 浏览器

```
console.profile([NAME])

*// 前端的逻辑

console.profileEnd([NAME])
```

- Node.JS

```
var profiler = require('v8-profiler');
profiler.startProfiling([name]);

*// 后端的逻辑


profiler.stopProfiling([name]);
```

---
background-image: url(http://7xsa3z.com1.z0.glb.clouddn.com/cpu-mixedmode-node.svg)

### 输出 Flame Chart

.text-muted[http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html]
---
name: flame
background-image: url(http://7xsa3z.com1.z0.glb.clouddn.com/cpu-mixedmode-node.svg)
<div id="move" style="z-index:1"></div>
<div id="left-window"></div>
<div id="right-window"></div>
---

background-image: url(http://7xsa3z.com1.z0.glb.clouddn.com/top_overview.png)

## 内存

--
count: false

```
var heapdump = require('heapdump');
heapdump.writeSnapshot(function(err, filename) {
  console.log('dump written to', filename);
});
```

---

background-image: url(http://7xsa3z.com1.z0.glb.clouddn.com/top_all.png)

---

## 划分

- `新生区`：大多数对象被分配在这里。新生区是一个很小的区域，垃圾回收在这个区域非常频繁，与其他区域相独立。

- `老生指针区`：这里包含大多数可能存在指向其他对象的指针的对象。大多数在新生区存活一段时间之后的对象都会被挪到这里。

- `老生数据区`：这里存放只包含原始数据的对象（这些对象没有指向其他对象的指针）。字符串、封箱的数字以及未封箱的双精度数字数组，在新生区存活一段时间后会被移动到这里。

- `大对象区`：这里存放体积超越其他区大小的对象。每个对象有自己mmap产生的内存。垃圾回收器从不移动大对象。

- `代码区`：代码对象，也就是包含JIT之后指令的对象，会被分配到这里。这是唯一拥有执行权限的内存区（不过如果代码对象因过大而放在大对象区，则该大对象所对应的内存也是可执行的。译注：但是大对象内存区本身不是可执行的内存区）。

![](http://7xsa3z.com1.z0.glb.clouddn.com/inheritance.png)

---
class: middle,left
background-image: url(http://7xsa3z.com1.z0.glb.clouddn.com/gc.png)


GC 双刃剑

- 简化内存管理
- 无法掌控内存

* .small.text-muted[[图片 / Valtteri Mäki]]

---
class: middle,center

# Docker 在 Node.JS 项目中的运用

---

## 兼容性测试

- **Node.JS** `0.8` `0.10` `0.12` `4.x` `5.x`
- <del>**IO.JS**</del> `1.x` `2.x` `3.x`
- MySQL
- MongoDB
- Redis

--
count:false

<br/>
<br/>

## 分布式环境

> 监控 15 台机器上的 Node.JS 程序之间互相调用情况

---
class: bottom
background-image: url(http://7xsa3z.com1.z0.glb.clouddn.com/etcd.png)


.alert.alert-success[
docker compose scale web=15
]


---
class: bottom

background-image: url(http://7xsa3z.com1.z0.glb.clouddn.com/vis.png)


---

## 基于 Docker 的前端自动化工作流

--

count:false

### 流程

`Dockerfile` -> `Gitlab` -> `Jenkins` -> `ESLint` -> `Unit Test` -> `Build` -> `Tag` -> `Push` -> `Run` -> `A/B Testing`

--
count:false

<hr/>

### ab-docker

> 点点鼠标, Sandbox 的前端代码就会切换到制定的容器
>
> .fa.fa-github[[@ wyvernnot/ab-docker](https://github.com/wyvernnot/ab-docker)]


--
count:false

### qiuniu-webpack-plugin

> 部署到 CDN
>
> .fa.fa-github[[@ wyvernnot/qiniu-webpack-plugin](https://github.com/wyvernnot/qiniu-webpack-plugin)]

--
count:false

### statsd-webpack-plugin

> 针对 Webpack 的 APM 方案
>
> .fa.fa-github[[@ wyvernnot/statsd-webpack-plugin](https://github.com/wyvernnot/statsd-webpack-plugin)]

---

background-image: url(http://7xsa3z.com1.z0.glb.clouddn.com/webpack.png)